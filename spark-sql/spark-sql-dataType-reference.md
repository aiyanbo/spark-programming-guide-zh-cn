# Spark SQL数据类型

- 数字类型
    - ByteType：代表一个字节的整数。范围是-128到127
    - ShortType：代表两个字节的整数。范围是-32768到32767
    - IntegerType：代表4个字节的整数。范围是-2147483648到2147483647
    - LongType：代表8个字节的整数。范围是-9223372036854775808到9223372036854775807
    - FloatType：代表4字节的单精度浮点数
    - DoubleType：代表8字节的双精度浮点数
    - DecimalType：代表任意精度的10进制数据。通过内部的java.math.BigDecimal支持。BigDecimal由一个任意精度的整型非标度值和一个32位整数组成
    - StringType：代表一个字符串值
    - BinaryType：代表一个byte序列值
    - BooleanType：代表boolean值
    - Datetime类型
        - TimestampType：代表包含字段年，月，日，时，分，秒的值
        - DateType：代表包含字段年，月，日的值
    - 复杂类型
        - ArrayType(elementType, containsNull)：代表由elementType类型元素组成的序列值。`containsNull`用来指明`ArrayType`中的值是否有null值
        - MapType(keyType, valueType, valueContainsNull)：表示包括一组键 - 值对的值。通过keyType表示key数据的类型，通过valueType表示value数据的类型。`valueContainsNull`用来指明`MapType`中的值是否有null值
        - StructType(fields):表示一个拥有`StructFields (fields)`序列结构的值
            - StructField(name, dataType, nullable):代表`StructType`中的一个字段，字段的名字通过`name`指定，`dataType`指定field的数据类型，`nullable`表示字段的值是否有null值。

Spark的所有数据类型都定义在包`org.apache.spark.sql`中，你可以通过`import  org.apache.spark.sql._`访问它们。

数据类型 | Scala中的值类型 | 访问或者创建数据类型的API
--- | --- | ---
ByteType | Byte | ByteType
ShortType | Short | ShortType
IntegerType | Int | IntegerType
LongType | Long | LongType
FloatType | Float | FloatType
DoubleType | Double | DoubleType
DecimalType | scala.math.BigDecimal | DecimalType
StringType | String | StringType
BinaryType | Array[Byte] | BinaryType
BooleanType | Boolean | BooleanType
TimestampType | java.sql.Timestamp | TimestampType
DateType | java.sql.Date | DateType
ArrayType | scala.collection.Seq | ArrayType(elementType, [containsNull]) 注意containsNull默认为true
MapType | scala.collection.Map | MapType(keyType, valueType, [valueContainsNull]) 注意valueContainsNull默认为true
StructType | org.apache.spark.sql.Row | StructType(fields) ，注意fields是一个StructField序列，相同名字的两个StructField不被允许
StructField | The value type in Scala of the data type of this field (For example, Int for a StructField with the data type IntegerType) | StructField(name, dataType, nullable)