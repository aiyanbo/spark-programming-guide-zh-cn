# 监控应用程序

除了Spark的监控功能，Spark Streaming增加了一些专有的功能。应用StreamingContext的时候，[Spark web UI](https://spark.apache.org/docs/latest/monitoring.html#web-interfaces)
显示添加的`Streaming`菜单，用以显示运行的receivers（receivers是否是存活状态、接收的记录数、receiver错误等）和完成的批的统计信息（批处理时间、队列等待等待）。这可以用来监控
流应用程序的处理过程。

在WEB UI中的`Processing Time`和`Scheduling Delay`两个度量指标是非常重要的。第一个指标表示批数据处理的时间，第二个指标表示前面的批处理完毕之后，当前批在队列中的等待时间。如果
批处理时间比批间隔时间持续更长或者队列等待时间持续增加，这就预示系统无法以批数据产生的速度处理这些数据，整个处理过程滞后了。在这种情况下，考虑减少批处理时间。

Spark Streaming程序的处理过程也可以通过[StreamingListener](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.scheduler.StreamingListener)接口来监控，这
个接口允许你获得receiver状态和处理时间。注意，这个接口是开发者API，它有可能在未来提供更多的信息。