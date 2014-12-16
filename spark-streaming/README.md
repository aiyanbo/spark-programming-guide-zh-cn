# Spark Streaming

Spark streaming是Spark核心API的一个扩展，它对实时流式数据的处理具有可扩展性、高吞吐量、可容错性等特点。我们可以从kafka、flume、Twitter、 ZeroMQ、Kinesis等源获取数据，也可以通过由
高阶函数map、reduce、join、window等组成的复杂算法计算出数据。最后，处理后的数据可以推送到文件系统、数据库、实时仪表盘中。事实上，你可以将处理后的数据应用到Spark的[机器学习算法](https://spark.apache.org/docs/latest/mllib-guide.html)、
[图处理算法](https://spark.apache.org/docs/latest/graphx-programming-guide.html)中去。

![Spark Streaming处理流程](../img/streaming-arch.png)

在内部，它的工作原理如下图所示。Spark Streaming接收实时的输入数据流，然后将这些数据切分为批数据供Spark引擎处理，Spark引擎将数据生成最终的结果数据。

![Spark Streaming处理原理](../img/streaming-flow.png)

Spark Streaming支持一个高层的抽象，叫做离散流(`discretized stream`)或者`DStream`，它代表连续的数据流。DStream既可以利用从Kafka, Flume和Kinesis等源获取的输入数据流创建，也可以
在其他DStream的基础上通过高阶函数获得。在内部，DStream是由一系列RDDs组成。

本指南指导用户开始利用DStream编写Spark Streaming程序。用户能够利用scala或者是java来编写Spark Streaming程序。

* [一个快速的例子](a-quick-example.md)
* [基本概念](basic-concepts/README.md)
  * [链接](basic-concepts/linking.md)
