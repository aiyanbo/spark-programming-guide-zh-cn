# 输入DStreams和receivers

输入DStreams表示从数据源获取输入数据流的DStreams。在[快速例子](../a-quick-example.md)中，`lines`表示输入DStream，它代表从netcat服务器获取的数据流。每一个输入流DStream
和一个`Receiver`对象相关联，这个`Receiver`从源中获取数据，并将数据存入内存中用于处理。

输入DStreams表示从数据源获取的原始数据流。Spark Streaming拥有两类数据源
- 基本源（Basic sources）：这些源在StreamingContext API中直接可用。例如文件系统、套接字连接、Akka的actor等。
- 高级源（Advanced sources）：这些源包括Kafka,Flume,Kinesis,Twitter等等。它们需要通过额外的类来使用。我们在[链接](linking.md)那一节讨论了类依赖。

需要注意的是，如果你想在一个流应用中并行地创建多个输入DStream来接收多个数据流，你能够创建多个输入流（这将在[性能调优](../performance-tuning/README.md)那一节介绍）
。它将创建多个Receiver同时接收多个数据流。但是，`receiver`作为一个长期运行的任务运行在Spark worker或executor中。因此，它占有一个核，这个核是分配给Spark Streaming应用程序的所有
核中的一个（it occupies one of the cores allocated to the Spark Streaming application）。所以，为Spark Streaming应用程序分配足够的核（如果是本地运行，那么是线程）
用以处理接收的数据并且运行`receiver`是非常重要的。

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

    1 所有文件必须具有相同的数据格式
    2 所有文件必须在`dataDirectory`目录下创建，文件是自动的移动和重命名到数据目录下
    3 一旦移动，文件必须被修改。所以如果文件被持续的附加数据，新的数据不会被读取。

对于简单的文本文件，有一个更简单的方法`streamingContext.textFileStream(dataDirectory)`可以被调用。文件流不需要运行一个receiver，所以不需要分配核。

在Spark1.2中，`fileStream`在Python API中不可用，只有`textFileStream`可用。

- 基于自定义actor的流：DStream可以调用`streamingContext.actorStream(actorProps, actor-name)`方法从Akka actors获取的数据流来创建。具体的信息见[自定义receiver指南](https://spark.apache.org/docs/latest/streaming-custom-receivers.html#implementing-and-using-a-custom-actor-based-receiver)
`actorStream`在Python API中不可用。
- RDD队列作为数据流：为了用测试数据测试Spark Streaming应用程序，人们也可以调用`streamingContext.queueStream(queueOfRDDs)`方法基于RDD队列创建DStreams。每个push到队列的RDD都被
当做DStream的批数据，像流一样处理。

关于从套接字、文件和actor中获取流的更多细节，请看[StreamingContext](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.streaming.StreamingContext)和
[JavaStreamingContext](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/streaming/api/java/JavaStreamingContext.html)

## 高级源

这类源需要非Spark库接口，并且它们中的部分还需要复杂的依赖（例如kafka和flume）。为了减少依赖的版本冲突问题，从这些源创建DStream的功能已经被移到了独立的库中，你能在[链接](linking.md)查看
细节。例如，如果你想用来自推特的流数据创建DStream，你需要按照如下步骤操作：

- 链接：添加`spark-streaming-twitter_2.10`到SBT或maven项目的依赖中
- 编写：导入`TwitterUtils`类，用`TwitterUtils.createStream`方法创建DStream,如下所示
```scala
import org.apache.spark.streaming.twitter._
TwitterUtils.createStream(ssc)
```
- 部署：将编写的程序以及其所有的依赖（包括spark-streaming-twitter_2.10的依赖以及它的传递依赖）打为jar包，然后部署。这在[部署章节](deploying-applications.md)将会作更进一步的介绍。

需要注意的是，这些高级的源在`spark-shell`中不能被使用，因此基于这些源的应用程序无法在shell中测试。

下面将介绍部分的高级源：

- Twitter：Spark Streaming利用`Twitter4j 3.0.3`获取公共的推文流，这些推文通过[推特流API](https://dev.twitter.com/docs/streaming-apis)获得。认证信息可以通过Twitter4J库支持的
任何[方法](http://twitter4j.org/en/configuration.html)提供。你既能够得到公共流，也能够得到基于关键字过滤后的流。你可以查看API文档（[scala](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.streaming.twitter.TwitterUtils$)和[java](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/streaming/twitter/TwitterUtils.html)）
和例子（[TwitterPopularTags](https://github.com/apache/spark/blob/master/examples/src/main/scala/org/apache/spark/examples/streaming/TwitterPopularTags.scala)和[TwitterAlgebirdCMS](https://github.com/apache/spark/blob/master/examples/src/main/scala/org/apache/spark/examples/streaming/TwitterAlgebirdCMS.scala)）
- Flume：Spark Streaming 1.2能够从flume 1.4.0中获取数据，可以查看[flume集成指南](flume-integration-guide.md)了解详细信息
- Kafka：Spark Streaming 1.2能够从kafka 0.8.0中获取数据，可以查看[kafka集成指南](kafka-integration-guide.md)了解详细信息
- Kinesis：查看[Kinesis集成指南](kinesis-integration.md)了解详细信息

## 自定义源

在Spark 1.2中，这些源不被Python API支持。
输入DStream也可以通过自定义源创建，你需要做的是实现用户自定义的`receiver`，这个`receiver`可以从自定义源接收数据以及将数据推到Spark中。通过[自定义receiver指南](custom-receiver.md)了解详细信息

## Receiver可靠性

基于可靠性有两类数据源。源(如kafka、flume)允许。如果从这些可靠的源获取数据的系统能够正确的应答所接收的数据，它就能够确保在任何情况下不丢失数据。这样，就有两种类型的receiver：

- Reliable Receiver：一个可靠的receiver正确的应答一个可靠的源，数据已经收到并且被正确地复制到了Spark中。
- Unreliable Receiver ：这些receivers不支持应答。即使对于一个可靠的源，开发者可能实现一个非可靠的receiver，这个receiver不会正确应答。

怎样编写可靠的Receiver的细节在[自定义receiver](custom-receiver.md)中有详细介绍。
