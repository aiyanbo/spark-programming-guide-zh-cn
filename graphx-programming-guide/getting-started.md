# 开始

开始的第一步是引入Spark和GraphX到你的项目中，如下面所示

```scala
mport org.apache.spark._
import org.apache.spark.graphx._
// To make some of the examples work we will also need RDD
import org.apache.spark.rdd.RDD
```
如果你没有用到Spark shell，你还将需要SparkContext。