# Parquet文件

Parquet是一种柱状(columnar)格式，可以被许多其它的数据处理系统支持。Spark SQL提供支持读和写Parquet文件的功能，这些文件可以自动地保留原始数据的模式。

## 加载数据

```scala
// sqlContext from the previous example is used in this example.
// createSchemaRDD is used to implicitly convert an RDD to a SchemaRDD.
import sqlContext.createSchemaRDD

val people: RDD[Person] = ... // An RDD of case class objects, from the previous example.

// The RDD is implicitly converted to a SchemaRDD by createSchemaRDD, allowing it to be stored using Parquet.
people.saveAsParquetFile("people.parquet")

// Read in the parquet file created above.  Parquet files are self-describing so the schema is preserved.
// The result of loading a Parquet file is also a SchemaRDD.
val parquetFile = sqlContext.parquetFile("people.parquet")

//Parquet files can also be registered as tables and then used in SQL statements.
parquetFile.registerTempTable("parquetFile")
val teenagers = sqlContext.sql("SELECT name FROM parquetFile WHERE age >= 13 AND age <= 19")
teenagers.map(t => "Name: " + t(0)).collect().foreach(println)
```

## 配置

可以在SQLContext上使用setConf方法配置Parquet或者在用SQL时运行`SET key=value`命令。

Property Name | Default | Meaning
--- | --- | ---
spark.sql.parquet.binaryAsString | false | 一些其它的Parquet-producing系统，特别是Impala和其它版本的Spark SQL，当写出Parquet模式的时候，二进制数据和字符串之间无法区分。
这个标记告诉Spark SQL将二进制数据解释为字符串来提供这些系统的兼容性。

spark.sql.parquet.cacheMetadata | true | 打开parquet元数据的缓存，可以提高静态数据的查询速度
spark.sql.parquet.compression.codec | gzip | 设置写parquet文件时的压缩算法，可以接受的值包括：uncompressed, snappy, gzip, lzo
spark.sql.parquet.filterPushdown | false | 打开Parquet过滤器的pushdown优化。因为已知的Paruet错误，这个特征默认是关闭的。如果你的表不包含任何空的字符串或者二进制列，打开这个特征仍是安全的
spark.sql.hive.convertMetastoreParquet | true | 当设置为false时，Spark SQL将使用Hive SerDe代替内置的支持
