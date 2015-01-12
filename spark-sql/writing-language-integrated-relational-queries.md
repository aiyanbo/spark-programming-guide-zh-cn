# 编写语言集成(Language-Integrated)的相关查询

语言集成的相关查询是实验性的，现在暂时只支持scala。

Spark SQL也支持用领域特定语言编写查询。

```scala
// sc is an existing SparkContext.
val sqlContext = new org.apache.spark.sql.SQLContext(sc)
// Importing the SQL context gives access to all the public SQL functions and implicit conversions.
import sqlContext._
val people: RDD[Person] = ... // An RDD of case class objects, from the first example.

// The following is the same as 'SELECT name FROM people WHERE age >= 10 AND age <= 19'
val teenagers = people.where('age >= 10).where('age <= 19).select('name)
teenagers.map(t => "Name: " + t(0)).collect().foreach(println)
```

DSL使用Scala的符号来表示在潜在表(underlying table)中的列，这些列以前缀(')标示。将这些符号隐式转换成由SQL执行引擎计算的表达式。你可以在[ScalaDoc](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.sql.SchemaRDD)
中了解详情。

