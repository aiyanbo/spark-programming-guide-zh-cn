# 概论

在高层中，每个 Spark 应用程序都由一个*驱动程序(driver programe)*构成，驱动程序在集群上运行用户的 `mian` 函数来执行各种各样的*并行操作(parallel operations)*。Spark 的主要抽象是提供一个*弹性分布式数据集(RDD)*，RDD 是指在有并行处理能力的集群节点中被分割(分区)的元素的集合。RDDs 从 Hadoop 的文件系统中的一个文件中创建而来(或其他 Hadoop 支持的文件系统)，或者从一个已有的 Scala 集合转换得到。用户可以要求 Spark 将 RDD *持久化(persist)*到内存中，来让它在并行计算中高效地重用。最后，RDDs 能在节点失败中自动地恢复过来。

