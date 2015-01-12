# RDDs

Spark支持两种方法将存在的RDDs转换为SchemaRDDs。第一种方法使用反射来推断包含特定对象类型的RDD的模式(schema)。在你写spark程序的同时，当你已经知道了模式，这种基于反射的
方法可以使代码更简洁并且程序工作得更好。

创建SchemaRDDs的第二种方法是通过一个编程接口来实现，这个接口允许你构造一个模式，然后在存在的RDDs上使用它。虽然这种方法更冗长，但是它允许你在运行期之前不知道列以及列
的类型的情况下构造SchemaRDDs。

## 利用反射推断模式

Spark SQL的Scala接口支持将包含样本类的RDDs自动转换为SchemaRDD。这个样本类定义了表的模式。

给样本类的参数名字通过反射来读取，然后作为列的名字。样本类可以嵌套或者包含复杂的类型如序列或者数组。这个RDD可以隐式转化为一个SchemaRDD，然后注册为一个表。表可以在后续的
sql语句中使用。

```scala
// sc is an existing SparkContext.
val sqlContext = new org.apache.spark.sql.SQLContext(sc)
// createSchemaRDD is used to implicitly convert an RDD to a SchemaRDD.
import sqlContext.createSchemaRDD

// Define the schema using a case class.
// Note: Case classes in Scala 2.10 can support only up to 22 fields. To work around this limit,
// you can use custom classes that implement the Product interface.
case class Person(name: String, age: Int)

// Create an RDD of Person objects and register it as a table.
val people = sc.textFile("examples/src/main/resources/people.txt").map(_.split(",")).map(p => Person(p(0), p(1).trim.toInt))
people.registerTempTable("people")

// SQL statements can be run by using the sql methods provided by sqlContext.
val teenagers = sqlContext.sql("SELECT name FROM people WHERE age >= 13 AND age <= 19")

// The results of SQL queries are SchemaRDDs and support all the normal RDD operations.
// The columns of a row in the result can be accessed by ordinal.
teenagers.map(t => "Name: " + t(0)).collect().foreach(println)
```

## 编程指定模式

当样本类不能提前确定（例如，记录的结构是经过编码的字符串，或者一个文本集合将会被解析，不同的字段投影给不同的用户），一个SchemaRDD可以通过三步来创建。

- 从原来的RDD创建一个行的RDD
- 创建由一个`StructType`表示的模式与第一步创建的RDD的行结构相匹配
- 在行RDD上通过`applySchema`方法应用模式

```scala
// sc is an existing SparkContext.
val sqlContext = new org.apache.spark.sql.SQLContext(sc)

// Create an RDD
val people = sc.textFile("examples/src/main/resources/people.txt")

// The schema is encoded in a string
val schemaString = "name age"

// Import Spark SQL data types and Row.
import org.apache.spark.sql._

// Generate the schema based on the string of schema
val schema =
  StructType(
    schemaString.split(" ").map(fieldName => StructField(fieldName, StringType, true)))

// Convert records of the RDD (people) to Rows.
val rowRDD = people.map(_.split(",")).map(p => Row(p(0), p(1).trim))

// Apply the schema to the RDD.
val peopleSchemaRDD = sqlContext.applySchema(rowRDD, schema)

// Register the SchemaRDD as a table.
peopleSchemaRDD.registerTempTable("people")

// SQL statements can be run by using the sql methods provided by sqlContext.
val results = sqlContext.sql("SELECT name FROM people")

// The results of SQL queries are SchemaRDDs and support all the normal RDD operations.
// The columns of a row in the result can be accessed by ordinal.
results.map(t => "Name: " + t(0)).collect().foreach(println)
```