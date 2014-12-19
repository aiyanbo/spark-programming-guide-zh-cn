# 缓存或持久化

和RDD相似，DStreams也允许开发者持久化流数据到内存中。在DStream上使用`persist()`方法可以自动地持久化DStream中的RDD到内存中。如果DStream中的数据需要计算多次，这是非常有用的。
像`reduceByWindow`和`reduceByKeyAndWindow`这种窗口操作、`updateStateByKey`这种基于状态的操作，持久化是默认的，不需要开发者调用`persist()`方法。

例如通过网络（如kafka，flume等）获取的输入数据流，默认的持久化策略是复制数据到两个不同的节点以容错。

注意，与RDD不同的是，DStreams默认持久化级别是存储序列化数据到内存中，这将在[性能调优](../performance-tuning/README.md)章节介绍。更多的信息请看[rdd持久化](../../programming-guide/rdds/rdd-persistences.md)