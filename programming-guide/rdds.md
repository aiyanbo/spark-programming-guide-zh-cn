# Resilient Distributed Datasets (RDDs)

Spark 围绕 _Resilient Distributed Dataset (RDD)_ 是一个可并行操作的可容错的数据集合的概念。有 2 种方式创建 RDDs：第一种是在你的驱动程序中并行化一个已经存在的集合；另外一种是引用一个外部存储系统的数据集，例如共享的文件系统，HDFS，HBase或其他 Hadoop 数据格式的数据源。

## 并行集合

并行集合 (_Parallelized collections_) 的创建是通过在一个已有的集合(Scala `Seq`)上调用 SparkContext 的 `parallelize` 方法实现的。集合中的元素被复制到一个可并行操作的分布式数据集中。例如，这里演示了如何在一个包含 1 到 5 的数组中创建并行集合：

```scala
val data = Array(1, 2, 3, 4, 5)
val distData = sc.parallelize(data)
```

一旦创建，这个分布式数据集(`distData`)就可以被并行操作。例如，我们可以调用 `distData.reduce((a, b) => a + b)` 将这个数组中的元素相加。我们以后再描述在分布式上的一些操作。

并行集合一个很重要的参数是将一个数据集切分的切片(_slices_)数。Spark 运行一个任务时会遍历集群上的每一个切片。通常地你想要在你的集群里为每个 CPU 设置 2-4 个切片(slices)。正常情况下，Spark 会试着基于你的集群自动地设置切片的数量。然而，你也可以通过 `parallelize` 的第二个参数手动地设置它(例如：`sc.parallelize(data, 10)`)。

## 外部数据集

Spark 可以从任何一个 Hadoop 支持的存储源创建分布式数据集，包括你的本地文件系统，HDFS，Cassandra，HBase，[Amazon S3](http://wiki.apache.org/hadoop/AmazonS3)等。 Spark 支持文本文件(text files)，[SequenceFiles](http://hadoop.apache.org/docs/current/api/org/apache/hadoop/mapred/SequenceFileInputFormat.html) 和其他 Hadoop [InputFormat](http://hadoop.apache.org/docs/stable/api/org/apache/hadoop/mapred/InputFormat.html)。

文本文件 RDDs 可以使用 SparkContext 的 `textFile` 方法创建。 在这个方法里传入文件的 URI (机器上的本地路径或 `hdfs://`，`s3n://` 等)，然后它会将文件读取成一个行集合。这里是一个调用例子：

```scala
scala> val distFile = sc.textFile("data.txt")
distFile: RDD[String] = MappedRDD@1d4cee08
```

一旦创建，`distFiile` 就能做数据集操作。例如，我们可以用下面的方式使用 `map` 和 `reduce` 操作统计所有行的大小：`distFile.map(s => s.length).reduce((a, b) => a + b)`。

注意，Spark 读文件时：

- 如果使用本地文件系统路径，文件必须能在 work 节点上用相同的路径访问到。要么复制文件到所有的 workers，要么使用网络的方式共享文件系统。
- 所有 Spark 的基于文件的方法，包括 `textFile`，能很好地支持文件目录，压缩过的文件和通配符。例如，你可以使用 `textFile("/my/文件目录")`，`textFile("/my/文件目录/*.txt")` 和 `textFile("/my/文件目录/*.gz")`。
- `textFile` 方法也可以选择第二个可选参数来控制切片(_slices_)的个数。默认情况下，Spark 为每一个文件块(HDFS 默认文件块大小是 64M)创建一个切片(_slice_)。但是你也可以通过一个更大的值设置一个高的切片个数。注意你不能设置一个小于文件块个数的切片数。

除了文本文件，Spark 的 Scala API 支持其他几种数据格式：

- `SparkContext.sholeTextFiles` 让你读取一个包含多个小文本文件的文件目录并且返回每一个(filename, content)对。对比 `textFile` 是返回每个文件中每一行的记录。
- ...