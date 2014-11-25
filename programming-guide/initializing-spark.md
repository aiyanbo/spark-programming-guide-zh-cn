# 初始化 Spark

Spark 编程的第一步是需要创建一个 [SparkContext](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.SparkContext) 对象，用来告诉 Spark 如何访问集群。在创建 `SparkContext` 之前，你需要构建一个 [SparkConf](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.SparkConf) 对象， SparkConf 对象包含了一些你应用程序的信息。

```scala
val conf = new SparkConf().setAppName(appName).setMaster(master)
new SparkContext(conf)
```

`appName` 参数是你程序的名字，它会显示在 cluster UI 上。`master` 是 [Spark, Mesos 或 YARN 集群的 URL](https://spark.apache.org/docs/latest/submitting-applications.html#master-urls)，或运行在本地模式时，使用专用字符串 “local”。在实践中，当应用程序运行在一个集群上时，你并不想要把 `master` 硬编码到你的程序中，你可以[用 spark-submit 启动你的应用程序](https://spark.apache.org/docs/latest/submitting-applications.html)的时候传递它。然而，你可以在本地测试和单元测试中使用 “local” 运行 Spark 进程。

## 使用 Shell

在 Spark shell 中，有一个专有的 SparkContext 已经为你创建好。在变量中叫做 `sc`。你自己创建的 SparkContext 将无法工作。可以用 `--master` 参数来设置 SparkContext 要连接的集群，用 `--jars` 来设置需要添加到 classpath 中的 JAR 包，如果有多个 JAR 包使用**逗号**分割符连接它们。例如：在一个拥有 4 核的环境上运行 `bin/spark-shell`，使用：

```
$ ./bin/spark-shell --master local[4]
```

或在 classpath 中添加 `code.jar`，使用：

```
$ ./bin/spark-shell --master local[4] --jars code.jar
```

执行 `spark-shell --help` 获取完整的选项列表。在这之后，调用 `spark-shell` 会比 [spark-submit 脚本](https://spark.apache.org/docs/latest/submitting-applications.html)更为普遍。