# 使用 Spark Shell

## 基础

Spark 的 shell 作为一个强大的交互式数据分析工具，提供了一个简单的方式来学习 API。它可以使用 Scala(在 Java 虚拟机上运行现有的 Java 库的一个很好方式) 或 Python。在 Spark 目录里使用下面的方式开始运行：

```scala
./bin/spark-shell
```

Spark 最主要的抽象是叫Resilient Distributed Dataset(RDD) 的弹性分布式集合。RDDs 可以使用 Hadoop InputFormats(例如 HDFS 文件)创建，也可以从其他的 RDDs 转换。让我们在 Spark 源代码目录从 README 文本文件中创建一个新的 RDD。

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

这里，我们结合 [flatMap](), [map]() 和 [reduceByKey]() 来计算文件里每个单词出现的数量，它的结果是包含一组(String, Int) 键值对的 RDD。我们可以使用 [collect] 操作在我们的 shell 中收集单词的数量：

```scala
scala> wordCounts.collect()
res6: Array[(String, Int)] = Array((means,1), (under,2), (this,3), (Because,1), (Python,2), (agree,1), (cluster.,1), ...)
```

## 缓存

Spark 支持把数据集拉到集群内的内存缓存中。当要重复访问时这是非常有用的，例如当我们在一个小的热(hot)数据集中查询，或者运行一个像网页搜索排序这样的重复算法。作为一个简单的例子，让我们把 `linesWithSpark` 数据集标记在缓存中：

```scala
scala> linesWithSpark.cache()
res7: spark.RDD[String] = spark.FilteredRDD@17e51082

scala> linesWithSpark.count()
res8: Long = 15

scala> linesWithSpark.count()
res9: Long = 15
```

缓存 100 行的文本文件来研究 Spark 这看起来很傻。真正让人感兴趣的部分是我们可以在非常大型的数据集中使用同样的函数，甚至在 10 个或者 100 个节点中交叉计算。你同样可以使用 `bin/spark-shell` 连接到一个 cluster 来替换掉[编程指南](https://spark.apache.org/docs/latest/programming-guide.html#initializing-spark)中的方法进行交互操作。