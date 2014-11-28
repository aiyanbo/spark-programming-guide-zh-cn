# RDD 操作

RDDs 支持 2 种类型的操作：_转换(transformations)_ 从已经存在的数据集中创建一个新的数据集；_动作(actions)_ 在数据集上进行计算之后返回一个值到驱动程序。例如，`map` 是一个转换操作，它将每一个数据集元素传递给一个函数并且返回一个新的 RDD。另一方面，`reduce` 是一个动作，它使用相同的函数来聚合 RDD 的所有元素，并且将最终的结果返回到驱动程序(不过也有一个并行 `reduceByKey` 能返回一个分布式数据集)。

在 Spark 中，所有的转换(transformations)都是惰性(lazy)的，它们不会马上计算它们的结果。相反的，它们仅仅记录转换操作是应用到哪些基础数据集(例如一个文件)上的。转换仅仅在这个时候计算：当动作(action) 需要一个结果返回给驱动程序的时候。这个设计能够让 Spark 运行得更加高效。例如，我们可以实现：通过 `map` 创建一个新数据集在 `reduce` 中使用，并且仅仅返回 `reduce` 的结果给 driver，而不是整个大的映射过的数据集。

默认情况下，每一个转换过的 RDD 会在每次执行动作(action)的时候重新计算一次。然而，你也可以使用 `persist` (或 `cache`)方法持久化(`persist`)一个 RDD 到内存中。在这个情况下，Spark 会在集群上保存相关的元素，在你下次查询的时候会变得更快。在这里也同样支持持久化 RDD 到磁盘，或在多个节点间复制。

## 基础

为了说明 RDD 基本知识，考虑下面的简单程序：

```scala
val lines = sc.textFile("data.txt")
val lineLengths = lines.map(s => s.length)
val totalLength = lineLengths.reduce((a, b) => a + b)
```

第一行是定义来自于外部文件的 RDD。这个数据集并没有加载到内存或做其他的操作：`lines` 仅仅是一个指向文件的指针。第二行是定义 `lineLengths`，它是 `map` 转换(transformation)的结果。同样，`lineLengths` 由于懒惰模式也_没有_立即计算。最后，我们执行 `reduce`，它是一个动作(action)。在这个地方，Spark 把计算分成多个任务(task)，并且让它们运行在多个机器上。每台机器都运行自己的 map 部分和本地 reduce 部分。然后仅仅将结果返回给驱动程序。

如果我们想要再次使用 `lineLengths`，我们可以添加：

```scala
lineLengths.persist()
```

在 `reduce` 之前，它会导致 `lineLengths` 在第一次计算完成之后保存到内存中。