# 链接

与Spark类似，Spark Streaming也可以利用maven仓库。编写你自己的Spark Streaming程序，你需要引入下面的依赖到你的SBT或者Maven项目中

```maven
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-streaming_2.10</artifactId>
    <version>1.1.1</version>
</dependency>
```
为了从Kafka, Flume和Kinesis这些不在Spark核心API中提供的源获取数据，我们需要添加相关的模块`spark-streaming-xyz_2.10`到依赖中。例如，一些通用的组件如下表所示：

Source | Artifact
--- | ---
Kafka | spark-streaming-kafka_2.10
Flume | spark-streaming-flume_2.10
Kinesis | spark-streaming-kinesis-asl_2.10
Twitter | spark-streaming-twitter_2.10
ZeroMQ | spark-streaming-zeromq_2.10
MQTT | spark-streaming-mqtt_2.10

为了获取最新的列表，请访问[Apache repository](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22org.apache.spark%22%20AND%20v%3A%221.1.1%22)
