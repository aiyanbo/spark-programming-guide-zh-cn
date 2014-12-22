# 初始化StreamingContext

为了初始化Spark Streaming程序，一个StreamingContext对象必需被创建，它是Spark Streaming所有流操作的主要入口。一个[StreamingContext](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.streaming.StreamingContext)
对象可以用[SparkConf](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.SparkConf)对象创建。

```scala
import org.apache.spark._
import org.apache.spark.streaming._
val conf = new SparkConf().setAppName(appName).setMaster(master)
val ssc = new StreamingContext(conf, Seconds(1))
```

`appName`表示你的应用程序显示在集群UI上的名字，`master`是一个[Spark、Mesos、YARN](https://spark.apache.org/docs/latest/submitting-applications.html#master-urls)集群URL
或者一个特殊字符串“local[*]”，它表示程序用本地模式运行。当程序运行在集群中时，你并不希望在程序中硬编码`master`，而是希望用`spark-submit`启动应用程序，并从`spark-submit`中得到
`master`的值。对于本地测试或者单元测试，你可以传递“local”字符串在同一个进程内运行Spark Streaming。需要注意的是，它在内部创建了一个SparkContext对象，你可以通过` ssc.sparkContext`
访问这个SparkContext对象。

批时间片需要根据你的程序的潜在需求以及集群的可用资源来设定，你可以在[性能调优](../performance-tuning/README.md)那一节获取详细的信息。

可以利用已经存在的`SparkContext`对象创建`StreamingContext`对象。

```scala
import org.apache.spark.streaming._
val sc = ...                // existing SparkContext
val ssc = new StreamingContext(sc, Seconds(1))
```

当一个上下文（context）定义之后，你必须按照以下几步进行操作

- 定义输入源；
- 准备好流计算指令；
- 利用`streamingContext.start()`方法接收和处理数据；
- 处理过程将一直持续，直到`streamingContext.stop()`方法被调用。

几点需要注意的地方：

- 一旦一个context已经启动，就不能有新的流算子建立或者是添加到context中。
- 一旦一个context已经停止，它就不能再重新启动
- 在JVM中，同一时间只能有一个StreamingContext处于活跃状态
- 在StreamingContext上调用`stop()`方法，也会关闭SparkContext对象。如果只想仅关闭StreamingContext对象，设置`stop()`的可选参数为false
- 一个SparkContext对象可以重复利用去创建多个StreamingContext对象，前提条件是前面的StreamingContext在后面StreamingContext创建之前关闭（不关闭SparkContext）。