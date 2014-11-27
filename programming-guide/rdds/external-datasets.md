## 外部数据集

Spark 可以从任何一个 Hadoop 支持的存储源创建分布式数据集，包括你的本地文件系统，HDFS，Cassandra，HBase，[Amazon S3](http://wiki.apache.org/hadoop/AmazonS3)等。 Spark 支持文本文件(text files)，[SequenceFiles](http://hadoop.apache.org/docs/current/api/org/apache/hadoop/mapred/SequenceFileInputFormat.html) 和其他 Hadoop [InputFormat](http://hadoop.apache.org/docs/stable/api/org/apache/hadoop/mapred/InputFormat.html)。

文本文件 RDDs 可以使用 SparkContext 的 `textFile` 方法创建。 在这个方法里传入文件的 URI (机器上的本地路径或 `hdfs://`，`s3n://` 等)，然后它会将文件读取成一个行集合。这里是一个调用例子：

```scala
scala> val distFile = sc.textFile("data.txt")
distFile: RDD[String] = MappedRDD@1d4cee08
```

一旦创建完成，`distFiile` 就能做数据集操作。例如，我们可以用下面的方式使用 `map` 和 `reduce` 操作将所有行的长度相加：`distFile.map(s => s.length).reduce((a, b) => a + b)`。

注意，Spark 读文件时：

- 如果使用本地文件系统路径，文件必须能在 work 节点上用相同的路径访问到。要么复制文件到所有的 workers，要么使用网络的方式共享文件系统。
- 所有 Spark 的基于文件的方法，包括 `textFile`，能很好地支持文件目录，压缩过的文件和通配符。例如，你可以使用 `textFile("/my/文件目录")`，`textFile("/my/文件目录/*.txt")` 和 `textFile("/my/文件目录/*.gz")`。
- `textFile` 方法也可以选择第二个可选参数来控制切片(_slices_)的数目。默认情况下，Spark 为每一个文件块(HDFS 默认文件块大小是 64M)创建一个切片(_slice_)。但是你也可以通过一个更大的值来设置一个更高的切片数目。注意，你不能设置一个小于文件块数目的切片值。

除了文本文件，Spark 的 Scala API 支持其他几种数据格式：

- `SparkContext.sholeTextFiles` 让你读取一个包含多个小文本文件的文件目录并且返回每一个(filename, content)对。与 `textFile` 的差异是：它记录的是每个文件中的每一行。
- 对于 [SequenceFiles](http://hadoop.apache.org/docs/current/api/org/apache/hadoop/mapred/SequenceFileInputFormat.html)，可以使用 SparkContext 的 `sequenceFile[K, V]` 方法创建，K 和 V 分别对应的是 key 和 values 的类型。像 [IntWritable](http://hadoop.apache.org/docs/current/api/org/apache/hadoop/io/IntWritable.html) 与 [Text](http://hadoop.apache.org/docs/current/api/org/apache/hadoop/io/Text.html) 一样，它们必须是 Hadoop 的 [Writable](http://hadoop.apache.org/docs/current/api/org/apache/hadoop/io/Writable.html) 接口的子类。另外，对于几种通用的 Writables，Spark 允许你指定原声类型来替代。例如： `sequenceFile[Int, String]` 将会自动读取 IntWritables 和 Text。
- 对于其他的 Hadoop InputFormats，你可以使用 `SparkContext.hadoopRDD` 方法，它可以指定任意的 `JobConf`，输入格式(InputFormat)，key 类型，values 类型。你可以跟设置 Hadoop job 一样的方法设置输入源。你还可以在新的 MapReduce 接口(org.apache.hadoop.mapreduce)基础上使用 `SparkContext.newAPIHadoopRDD`(译者注：老的接口是 `SparkContext.newHadoopRDD`)。
- `RDD.saveAsObjectFile` 和 `SparkContext.objectFile` 支持保存一个RDD，保存格式是一个简单的 Java 对象序列化格式。这是一种效率不高的专有格式，如 Avro，它提供了简单的方法来保存任何一个 RDD。