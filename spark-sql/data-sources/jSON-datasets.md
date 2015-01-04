# JSON数据集

Spark SQL能够自动推断JSON数据集的模式，加载它为一个SchemaRDD。这种转换可以通过下面两种方法来实现

- jsonFile ：从一个包含JSON文件的目录中加载。文件中的每一行是一个JSON对象
- jsonRDD ：从存在的RDD加载数据，这些RDD的每个元素是一个包含JSON对象的字符串

注意，作为jsonFile的文件不是一个典型的JSON文件，每行必须是独立的并且包含一个有效的JSON对象。结果是，一个多行的JSON文件经常会失败

```scala
// sc is an existing SparkContext.
val sqlContext = new org.apache.spark.sql.SQLContext(sc)

// A JSON dataset is pointed to by path.
// The path can be either a single text file or a directory storing text files.
val path = "examples/src/main/resources/people.json"
// Create a SchemaRDD from the file(s) pointed to by path
val people = sqlContext.jsonFile(path)

// The inferred schema can be visualized using the printSchema() method.
people.printSchema()
// root
//  |-- age: integer (nullable = true)
//  |-- name: string (nullable = true)

// Register this SchemaRDD as a table.
people.registerTempTable("people")

// SQL statements can be run by using the sql methods provided by sqlContext.
val teenagers = sqlContext.sql("SELECT name FROM people WHERE age >= 13 AND age <= 19")

// Alternatively, a SchemaRDD can be created for a JSON dataset represented by
// an RDD[String] storing one JSON object per string.
val anotherPeopleRDD = sc.parallelize(
  """{"name":"Yin","address":{"city":"Columbus","state":"Ohio"}}""" :: Nil)
val anotherPeople = sqlContext.jsonRDD(anotherPeopleRDD)
```

