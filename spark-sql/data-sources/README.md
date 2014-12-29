# 数据源

Spark SQL支持通过SchemaRDD接口操作各种数据源。一个SchemaRDD能够作为一个一般的RDD被操作，也可以被注册为一个临时的表。注册一个SchemaRDD为一个表就
可以允许你在其数据上运行SQL查询。这节描述了加载数据为SchemaRDD的多种方法。

* [RDDs](rdds.md)
* [parquet文件](parquet-files.md)