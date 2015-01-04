# 与Apache Hive的兼容性

Spark SQL设计得和Hive元存储(Metastore)、序列化反序列化(SerDes)、用户自定义函数(UDFs)相兼容。当前的Spark SQL以Hive 0.12.0和0.13.1为基础。

## 部署存在的Hive仓库

Spark SQL Thrift JDBC服务器按照“开箱即用(out of the box)”的方式设计得和存在的Hive相兼容。你不必修改存在的Hive仓库或者改变数据的位置或者分割表。

## 支持的Hive特征

Spark SQL支持绝大部分的Hive特征。

- Hive查询语句
    - SELECT
    - GROUP BY
    - ORDER BY
    - CLUSTER BY
    - SORT BY
- 所有的Hive操作
    - 关系运算符(=, ⇔, ==, <>, <, >, >=, <=等)
    - 算术运算符(+, -, *, /, %等)
    - 逻辑运算符(AND, &&, OR, ||)
    - 复杂的类型构造函数
    - 数学函数(sign, ln, cos等)
    - 字符串函数(instr, length, printf等)
- 用户自定义函数
- 用户自定义聚合函数(UDAF)
- 用户自定义序列化格式
- Joins
    - JOIN
    - {LEFT|RIGHT|FULL} OUTER JOIN
    - LEFT SEMI JOIN
    - CROSS JOIN
- Unions
- 子查询
    - SELECT col FROM ( SELECT a + b AS col from t1) t2
- 采样
- Explain
- Partitioned表
- 视图
- 所有的Hive DDL函数
    - CREATE TABLE
    - CREATE TABLE AS SELECT
    - ALTER TABLE
- 大部分的Hive数据类型
    - TINYINT
    - SMALLINT
    - INT
    - BIGINT
    - BOOLEAN
    - FLOAT
    - DOUBLE
    - STRING
    - BINARY
    - TIMESTAMP
    - DATE
    - ARRAY<>
    - MAP<>
    - STRUCT<>

