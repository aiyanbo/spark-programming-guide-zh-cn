# 使用键值对

虽然很多 Spark 操作工作在包含任意类型对象的 RDDs 上的，但是少数几个特殊操作仅仅在键值(key-value)对 RDDs 上可用。最常见的是分布式 "shuffle" 操作，例如根据一个 key 对一组数据进行分组和聚合。

在 Scala 中，这些操作在包含[二元组(Tuple2)](http://www.scala-lang.org/api/2.10.4/index.html#scala.Tuple2)(在语言的内建元组中，通过简单的写 (a, b) 创建) 的 RDD 上自动地变成可用的，只要在你的程序中导入 `org.apache.spark.SparkContext._` 来启用 Spark 的隐式转换。在 PairRDDFunctions 的类里键值对操作是可以使用的，如果你导入隐式转换它会自动地包装成元组 RDD。

例如，下面的代码在键值对上使用 `reduceByKey` 操作来统计在一个文件里每一行文本内容出现的次数：

```scala
val lines = sc.textFile("data.txt")
val pairs = lines.map(s => (s, 1))
val counts = pairs.reduceByKey((a, b) => a + b)
```

我们也可以使用 `counts.sortByKey()`，例如，将键值对按照字母进行排序，最后 `counts.collect()` 把它们作为一个对象数组带回到驱动程序。

注意：当使用一个自定义对象作为 key 在使用键值对操作的时候，你需要确保自定义 `equals()` 方法和 `hashCode()` 方法是匹配的。更加详细的内容，查看 [Object.hashCode() 文档](http://docs.oracle.com/javase/7/docs/api/java/lang/Object.html#hashCode())中的契约概述。