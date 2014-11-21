# 快速开始

本节课程提供一个使用 Spark 的快速介绍，首先我们使用 Spark 的交互式 shell(用 Python 或 Scala) 介绍它的 API。当演示如何在 Java, Scala 和 Python 写独立的程序时，看[编码指南](https://spark.apache.org/docs/latest/programming-guide.html)里完整的参考。

依照这个指南，首先从 [Spark 网站](https://spark.apache.org/downloads.html)下载一个 Spark 发行包。因为我们不会使用 HDFS，你可以下载任何 Hadoop 版本的包。

# 使用 Spark Shell

## 基础

Spark 的 shell 作为一个强大的交互式数据分析工具，提供了一个简单的方式来学习 API。它可以使用 Scala(在 Java 虚拟机上运行现有的 Java 库的一个很好方式) 或 Python。在 Spark 目录里使用下面的方式开始运行：

```scala
./bin/spark-shell
```

Spark 最主要的抽象是叫Resilient Distributed Dataset(RDD) 的分布式集合。RDDs 可以使用 Hadoop InputFormats(例如 HDFS 文件)创建，也可以从其他的 RDDs 转换。让我们在 Spark 源代码目录从 README 文本文件中创建一个新的 RDD。

```scala
scala> val textFile = sc.textFile("README.md")
textFile: spark.RDD[String] = spark.MappedRDD@2ee9b6e3
```

RDD 的 [actions](https://spark.apache.org/docs/latest/programming-guide.html#actions) 从 RDD 中返回值，[transformations](https://spark.apache.org/docs/latest/programming-guide.html#transformations) 可以转换成一个新 RDD 并返回它的引用。让我们开始使用几个操作：

```scala
scala> textFile.count() // RDD 的数据条数
res0: Long = 126

scala> textFile.first() // RDD 的第一行数据
res1: String = # Apache Spark
```

现在让我们使用一个 transformation，我们将使用 [filter](https://spark.apache.org/docs/latest/programming-guide.html#transformations) 在这个文件里返回一个包含子数据集的新 RDD。

```scala
scala> val linesWithSpark = textFile.filter(line => line.contains("Spark"))
linesWithSpark: spark.RDD[String] = spark.FilteredRDD@7dd4af09
```

我们可以把 actions 和 transformations 链接在一起：

```scala
scala> textFile.filter(line => line.contains("Spark")).count() // 有多少行包括 "Spark"?
res3: Long = 15
```

## 更多 RDD 操作

RDD actions 和 transformations 能被用在更多的复杂计算中。比方说，我们想要找到一行中最多的单词数量：

```scala
scala> textFile.map(line => line.split(" ").size).reduce((a, b) => if (a > b) a else b)
res4: Long = 15
```

首先将行映射成一个整型数值产生一个新 RDD。 在这个新的 RDD 上调用 `reduce` 找到行中最大的个数。 `map` 和 `reduce` 的参数是 Scala 的函数串(闭包)，并且可以使用任何语言特性或者 Scala/Java 类库。例如，我们可以很方便地调用其他的函数声明。 我们使用 `Math.max()` 函数让代码更容易理解：

```scala
scala> import java.lang.Math
import java.lang.Math

scala> textFile.map(line => line.split(" ").size).reduce((a, b) => Math.max(a, b))
res5: Int = 15
```

Hadoop 流行的一个通用的数据流模式是 MapReduce。Spark 能很容易地实现 MapReduce：

```scala
scala> val wordCounts = textFile.flatMap(line => line.split(" ")).map(word => (word, 1)).reduceByKey((a, b) => a + b)
wordCounts: spark.RDD[(String, Int)] = spark.ShuffledAggregatedRDD@71f027b8
```
