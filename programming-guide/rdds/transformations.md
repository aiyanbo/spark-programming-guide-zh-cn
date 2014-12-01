# Transformations

下面的表格列了 Sparkk 支持的一些常用 transformations。详细内容请参阅 RDD API 文档([Scala](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.rdd.RDD), [Java](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/api/java/JavaRDD.html), [Python](https://spark.apache.org/docs/latest/api/python/pyspark.rdd.RDD-class.html)) 和 PairRDDFunctions 文档([Scala](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.rdd.PairRDDFunctions), [Java](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/api/java/JavaPairRDD.html))。 

Transformation | Meaning
--- | ---
map(_func_) | 返回一个新的分布式数据集，将数据源的每一个元素传递给函数 _func_ 映射组成。
filter(_func_) | 返回一个新的数据集，从数据源中选中一些元素通过函数 _func_ 返回 true。
flatMap(_func_) | 类似于 map，但是每个输入项能被映射成多个输出项(所以 _func_ 必须返回一个 Seq，而不是单个 item)。
mapPartitions(_func_) | 类似于 map，但是分别运行在 RDD 的每个分区上，所以 _func_ 的类型必须是 `Iterator<T> => Iterator<U>` 当运行在类型为 T 的 RDD 上。
mapPartitionsWithIndex(_func_) | 类似于 mapPartitions，但是 _func_ 需要提供一个 integer 值描述索引(index)，所以 _func_ 的类型必须是 (Int, Iterator<T>) => Iterator<U> 当运行在类型为 T 的 RDD 上。
sample(withReplacement, fraction, seed) | 对数据进行采样。
union(otherDataset) | Return a new dataset that contains the union of the elements in the source dataset and the argument.
intersection(otherDataset) | Return a new RDD that contains the intersection of elements in the source dataset and the argument.
distinct([numTasks])) | Return a new dataset that contains the distinct elements of the source dataset.
groupByKey([numTasks]) | When called on a dataset of (K, V) pairs, returns a dataset of (K, Iterable<V>) pairs. 
Note: If you are grouping in order to perform an aggregation (such as a sum or average) over each key, using reduceByKey or combineByKey will yield much better performance. 
Note: By default, the level of parallelism in the output depends on the number of partitions of the parent RDD. You can pass an optional numTasks argument to set a different number of tasks.
reduceByKey(func, [numTasks]) | When called on a dataset of (K, V) pairs, returns a dataset of (K, V) pairs where the values for each key are aggregated using the given reduce function func, which must be of type (V,V) => V. Like in groupByKey, the number of reduce tasks is configurable through an optional second argument.
aggregateByKey(zeroValue)(seqOp, combOp, [numTasks]) | When called on a dataset of (K, V) pairs, returns a dataset of (K, U) pairs where the values for each key are aggregated using the given combine functions and a neutral "zero" value. Allows an aggregated value type that is different than the input value type, while avoiding unnecessary allocations. Like in groupByKey, the number of reduce tasks is configurable through an optional second argument.
sortByKey([ascending], [numTasks]) | When called on a dataset of (K, V) pairs where K implements Ordered, returns a dataset of (K, V) pairs sorted by keys in ascending or descending order, as specified in the boolean ascending argument.
join(otherDataset, [numTasks]) | When called on datasets of type (K, V) and (K, W), returns a dataset of (K, (V, W)) pairs with all pairs of elements for each key. Outer joins are also supported through leftOuterJoin and rightOuterJoin.
cogroup(otherDataset, [numTasks]) | When called on datasets of type (K, V) and (K, W), returns a dataset of (K, Iterable<V>, Iterable<W>) tuples. This operation is also called groupWith.
cartesian(otherDataset) | When called on datasets of types T and U, returns a dataset of (T, U) pairs (all pairs of elements).
pipe(command, [envVars]) | Pipe each partition of the RDD through a shell command, e.g. a Perl or bash script. RDD elements are written to the process's stdin and lines output to its stdout are returned as an RDD of strings.
coalesce(numPartitions) | Decrease the number of partitions in the RDD to numPartitions. Useful for running operations more efficiently after filtering down a large dataset.
repartition(numPartitions) | Reshuffle the data in the RDD randomly to create either more or fewer partitions and balance it across them. This always shuffles all data over the network.
