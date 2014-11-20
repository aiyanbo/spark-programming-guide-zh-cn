# 快速开始

本节课程提供一个使用 Spark 的快速介绍，首先我们使用 Spark 的交互式 shell(用 Python 或 Scala) 介绍它的 API。当演示如何在 Java, Scala 和 Python 写独立的程序时，看[编码指南](https://spark.apache.org/docs/latest/programming-guide.html)里完整的参考。

依照这个指南，首先从 [Spark 网站](https://spark.apache.org/downloads.html)下载一个 Spark 发行包。因为我们不会使用 HDFS，你可以下载任何 Hadoop 版本的包。

# 使用 Spark Shell

## 基础

Spark 的 shell 作为一个强大的交互式数据分析工具，提供了一个简单的方式来学习 API。它可以使用 Scala(在 Java 虚拟机上运行现有的 Java 库的一个很好方式) 或 Python。在 Spark 目录里使用下面的方式开始运行：

- Scala

```scala
./bin/spark-shell
```

- Python

```python
./bin/pyspark
```
