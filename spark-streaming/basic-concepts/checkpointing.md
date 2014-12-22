# Checkpointing

一个流应用程序必须全天候运行，所有必须能够解决应用程序逻辑无关的故障（如系统错误，JVM崩溃等）。为了使这成为可能，Spark Streaming需要checkpoint足够的信息到容错存储系统中，
以使系统从故障中恢复。

- Metadata checkpointing：保存流计算的定义信息到容错存储系统如HDFS中。这用来恢复应用程序中运行worker的节点的故障。元数据包括
    - Configuration ：创建Spark Streaming应用程序的配置信息
    - DStream operations ：定义Streaming应用程序的操作集合
    - Incomplete batches：操作存在队列中的未完成的批
- Data checkpointing ：保存生成的RDD到可靠的存储系统中，这在有状态transformation（如结合跨多个批次的数据）中是必须的。在这样一个transformation中，生成的RDD依赖于之前
批的RDD，随着时间的推移，这个依赖链的长度会持续增长。在恢复的过程中，为了避免这种无限增长。有状态的transformation的中间RDD将会定时地存储到可靠存储系统中，以截断这个依赖链。

元数据checkpoint主要是为了从driver故障中恢复数据。如果transformation操作被用到了，数据checkpoint即使在简单的操作中都是必须的。

## 何时checkpoint

应用程序在下面两种情况下必须开启checkpoint

- 使用有状态的transformation。如果在应用程序中用到了`updateStateByKey`或者`reduceByKeyAndWindow`，checkpoint目录必需提供用以定期checkpoint RDD。
- 从运行应用程序的driver的故障中恢复过来。使用元数据checkpoint恢复处理信息。

注意，没有前述的有状态的transformation的简单流应用程序在运行时可以不开启checkpoint。在这种情况下，从driver故障的恢复将是部分恢复（接收到了但是还没有处理的数据将会丢失）。
这通常是可以接受的，许多运行的Spark Streaming应用程序都是这种方式。

## 怎样配置Checkpointing

在容错、可靠的文件系统（HDFS、s3等）中设置一个目录用于保存checkpoint信息。着可以通过`streamingContext.checkpoint(checkpointDirectory)`方法来做。这运行你用之前介绍的
有状态transformation。另外，如果你想从driver故障中恢复，你应该以下面的方式重写你的Streaming应用程序。

- 当应用程序是第一次启动，新建一个StreamingContext，启动所有Stream，然后调用`start()`方法
- 当应用程序因为故障重新启动，它将会从checkpoint目录checkpoint数据重新创建StreamingContext

```scala
// Function to create and setup a new StreamingContext
def functionToCreateContext(): StreamingContext = {
    val ssc = new StreamingContext(...)   // new context
    val lines = ssc.socketTextStream(...) // create DStreams
    ...
    ssc.checkpoint(checkpointDirectory)   // set checkpoint directory
    ssc
}

// Get StreamingContext from checkpoint data or create a new one
val context = StreamingContext.getOrCreate(checkpointDirectory, functionToCreateContext _)

// Do additional setup on context that needs to be done,
// irrespective of whether it is being started or restarted
context. ...

// Start the context
context.start()
context.awaitTermination()
```

如果`checkpointDirectory`存在，上下文将会利用checkpoint数据重新创建。如果这个目录不存在，将会调用`functionToCreateContext`函数创建一个新的上下文，建立DStreams。
请看[RecoverableNetworkWordCount](https://github.com/apache/spark/tree/master/examples/src/main/scala/org/apache/spark/examples/streaming/RecoverableNetworkWordCount.scala)例子。

除了使用`getOrCreate`，开发者必须保证在故障发生时，driver处理自动重启。


