# 独立应用程序

现在假设我们想要使用 Spark API 写一个独立的应用程序。我们将通过使用 Scala(用 SBT)，Java(用 Maven) 和 Python 写一个简单的应用程序来学习。

我们用 Scala 创建一个非常简单的 Spark 应用程序。如此简单，事实上它的名字叫 `SimpleApp.scala`：

```scala
/* SimpleApp.scala */
import org.apache.spark.SparkContext
import org.apache.spark.SparkContext._
import org.apache.spark.SparkConf

object SimpleApp {
  def main(args: Array[String]) {
    val logFile = "YOUR_SPARK_HOME/README.md" // 应该是你系统上的某些文件
    val conf = new SparkConf().setAppName("Simple Application")
    val sc = new SparkContext(conf)
    val logData = sc.textFile(logFile, 2).cache()
    val numAs = logData.filter(line => line.contains("a")).count()
    val numBs = logData.filter(line => line.contains("b")).count()
    println("Lines with a: %s, Lines with b: %s".format(numAs, numBs))
  }
}
```

这个程序仅仅是在 Spark README 中计算行里面包含 'a' 和包含 'b' 的次数。你需要注意将 `YOUR_SPARK_HOME` 替换成你已经安装 Spark 的路径。不像之前的 Spark Shell 例子，这里初始化了自己的 SparkContext，我们把 SparkContext 初始化作为程序的一部分。

我们通过 SparkContext 的构造函数参入 [SparkConf](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.SparkConf) 对象，这个对象包含了一些关于我们程序的信息。

我们的程序依赖于 Spark API，所以我们需要包含一个 sbt 文件文件，`simple.sbt` 解释了 Spark 是一个依赖。这个文件还要补充 Spark 依赖于一个 repository：

```scala
name := "Simple Project"

version := "1.0"

scalaVersion := "2.10.4"

libraryDependencies += "org.apache.spark" %% "spark-core" % "1.1.0"
```

要让 sbt 正确工作，我们需要把 `SimpleApp.scala` 和 `simple.sbt` 按照标准的文件目录结构布局。上面的做好之后，我们可以把程序的代码创建成一个 JAR 包。然后使用 `spark-submit` 来运行我们的程序。

```
# Your directory layout should look like this
$ find .
.
./simple.sbt
./src
./src/main
./src/main/scala
./src/main/scala/SimpleApp.scala

# Package a jar containing your application
$ sbt package
...
[info] Packaging {..}/{..}/target/scala-2.10/simple-project_2.10-1.0.jar

# Use spark-submit to run your application
$ YOUR_SPARK_HOME/bin/spark-submit \
  --class "SimpleApp" \
  --master local[4] \
  target/scala-2.10/simple-project_2.10-1.0.jar
...
Lines with a: 46, Lines with b: 23
```