# Transformations

下面的表格列了 Sparkk 支持的一些常用 transformations。详细内容请参阅 RDD API 文档([Scala](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.rdd.RDD), [Java](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/api/java/JavaRDD.html), [Python](https://spark.apache.org/docs/latest/api/python/pyspark.rdd.RDD-class.html)) 和 PairRDDFunctions 文档([Scala](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.rdd.PairRDDFunctions), [Java](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/api/java/JavaPairRDD.html))。 

Transformation | Meaning
--- | ---
map(_func_) | 返回一个新的分布式数据集，将数据源的每一个元素传递给函数 _func_ 映射组成
filter(_func_) | 返回一个新的数据集，从数据源中选中一些元素通过函数 _func_ 返回 true
flatMap(_func_) | 类似于 map，但是每个输入项能被映射成多个输出项(所以 _func_ 必须返回一个 Seq，而不是单个 item)
mapPartitions(_func_) | 类似于 map，但是分别运行在 RDD 的每个分区上，所以 _func_ 的类型必须是 `Iterator<T> => Iterator<U>` 当运行在类型为 T 的 RDD 上
mapPartitionsWithIndex(_func_) | 类似于 mapPartitions，但是 _func_ 需要提供一个 integer 值描述索引(index)，所以 _func_ 的类型必须是 (Int, Iterator<T>) => Iterator<U> 当运行在类型为 T 的 RDD 上