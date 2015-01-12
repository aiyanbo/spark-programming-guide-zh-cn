# 内存调优

调整内存的使用以及Spark应用程序的垃圾回收行为已经在[Spark优化指南](../../other/tuning-spark.md)中详细介绍。在这一节，我们重点介绍几个强烈推荐的自定义选项，它们可以
减少Spark Streaming应用程序垃圾回收的相关暂停，获得更稳定的批处理时间。

- Default persistence level of DStreams：和RDDs不同的是，默认的持久化级别是序列化数据到内存中（DStream是`StorageLevel.MEMORY_ONLY_SER`，RDD是` StorageLevel.MEMORY_ONLY`）。
即使保存数据为序列化形态会增加序列化/反序列化的开销，但是可以明显的减少垃圾回收的暂停。
- Clearing persistent RDDs：默认情况下，通过Spark内置策略（LUR），Spark Streaming生成的持久化RDD将会从内存中清理掉。如果spark.cleaner.ttl已经设置了，比这个时间存在更老的持久化
RDD将会被定时的清理掉。正如前面提到的那样，这个值需要根据Spark Streaming应用程序的操作小心设置。然而，可以设置配置选项`spark.streaming.unpersist`为true来更智能的去持久化（unpersist）RDD。这个
配置使系统找出那些不需要经常保有的RDD，然后去持久化它们。这可以减少Spark RDD的内存使用，也可能改善垃圾回收的行为。
- Concurrent garbage collector：使用并发的标记-清除垃圾回收可以进一步减少垃圾回收的暂停时间。尽管并发的垃圾回收会减少系统的整体吞吐量，但是仍然推荐使用它以获得更稳定的批处理时间。