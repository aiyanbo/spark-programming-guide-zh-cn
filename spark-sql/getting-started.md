# 开始

Spark中所有相关功能的入口点是[SQLContext](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.SQLContext)类或者它的子类，
创建一个SQLContext的所有需要仅仅是一个SparkContext。

```scala
val sc: SparkContext // An existing SparkContext.
val sqlContext = new org.apache.spark.sql.SQLContext(sc)

// createSchemaRDD is used to implicitly convert an RDD to a SchemaRDD.
import sqlContext.createSchemaRDD
```
除了一个基本的SQLContext，你也能够创建一个HiveContext，它支持基本SQLContext所支持功能的一个超集。它的额外的功能包括用更完整的HiveQL分析器写查询去访问HiveUDFs的能力、
从Hive表读取数据的能力。用HiveContext你不需要一个已经存在的Hive开启，SQLContext可用的数据源对HiveContext也可用。HiveContext分开打包是为了避免在Spark构建时包含了所有
的Hive依赖。如果对你的应用程序来说，这些依赖不存在问题，Spark 1.2推荐使用HiveContext。以后的稳定版本将专注于为SQLContext提供与HiveContext等价的功能。

用来解析查询语句的特定SQL变种语言可以通过`spark.sql.dialect`选项来选择。这个参数可以通过两种方式改变，一种方式是通过`setConf`方法设定，另一种方式是在SQL命令中通过`SET key=value`
来设定。对于SQLContext，唯一可用的方言是“sql”，它是Spark SQL提供的一个简单的SQL解析器。在HiveContext中，虽然也支持"sql"，但默认的方言是“hiveql”。这是因为HiveQL解析器更
完整。在很多用例中推荐使用“hiveql”。