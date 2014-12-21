# 性能调优

集群中的Spark Streaming应用程序获得最好的性能需要一些调整。这章将介绍几个参数和配置，提高Spark Streaming应用程序的性能。你需要考虑两件事情：

- 高效地利用集群资源减少批数据的处理时间
- 设置正确的批容量（size），使数据的处理速度能够赶上数据的接收速度

* [减少批数据的执行时间](reducing-processing-time.md)
* [设置正确的批容量](setting-right-batch-size.md)
* [内存调优](memory-tuning.md)