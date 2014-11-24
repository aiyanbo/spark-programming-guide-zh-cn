# 概论

在高层中，每个 Spark 应用程序都由一个*驱动程序(driver programe)*构成，驱动程序在集群上运行用户的 `mian` 函数来执行各种各样的*并行操作(parallel operations)*。Spark 的主要抽象是提供一个*弹性分布式数据集(RDD)*，RDD 是指在有并行处理能力的集群节点中被分割(分区)的元素的集合。RDDs 从 Hadoop 的文件系统中的一个文件中创建而来(或其他 Hadoop 支持的文件系统)，或者从一个已有的 Scala 集合转换得到。用户可以要求 Spark 将 RDD *持久化(persist)*到内存中，来让它在并行计算中高效地重用。最后，RDDs 能在节点失败中自动地恢复过来。

Spark 的第二个抽象是*共享变量(shared variables)*，共享变量能被运行在并行计算中。默认情况下，当 Spark 运行一个并行函数时，这个并行函数会作为一个任务集在不同的节点上运行，它会把函数里使用的每个变量都复制搬运到每个任务中。有时，一个变量需要被共享到交叉任务中或驱动程序和任务之间。Spark 支持 2 种类型的共享变量：*广播变量(broadcast variables)*，用来在所有节点的内存中缓存一个值；累加器(accumulators)，仅仅只能执行“添加(added)”操作，例如：记数器(counters)和求和(sums)。

这个指南会在 Spark 支持的所有语言中演示它的每一个特征。非常简单地开始一个 Spark 交互式 shell - `bin/spark-shell` 开始一个 Scala shell，或 `bin/pyspark` 开始一个 Python shell。

* [引入 Spark](linking-with-spark.md)