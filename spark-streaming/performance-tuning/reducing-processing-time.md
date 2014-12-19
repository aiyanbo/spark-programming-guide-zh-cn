# 减少批数据的执行时间

在Spark中有几个优化可以减少批处理的时间。这些可以在[优化指南](../../other/tuning-spark.md)中作了讨论。这节重点讨论几个重要的。

## 数据接收的并行水平

通过网络(如kafka，flume，socket等)接收数据需要这些数据反序列化并被保存到Spark中。如果数据接收成为系统的瓶颈，就要考虑并行地接收数据。注意，每个输入DStream创建一个`receiver`（运行在worker机器上）
接收单个数据流。创建多个输入DStream并配置它们可以从源中接收不同分区的数据流，从而实现多数据流接收。例如，接收两个topic数据的单个输入DStream可以被切分为两个kafka输入流，每个接收一个topic。这将
在两个worker上运行两个`receiver`，因此允许数据并行接收，提高整体的吞吐量。多个DStream可以被合并生成单个DStream，这样运用在单个输入DStream的transformation操作可以运用在合并的DStream上。

```scala
val numStreams = 5
val kafkaStreams = (1 to numStreams).map { i => KafkaUtils.createStream(...) }
val unifiedStream = streamingContext.union(kafkaStreams)
unifiedStream.print()
```

另外一个需要考虑的参数是`receiver`的阻塞时间。对于大部分的`receiver`，在存入Spark内存之前，接收的数据都被合并成了一个大数据块。每批数据中块的个数决定了任务的个数。这些任务是用类
似map的transformation操作接收的数据。阻塞间隔由配置参数`spark.streaming.blockInterval`决定，默认的值是200毫秒。

多输入流或者多`receiver`的可选的方法是明确地重新分配输入数据流（利用`inputStream.repartition(<number of partitions>)`），在进一步操作之前，通过集群的机器数分配接收的批数据。

## 数据处理的并行水平

如果运行在计算stage上的并发任务数不足够大，就不会充分利用集群的资源。例如，对于分布式reduce操作如`reduceByKey`和`reduceByKeyAndWindow`，默认的并发任务数通过配置属性来确定（configuration.html#spark-properties）
`spark.default.parallelism`。你可以通过参数（`PairDStreamFunctions` (api/scala/index.html#org.apache.spark.streaming.dstream.PairDStreamFunctions)）传递并行度，或者设置参数
`spark.default.parallelism`修改默认值。

## 数据序列化

数据序列化的总开销是平常大的，特别是当sub-second级的批数据被接收时。下面有两个相关点：

- Spark中RDD数据的序列化。关于数据序列化请参照[优化指南](../../other/tuning-spark.md)。注意，与Spark不同的是，默认的RDD会被持久化为序列化的字节数组，以减少与垃圾回收相关的暂停。
- 输入数据的序列化。从外部获取数据存到Spark中，获取的byte数据需要从byte反序列化，然后再按照Spark的序列化格式重新序列化到Spark中。因此，输入数据的反序列化花费可能是一个瓶颈。

## 任务的启动开支

每秒钟启动的任务数是非常大的（50或者更多）。发送任务到slave的花费明显，这使请求很难获得亚秒（sub-second）级别的反应。通过下面的改变可以减小开支

- 任务序列化。运行kyro序列化任何可以减小任务的大小，从而减小任务发送到slave的时间。
- 执行模式。在Standalone模式下或者粗粒度的Mesos模式下运行Spark可以在比细粒度Mesos模式下运行Spark获得更短的任务启动时间。可以在[在Mesos下运行Spark](../../deploying/running-spark-on-mesos.md)中获取更多信息。

These changes may reduce batch processing time by 100s of milliseconds, thus allowing sub-second batch size to be viable.

