# 提交应用程序

在Spark bin目录下的`spark-submit`可以用来在集群上启动应用程序。它可以通过统一的接口使用Spark支持的所有[集群管理器](https://spark.apache.org/docs/latest/cluster-overview.html#cluster-manager-types)
，所有你不必为每一个管理器做相应的配置。

## 用spark-submit启动应用程序

`bin/spark-submit`脚本负责建立包含Spark以及其依赖的类路径（classpath），它支持不同的集群管理器以及Spark支持的加载模式。

```shell
./bin/spark-submit \
  --class <main-class>
  --master <master-url> \
  --deploy-mode <deploy-mode> \
  --conf <key>=<value> \
  ... # other options
  <application-jar> \
  [application-arguments]
```

一些常用的选项是：

- `--class`：你的应用程序的入口点(如org.apache.spark.examples.SparkPi)
- `--master`：集群的master URL(如spark://23.195.26.187:7077)
- `--deploy-mode`：在worker节点部署你的driver(cluster)或者本地作为外部客户端（client）。默认是client。
- `--conf`：任意的Spark配置属性，格式是key=value。
- `application-jar`：包含应用程序以及其依赖的jar包的路径。这个URL必须在集群中全局可见，例如，存在于所有节点的`hdfs://`路径或`file://`路径
- `application-arguments`：传递给主类的主方法的参数

一个通用的部署策略是从网关集群提交你的应用程序，这个网关机器和你的worker集群物理上协作。在这种设置下，`client`模式是适合的。在`client`模式下，driver直接在`spark-submit`进程
中启动，而这个进程直接作为集群的客户端。应用程序的输入和输出都和控制台相连接。因此，这种模式特别适合涉及REPL的应用程序。

另一种选择，如果你的应用程序从一个和worker机器相距很远的机器上提交，通常情况下用`cluster`模式减少drivers和executors的网络迟延。注意，`cluster`模式目前不支持独立集群、
mesos集群以及python应用程序。

有几个我们使用的集群管理器特有的可用选项。例如，在Spark独立集群的`cluster`模式下，你也可以指定`--supervise`用来确保driver自动重启（如果它因为非零退出码失败）。
为了列举spark-submit所有的可用选项，用`--help`运行它。

```shell
# Run application locally on 8 cores
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master local[8] \
  /path/to/examples.jar \
  100

# Run on a Spark Standalone cluster in client deploy mode
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master spark://207.184.161.138:7077 \
  --executor-memory 20G \
  --total-executor-cores 100 \
  /path/to/examples.jar \
  1000

# Run on a Spark Standalone cluster in cluster deploy mode with supervise
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master spark://207.184.161.138:7077 \
  --deploy-mode cluster
  --supervise
  --executor-memory 20G \
  --total-executor-cores 100 \
  /path/to/examples.jar \
  1000

# Run on a YARN cluster
export HADOOP_CONF_DIR=XXX
./bin/spark-submit \
  --class org.apache.spark.examples.SparkPi \
  --master yarn-cluster \  # can also be `yarn-client` for client mode
  --executor-memory 20G \
  --num-executors 50 \
  /path/to/examples.jar \
  1000

# Run a Python application on a Spark Standalone cluster
./bin/spark-submit \
  --master spark://207.184.161.138:7077 \
  examples/src/main/python/pi.py \
  1000
```

## Master URLs

传递给Spark的url可以用下面的模式

Master URL | Meaning
--- | ---
local | 用一个worker线程本地运行Spark
local[K] | 用k个worker线程本地运行Spark(理想情况下，设置这个值为你的机器的核数)
local[*] | 用尽可能多的worker线程本地运行Spark
spark://HOST:PORT | 连接到给定的Spark独立部署集群master。端口必须是master配置的端口，默认是7077
mesos://HOST:PORT | 连接到给定的mesos集群
yarn-client | 以`client`模式连接到Yarn集群。群集位置将基于通过HADOOP_CONF_DIR变量找到
yarn-cluster | 以`cluster`模式连接到Yarn集群。群集位置将基于通过HADOOP_CONF_DIR变量找到
