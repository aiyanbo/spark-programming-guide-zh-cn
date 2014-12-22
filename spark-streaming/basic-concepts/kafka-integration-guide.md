# kafka集成指南

[Apache kafka](http://kafka.apache.org/)是一个分布式的发布-订阅消息系统，它可以分布式的、可分区的、可重复提交的方式读写日志数据。下面我们将具体介绍Spark Streaming怎样从kafka中
接收数据。

- 关联：在你的SBT或者Maven项目定义中，引用下面的组件到流应用程序中。

```
 groupId = org.apache.spark
 artifactId = spark-streaming-kafka_2.10
 version = 1.1.1
```

- 编程：在应用程序代码中，引入`FlumeUtils`创建输入DStream。

```scala
 import org.apache.spark.streaming.kafka._
 val kafkaStream = KafkaUtils.createStream(
 	streamingContext, [zookeeperQuorum], [group id of the consumer], [per-topic number of Kafka partitions to consume])
```

有两点需要注意的地方：

   1. kafka的topic分区(partition)和由Spark Streaming生成的RDD分区不相关。所以在`KafkaUtils.createStream()`方法中，增加特定topic的分区数只能够增加单个`receiver`消费这个
    topic的线程数，不能增加Spark处理数据的并发数。

   2. 通过不同的group和topic，可以创建多个输入DStream，从而利用多个`receiver`并发的接收数据。

- 部署：将`spark-streaming-kafka_2.10`和它的依赖（除了`spark-core_2.10`和`spark-streaming_2.10`）打包到应用程序的jar包中。然后用`spark-submit`方法启动你的应用程序。