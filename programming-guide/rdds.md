# Resilient Distributed Datasets (RDDs)

Spark 围绕 _Resilient Distributed Dataset (RDD)_ 是一个可并行操作的可容错的数据集合的概念。有 2 种方式创建 RDDs：第一种是在你的驱动程序中并行化一个已经存在的集合；另外一种是引用一个外部存储系统的数据集，例如共享的文件系统，HDFS，HBase或其他 Hadoop 数据格式的数据源。

## 并行集合

并行集合 (_Parallelized collections_) 的创建是通过在一个已有的集合(Scala `Seq`)上调用 SparkContext 的 `parallelize` 方法实现的。集合中的元素被复制到一个可并行操作的分布式数据集中。例如，这里演示了如何在一个包含 1 到 5 的数组中创建并行集合：

```scala
val data = Array(1, 2, 3, 4, 5)
val distData = sc.parallelize(data)
```

一旦被创建，这个分布式数据集(`distData`)就可以被并行操作。例如，我们可以调用 `distData.reduce((a, b) => a + b)` 将这个数组中的元素相加。我们以后再描述在分布式上的一些操作。

并行集合一个很重要的参数是将一个数据集切分的切片(_slices_)数。Spark 运行一个任务时会遍历集群上的每一个切片。通常地你想要在你的集群里根据每个 CPU 设置 2-4 个切片(slices)。正常情况下，Spark 会试着基于你的集群自动地设置切片的数量。然而，你也可以通过 `parallelize` 的第二个参数手动地设置它(例如：`sc.parallelize(data, 10)`)。

## 外部数据集

Spark 可以从任何一个 Hadoop 支持的存储源创建分布式数据集，包括你的本地文件系统，HDFS，Cassandra，HBase，[Amazon S3](http://wiki.apache.org/hadoop/AmazonS3)等。 Spark 支持文本文件(text files)，[SequenceFiles](http://hadoop.apache.org/docs/current/api/org/apache/hadoop/mapred/SequenceFileInputFormat.html) 和其他 Hadoop [InputFormat](http://hadoop.apache.org/docs/stable/api/org/apache/hadoop/mapred/InputFormat.html)。

文本文件 RDDs 可以使用 SparkContext 的 `textFile` 方法创建。 在这个方法里传入文件的 URI (机器上的本地路径或 `hdfs://`，`s3n://` 等)，然后它会将文件读取成一个行集合。这里是一个调用例子：

```scala
scala> val distFile = sc.textFile("data.txt")
distFile: RDD[String] = MappedRDD@1d4cee08
```

