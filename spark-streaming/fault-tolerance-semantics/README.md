# 容错语义

这一节，我们将讨论在节点错误事件时Spark Streaming的行为。为了理解这些，让我们先记住一些Spark RDD的基本容错语义。

- 一个RDD是不可变的、确定可重复计算的、分布式数据集。每个RDD记住一个确定性操作的谱系(lineage)，这个谱系用在容错的输入数据集上来创建该RDD。
- 如果任何一个RDD的分区因为节点故障而丢失，这个分区可以通过操作谱系从源容错的数据集中重新计算得到。
- 假定所有的RDD transformations是确定的，那么最终转换的数据是一样的，不论Spark机器中发生何种错误。

Spark运行在像HDFS或S3等容错系统的数据上。因此，任何从容错数据而来的RDD都是容错的。然而，这不是在Spark Streaming的情况下，因为Spark Streaming的数据大部分情况下是从
网络中得到的。为了获得生成的RDD相同的容错属性，接收的数据需要重复保存在worker node的多个Spark executor上（默认的复制因子是2），这导致了当出现错误事件时，有两类数据需要被恢复

- Data received and replicated ：在单个worker节点的故障中，这个数据会幸存下来，因为有另外一个节点保存有这个数据的副本。
- Data received but buffered for replication：因为没有重复保存，所以为了恢复数据，唯一的办法是从源中重新读取数据。

有两种错误我们需要关心

- worker节点故障：任何运行executor的worker节点都有可能出故障，那样在这个节点中的所有内存数据都会丢失。如果有任何receiver运行在错误节点，它们的缓存数据将会丢失
- Driver节点故障：如果运行Spark Streaming应用程序的Driver节点出现故障，很明显SparkContext将会丢失，所有执行在其上的executors也会丢失。

## 作为输入源的文件语义（Semantics with files as input source）

如果所有的输入数据都存在于一个容错的文件系统如HDFS，Spark Streaming总可以从任何错误中恢复并且执行所有数据。这给出了一个恰好一次(exactly-once)语义，即无论发生什么故障，
所有的数据都将会恰好处理一次。

## 基于receiver的输入源语义

对于基于receiver的输入源，容错的语义既依赖于故障的情形也依赖于receiver的类型。正如之前讨论的，有两种类型的receiver

- Reliable Receiver：这些receivers只有在确保数据复制之后才会告知可靠源。如果这样一个receiver失败了，缓冲（非复制）数据不会被源所承认。如果receiver重启，源会重发数
据，因此不会丢失数据。
- Unreliable Receiver：当worker或者driver节点故障，这种receiver会丢失数据

选择哪种类型的receiver依赖于这些语义。如果一个worker节点出现故障，Reliable Receiver不会丢失数据，Unreliable Receiver会丢失接收了但是没有复制的数据。如果driver节点
出现故障，除了以上情况下的数据丢失，所有过去接收并复制到内存中的数据都会丢失，这会影响有状态transformation的结果。

为了避免丢失过去接收的数据，Spark 1.2引入了一个实验性的特征`write ahead logs`，它保存接收的数据到容错存储系统中。有了`write ahead logs`和Reliable Receiver，我们可以
做到零数据丢失以及exactly-once语义。

下面的表格总结了错误语义：

Deployment Scenario | Worker Failure | Driver Failure
--- | --- | ---
Spark 1.1 或者更早, 没有write ahead log的Spark 1.2 | 在Unreliable Receiver情况下缓冲数据丢失；在Reliable Receiver和文件的情况下，零数据丢失 | 在Unreliable Receiver情况下缓冲数据丢失；在所有receiver情况下，过去的数据丢失；在文件的情况下，零数据丢失
带有write ahead log的Spark 1.2 | 在Reliable Receiver和文件的情况下，零数据丢失 | 在Reliable Receiver和文件的情况下，零数据丢失

## 输出操作的语义

根据其确定操作的谱系，所有数据都被建模成了RDD，所有的重新计算都会产生同样的结果。所有的DStream transformation都有exactly-once语义。那就是说，即使某个worker节点出现
故障，最终的转换结果都是一样。然而，输出操作（如`foreachRDD`）具有`at-least once`语义，那就是说，在有worker事件故障的情况下，变换后的数据可能被写入到一个外部实体不止一次。
利用`saveAs***Files`将数据保存到HDFS中的情况下，以上写多次是能够被接受的（因为文件会被相同的数据覆盖）。