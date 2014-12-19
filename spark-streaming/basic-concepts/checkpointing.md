# Checkpointing

一个有状态的操作操作在多批次的数据上面。这些操作包括基于窗口的操作以及`updateStateByKey`操作。因为有状态的操作依赖于操作之前的批数据，所以随着时间的推移它们会不断积累元数据。为了
清除这些元数据，Spark流支持定期的`checkpointing`-通过保存中间数据到HDFS中。需要注意的是，把数据存储到HDFS也会产生开销，这可能是相关批数据占用更长的时间处理。因此，要小心的设置
`checkpointing`的间隔时间。在最小的批(1秒的数据)情况下，每个批都`checkpointing`会明显的减少操作的吞吐量。相反地，`checkpointing`间隔过大会导致lineage和任务数增加，产生不利的
影响。`checkpointing`的间隔设定为DStream滑动间隔的5~10倍是一个可以尝试的选择。

为了`checkpointing`，开发者需要提供存储RDD的HDFS路径。

```scala
ssc.checkpoint(hdfsPath) // assuming ssc is the StreamingContext or JavaStreamingContext
```

DStream的`checkpointing`的时间间隔可以通过如下方式设定：

```scala
dstream.checkpoint(checkpointInterval)
```

对于必须得checkpoint的DStream（通过`updateStateByKey`方法和带有反转（inverse）函数的`reduceByKeyAndWindow`方法创建），checkpoint的时间间隔默认是DStream滑动时间间隔的倍数（至少10秒）。

