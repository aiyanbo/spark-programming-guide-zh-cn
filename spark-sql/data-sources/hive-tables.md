# Hive表

Spark SQL也支持从Apache Hive中读出和写入数据。然而，Hive有大量的依赖，所以它不包含在Spark集合中。可以通过`-Phive`和`-Phive-thriftserver`参数构建Spark，使其
支持Hive。注意这个重新构建的jar包必须存在于所有的worker节点中，因为它们需要通过Hive的序列化和反序列化库访问存储在Hive中的数据。

当和Hive一起工作是，开发者需要提供HiveContext。HiveContext从SQLContext继承而来，它增加了在MetaStore中发现表以及利用HiveSql写查询的功能。没有Hive部署的用户也
可以创建HiveContext。当没有通过`hive-site.xml`配置，上下文将会在当前目录自动地创建`metastore_db`和`warehouse`。

```scala
// sc is an existing SparkContext.
val sqlContext = new org.apache.spark.sql.hive.HiveContext(sc)

sqlContext.sql("CREATE TABLE IF NOT EXISTS src (key INT, value STRING)")
sqlContext.sql("LOAD DATA LOCAL INPATH 'examples/src/main/resources/kv1.txt' INTO TABLE src")

// Queries are expressed in HiveQL
sqlContext.sql("FROM src SELECT key, value").collect().foreach(println)
```