# 输入DStreams

输入DStreams表示从数据源获取的原始数据流。Spark Streaming拥有两类数据源
- 基本源（Basic sources）：这些源在StreamingContext API中直接可用。例如文件系统、套接字连接、Akka的actor等。
- 高级源（Advanced sources）：这些源包括Kafka,Flume,Kinesis,Twitter等等。它们需要通过额外的类来使用。我们在[链接](linking.md)那一节讨论了类依赖。

每一个输入DStreams（除了文件流）都和一个单独的[Receiver](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.streaming.receiver.Receiver)对象相关联。
这个对象从源中接收数据，然后将数据保存到Spark的内存中用来处理。所以，每一个输入DStream都接收一个单独的数据流。需要注意的是，在一个流应用中，你能够并行的创建多个输入DStream来接收多个
数据流。这将在[性能调优](../performance-tuning/README.md)那一节介绍。

`receiver`作为一个长期运行的任务运行在Spark worker或executor中。因此，它占有一个核，这个核是分配给Spark Streaming应用程序的所有核中的一个（it occupies one of the cores
allocated to the Spark Streaming application）。所以，为Spark Streaming应用程序分配足够的核用以处理接收的数据和运行`receiver`是非常重要的。

几点需要注意的地方：
- 如果分配给应用程序的核的数量少于或者等于输入DStreams或者receivers的数量，系统只能够接收数据而不能处理它们。
- 当运行在本地，如果你的master URL被设置成了“local”，这样就只有一个核运行任务。这对程序来说是不足的，因为作为`receiver`的输入DStream将会占用这个核，这样就没有剩余的核来处理数据了。

## 基本源

我们已经在[快速例子](../a-quick-example.md)中看到，`ssc.socketTextStream(...)`方法用来把从TCP套接字获取的文本数据创建成DStream。除了套接字，StreamingContext API也支持把文件
以及Akka actors作为输入源创建DStream。

- 文件流（File Streams）：从任何与HDFS API兼容的文件系统中读取数据，一个DStream可以通过如下方式创建

```scala
streamingContext.fileStream[keyClass, valueClass, inputFormatClass](dataDirectory)
```
Spark Streaming将会监控`dataDirectory`目录，并且处理目录下生成的任何文件（嵌套目录不被支持）。需要注意一下三点：

    - 所有文件必须具有相同的数据格式
    - 所有文件必须在`dataDirectory`目录下创建，文件是自动的移动和重命名到数据目录下
    - 一旦移动，文件必须被修改。所以如果文件被持续的附加数据，新的数据不会被读取。

对于简单的文本文件，有一个更简单的方法`streamingContext.textFileStream(dataDirectory)`可以被调用。文件流不需要运行一个receiver，所以不需要分配核。

- 基于自定义actor的流：DStream可以调用`streamingContext.actorStream(actorProps, actor-name)`方法从Akka actors获取的数据流来创建。具体的信息见[自定义receiver指南](https://spark.apache.org/docs/latest/streaming-custom-receivers.html#implementing-and-using-a-custom-actor-based-receiver)
- RDD队列作为数据流：为了用测试数据测试Spark Streaming应用程序，人们也可以调用`streamingContext.queueStream(queueOfRDDs)`方法基于RDD队列创建DStreams。每个push到队列的RDD都被
当做DStream的批数据，像流一样处理。

关于从套接字、文件和actor中获取流的更多细节，请看[StreamingContext](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.streaming.StreamingContext)和
[JavaStreamingContext](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/streaming/api/java/JavaStreamingContext.html)