# 一个快速的例子

在我们进入如何编写Spark Streaming程序的细节之前，让我们快速地浏览一个简单的例子。这个例子从监听TCP套接字的数据服务器获取文本数据，然后计算文本中包含的单词数。做法如下：

首先，我们导入Spark Streaming的相关类以及一些从StreamingContext获得的隐式转换到我们的环境中，为我们所需的其他类（如DStream）提供有用的方法。[StreamingContext](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.streaming.StreamingContext)
是所有流函数的主要入口。我们创建了一个具有两个执行线程以及1秒批间隔时间(即将每秒的数据分为一批)的本地StreamingContext。

```scala
import org.apache.spark._
import org.apache.spark.streaming._
import org.apache.spark.streaming.StreamingContext._
// Create a local StreamingContext with two working thread and batch interval of 1 second
val conf = new SparkConf().setMaster("local[2]").setAppName("NetworkWordCount")
val ssc = new StreamingContext(conf, Seconds(1))
```
利用这个上下文，我们能够创建一个DStream，它表示从TCP源（主机位localhost，端口为9999）获取的流式数据。

```scala
// Create a DStream that will connect to hostname:port, like localhost:9999
val lines = ssc.socketTextStream("localhost", 9999)
```
这个`lines`变量是一个DStream，表示即将从数据服务器获得的流数据。这个DStream的每条记录都代表一行文本。下一步，我们需要将DStream中的每行文本都切分为单词。

```scala
// Split each line into words
val words = lines.flatMap(_.split(" "))
```
`flatMap`是一个一对多的DStream操作，它通过把源DStream的每条记录都生成多条新记录来创建一个新的DStream。在这个例子中，每行文本都被切分成了多个单词，我们把切分
的单词流用`words`这个DStream表示。下一步，我们需要计算单词的个数。

```scala
import org.apache.spark.streaming.StreamingContext._
// Count each word in each batch
val pairs = words.map(word => (word, 1))
val wordCounts = pairs.reduceByKey(_ + _)
// Print the first ten elements of each RDD generated in this DStream to the console
wordCounts.print()
```
`words`这个DStream被mapper(一对一转换操作)成了一个新的DStream，它由（word，1）对组成。然后，我们就可以用这个新的DStream计算每批数据的词频。最后，我们用`wordCounts.print()`
打印每秒计算的词频。

需要注意的是，当以上这些代码被执行时，Spark Streaming仅仅准备好了它要执行的计算，实际上并没有真正开始执行。在这些转换操作准备好之后，要真正执行计算，需要调用如下的方法

```scala
ssc.start()             // Start the computation
ssc.awaitTermination()  // Wait for the computation to terminate
```
完整的例子可以在[NetworkWordCount](https://github.com/apache/spark/blob/master/examples/src/main/scala/org/apache/spark/examples/streaming/NetworkWordCount.scala)中找到。

如果你已经下载和构建了Spark环境，你就能够用如下的方法运行这个例子。首先，你需要运行Netcat作为数据服务器

```shell
$ nc -lk 9999
```
然后，在不同的终端，你能够用如下方式运行例子
```shell
$ ./bin/run-example streaming.NetworkWordCount localhost 9999
```
