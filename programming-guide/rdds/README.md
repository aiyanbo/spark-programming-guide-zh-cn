# 弹性分布式数据集 (RDDs)

Spark 围绕的概念是 _Resilient Distributed Dataset (RDD)_：一个可并行操作的有容错机制的数据集合。有 2 种方式创建 RDDs：第一种是在你的驱动程序中并行化一个已经存在的集合；另外一种是引用一个外部存储系统的数据集，例如共享的文件系统，HDFS，HBase或其他 Hadoop 数据格式的数据源。

* [并行集合](parallelized-collections.md)
* [外部数据集](external-datasets.md)
* [RDD 操作](rdd-operations.md)
  * [传递函数到 Spark](passing-functions-to-spark.md)
  * [使用键值对](working-with-key-value-pairs.md)
  * [Transformations](transformations.md)
  * [Actions](actions.md)
* [RDD 持久化](rdd_persistence.md)
