## 并行集合

并行集合 (_Parallelized collections_) 的创建是通过在一个已有的集合(Scala `Seq`)上调用 SparkContext 的 `parallelize` 方法实现的。集合中的元素被复制到一个可并行操作的分布式数据集中。例如，这里演示了如何在一个包含 1 到 5 的数组中创建并行集合：

```scala
val data = Array(1, 2, 3, 4, 5)
val distData = sc.parallelize(data)
```

一旦创建完成，这个分布式数据集(`distData`)就可以被并行操作。例如，我们可以调用 `distData.reduce((a, b) => a + b)` 将这个数组中的元素相加。我们以后再描述在分布式上的一些操作。

并行集合一个很重要的参数是切片数(_slices_)，表示一个数据集切分的份数。Spark 会在集群上为每一个切片运行一个任务。你可以在集群上为每个 CPU 设置 2-4 个切片(slices)。正常情况下，Spark 会试着基于你的集群状况自动地设置切片的数目。然而，你也可以通过 `parallelize` 的第二个参数手动地设置(例如：`sc.parallelize(data, 10)`)。