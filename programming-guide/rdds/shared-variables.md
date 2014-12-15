# 共享变量

一般情况下，当一个传递给Spark操作(例如map和reduce)的函数在远程节点上面运行时，Spark操作实际上操作的是这个函数所用变量的一个独立副本。这些变量被复制到每台机器上，并且这些变量在远程机器上
的所有更新都不会传递回驱动程序。通常跨任务的读写变量是低效的，但是，Spark还是为两种常见的使用模式提供了两种有限的共享变量：广播变量（broadcast variable）和累加器（accumulator）

## 广播变量

广播变量允许程序员缓存一个只读的变量在每台机器上面，而不是每个任务保存一份拷贝。例如，利用广播变量，我们能够以一种更有效率的方式将一个大数据量输入集合的副本分配给每个节点。（Broadcast variables allow the
programmer to keep a read-only variable cached on each machine rather than shipping a copy of it with tasks.They can be used, for example,
to give every node a copy of a large input dataset in an efficient manner.）Spark也尝试着利用有效的广播算法去分配广播变量，以减少通信的成本。

一个广播变量可以通过调用`SparkContext.broadcast(v)`方法从一个初始变量v中创建。广播变量是v的一个包装变量，它的值可以通过`value`方法访问，下面的代码说明了这个过程：

```scala
 scala> val broadcastVar = sc.broadcast(Array(1, 2, 3))
 broadcastVar: spark.Broadcast[Array[Int]] = spark.Broadcast(b5c40191-a864-4c7d-b9bf-d87e1a4e787c)
 scala> broadcastVar.value
 res0: Array[Int] = Array(1, 2, 3)
```
广播变量创建以后，我们就能够在集群的任何函数中使用它来代替变量v，这样我们就不需要再次传递变量v到每个节点上。另外，为了保证所有的节点得到广播变量具有相同的值，对象v不能在广播之后被修改。

## 累加器

顾名思义，累加器是一种只能通过关联操作进行“加”操作的变量，因此它能够高效的应用于并行操作中。它们能够用来实现`counters`和`sums`。Spark原生支持数值类型的累加器，开发者可以自己添加支持的类型。
如果创建了一个具名的累加器，它可以在spark的UI中显示。这对于理解运行阶段(running stages)的过程有很重要的作用。（注意：这在python中还不被支持）

一个累加器可以通过调用`SparkContext.accumulator(v)`方法从一个初始变量v中创建。运行在集群上的任务可以通过`add`方法或者使用`+=`操作来给它加值。然而，它们无法读取这个值。只有驱动程序可以使用`value`方法来读取累加器的值。
如下的代码，展示了如何利用累加器将一个数组里面的所有元素相加：

```scala
scala> val accum = sc.accumulator(0, "My Accumulator")
accum: spark.Accumulator[Int] = 0
scala> sc.parallelize(Array(1, 2, 3, 4)).foreach(x => accum += x)
...
10/09/29 18:41:08 INFO SparkContext: Tasks finished in 0.317106 s
scala> accum.value
res2: Int = 10
```
这个例子利用了内置的整数类型累加器。开发者可以利用子类[AccumulatorParam](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.AccumulatorParam)创建自己的
累加器类型。AccumulatorParam接口有两个方法：`zero`方法为你的数据类型提供一个“0 值”（zero value）；`addInPlace`方法计算两个值的和。例如，假设我们有一个`Vector`类代表数学上的向量，我们能够
如下定义累加器：

```scala
object VectorAccumulatorParam extends AccumulatorParam[Vector] {
  def zero(initialValue: Vector): Vector = {
    Vector.zeros(initialValue.size)
  }
  def addInPlace(v1: Vector, v2: Vector): Vector = {
    v1 += v2
  }
}
// Then, create an Accumulator of this type:
val vecAccum = sc.accumulator(new Vector(...))(VectorAccumulatorParam)
```
在scala中，Spark支持用更一般的[Accumulable](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.Accumulable)接口来累积数据-结果类型和用于累加的元素类型
不一样（例如通过收集的元素建立一个列表）。Spark也支持用`SparkContext.accumulableCollection`方法累加一般的scala集合类型。
