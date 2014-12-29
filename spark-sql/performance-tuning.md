# 性能调优

对于某些工作负载，可以在通过在内存中缓存数据或者打开一些实验选项来提高性能。

## 在内存中缓存数据

Spark SQL可以通过调用`sqlContext.cacheTable("tableName")`方法来缓存使用柱状格式的表。然后，Spark将会仅仅浏览需要的列并且自动地压缩数据以减少内存的使用以及垃圾回收的
压力。你可以通过调用`sqlContext.uncacheTable("tableName")`方法在内存中删除表。

注意，如果你调用`schemaRDD.cache()`而不是`sqlContext.cacheTable(...)`,表将不会用柱状格式来缓存。在这种情况下，`sqlContext.cacheTable(...)`是强烈推荐的用法。

可以在SQLContext上使用setConf方法或者在用SQL时运行`SET key=value`命令来配置内存缓存。

Property Name | Default | Meaning
--- | --- | ---
spark.sql.inMemoryColumnarStorage.compressed | true | 当设置为true时，Spark SQL将为基于数据统计信息的每列自动选择一个压缩算法。
spark.sql.inMemoryColumnarStorage.batchSize | 10000 | 柱状缓存的批数据大小。更大的批数据可以提高内存的利用率以及压缩效率，但有OOMs的风险

## 其它的配置选项

以下的选项也可以用来调整查询执行的性能。有可能这些选项会在以后的版本中弃用，这是因为更多的优化会自动执行。

Property Name | Default | Meaning
--- | --- | ---
spark.sql.autoBroadcastJoinThreshold | 10485760(10m) | 配置一个表的最大大小(byte)。当执行join操作时，这个表将会广播到所有的worker节点。可以将值设置为-1来禁用广播。注意，目前的统计数据只支持Hive Metastore表，命令`ANALYZE TABLE <tableName> COMPUTE STATISTICS noscan`已经在这个表中运行。
spark.sql.codegen | false | 当为true时，特定查询中的表达式求值的代码将会在运行时动态生成。对于一些拥有复杂表达式的查询，此选项可导致显著速度提升。然而，对于简单的查询，这个选项会减慢查询的执行
spark.sql.shuffle.partitions | 200 | 配置join或者聚合操作shuffle数据时分区的数量



