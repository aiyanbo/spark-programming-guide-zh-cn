# 开始翻滚吧!

祝贺你成功运行你的第一个 Spark 应用程序!

- 要深入了解 API，可以从[Spark编程指南](https://spark.apache.org/docs/latest/programming-guide.html)开始，或者从其他的组件开始，例如：Spark Streaming。
- 要让程序运行在集群(cluster)上，前往[部署概论](https://spark.apache.org/docs/latest/cluster-overview.html)。
- 最后，Spark 在 `examples` 文件目录里包含了 [Scala](https://github.com/apache/spark/tree/master/examples/src/main/scala/org/apache/spark/examples), [Java](https://github.com/apache/spark/tree/master/examples/src/main/java/org/apache/spark/examples) 和 [Python](https://github.com/apache/spark/tree/master/examples/src/main/python) 的几个简单的例子，你可以直接运行它们：

```
# For Scala and Java, use run-example:
./bin/run-example SparkPi

# For Python examples, use spark-submit directly:
./bin/spark-submit examples/src/main/python/pi.py
```