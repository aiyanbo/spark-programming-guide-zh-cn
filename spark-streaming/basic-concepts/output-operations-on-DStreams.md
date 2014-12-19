# DStreams上的输出操作

输出操作允许DStream的操作推到如数据库、文件系统等外部系统中。因为输出操作实际上是允许外部系统消费转换后的数据，它们触发的实际操作是DStream转换。目前，定义了下面几种输出操作：

Output Operation | Meaning
--- | ---
print() | 在DStream的每个批数据中打印前10条元素，这个操作在开发和调试中都非常有用
saveAsObjectFiles(prefix, [suffix]) | 保存DStream的内容为一个序列化的文件`SequenceFile`。每一个批间隔的文件的文件名基于`prefix`和`suffix`生成。"prefix-TIME_IN_MS[.suffix]"
saveAsTextFiles(prefix, [suffix]) | 保存DStream的内容为一个文本文件。每一个批间隔的文件的文件名基于`prefix`和`suffix`生成。"prefix-TIME_IN_MS[.suffix]"
saveAsHadoopFiles(prefix, [suffix]) | 保存DStream的内容为一个hadoop文件。每一个批间隔的文件的文件名基于`prefix`和`suffix`生成。"prefix-TIME_IN_MS[.suffix]"
foreachRDD(func) | 在从流中生成的每个RDD上应用函数`func`的最通用的输出操作。这个函数应该推送每个RDD的数据到外部系统，例如保存RDD到文件或者通过网络写到数据库中。需要注意的是，`func`函数在驱动程序中执行，并且通常都有RDD action在里面推动RDD流的计算。

## 利用foreachRDD的设计模式

dstream.foreachRDD是一个强大的原语，发送数据到外部系统中。然而，明白怎样正确地、有效地用这个原语是非常重要的。下面几点介绍了如何避免一般错误。
- 经常写数据到外部系统需要建一个连接对象（例如到远程服务器的TCP连接），用它发送数据到远程系统。为了达到这个目的，开发人员可能不经意的在Spark驱动中创建一个连接对象，但是在Spark worker中
尝试调用这个连接对象保存记录到RDD中，如下：

```scala
  dstream.foreachRDD(rdd => {
      val connection = createNewConnection()  // executed at the driver
      rdd.foreach(record => {
          connection.send(record) // executed at the worker
      })
  })
```

这是不正确的，因为这需要先序列化连接对象，然后将它从driver发送到worker中。这样的连接对象在机器之间不能传送。它可能表现为序列化错误（连接对象不可序列化）或者初始化错误（连接对象应该
在worker中初始化）等等。正确的解决办法是在worker中创建连接对象。

- 然而，这会造成另外一个常见的错误-为每一个记录创建了一个连接对象。例如：

```
  dstream.foreachRDD(rdd => {
      rdd.foreach(record => {
          val connection = createNewConnection()
          connection.send(record)
          connection.close()
      })
  })
```

通常，创建一个连接对象有资源和时间的开支。因此，为每个记录创建和销毁连接对象会导致非常高的开支，明显的减少系统的整体吞吐量。一个更好的解决办法是利用`rdd.foreachPartition`方法。
为RDD的partition创建一个连接对象，用这个两件对象发送partition中的所有记录。

```
 dstream.foreachRDD(rdd => {
      rdd.foreachPartition(partitionOfRecords => {
          val connection = createNewConnection()
          partitionOfRecords.foreach(record => connection.send(record))
          connection.close()
      })
  })
```
这就将连接对象的创建开销分摊到了partition的所有记录上了。

- 最后，可以通过在多个RDD或者批数据间重用连接对象做更进一步的优化。开发者可以保有一个静态的连接对象池，重复使用池中的对象将多批次的RDD推送到外部系统，以进一步节省开支。

```
  dstream.foreachRDD(rdd => {
      rdd.foreachPartition(partitionOfRecords => {
          // ConnectionPool is a static, lazily initialized pool of connections
          val connection = ConnectionPool.getConnection()
          partitionOfRecords.foreach(record => connection.send(record))
          ConnectionPool.returnConnection(connection)  // return to the pool for future reuse
      })
  })
```

需要注意的是，池中的连接对象应该根据需要延迟创建，并且在空闲一段时间后自动超时。这样就获取了最有效的方式发生数据到外部系统。

其它需要注意的地方：

- 输出操作通过懒执行的方式操作DStreams，正如RDD action通过懒执行的方式操作RDD。具体地看，RDD actions和DStreams输出操作接收数据的处理。因此，如果你的应用程序没有任何输出操作或者
用于输出操作`dstream.foreachRDD()`，但是没有任何RDD action操作在`dstream.foreachRDD()`里面，那么什么也不会执行。系统仅仅会接收输入，然后丢弃它们。
- 默认情况下，DStreams输出操作是分时执行的，它们按照应用程序的定义顺序按序执行。


