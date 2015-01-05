# 在YARN上运行Spark

## 配置

大部分为`Spark on YARN`模式提供的配置与其它部署模式提供的配置相同。下面这些是为`Spark on YARN`模式提供的配置。

### Spark属性

Property Name | Default | Meaning
--- | --- | ---
spark.yarn.applicationMaster.waitTries | 10 | ApplicationMaster等待Spark master的次数以及SparkContext初始化尝试的次数
spark.yarn.submit.file.replication | HDFS默认的复制次数（3） | 上传到HDFS的文件的HDFS复制水平。这些文件包括Spark jar、app jar以及任何分布式缓存文件/档案
spark.yarn.preserve.staging.files | false | 设置为true，则在作业结束时保留阶段性文件（Spark jar、app jar以及任何分布式缓存文件）而不是删除它们
spark.yarn.scheduler.heartbeat.interval-ms | 5000 | Spark application master给YARN ResourceManager发送心跳的时间间隔（ms）
spark.yarn.max.executor.failures | numExecutors * 2,最小为3 | 失败应用程序之前最大的执行失败数
spark.yarn.historyServer.address | (none) | Spark历史服务器（如host.com:18080）的地址。这个地址不应该包含一个模式（http://）。默认情况下没有设置值，这是因为该选项是一个可选选项。当Spark应用程序完成从ResourceManager UI到Spark历史服务器UI的连接时，这个地址从YARN ResourceManager得到
spark.yarn.dist.archives | (none) | 提取逗号分隔的档案列表到每个执行器的工作目录
spark.yarn.dist.files | (none) | 放置逗号分隔的文件列表到每个执行器的工作目录
spark.yarn.executor.memoryOverhead | executorMemory * 0.07,最小384 | 分配给每个执行器的堆内存大小（以MB为单位）。它是VM开销、interned字符串或者其它本地开销占用的内存。这往往随着执行器大小而增长。（典型情况下是6%-10%）
spark.yarn.driver.memoryOverhead | driverMemory * 0.07,最小384 | 分配给每个driver的堆内存大小（以MB为单位）。它是VM开销、interned字符串或者其它本地开销占用的内存。这往往随着执行器大小而增长。（典型情况下是6%-10%）
spark.yarn.queue | default | 应用程序被提交到的YARN队列的名称
spark.yarn.jar | (none) | Spark jar文件的位置，覆盖默认的位置。默认情况下，Spark on YARN将会用到本地安装的Spark jar。但是Spark jar也可以HDFS中的一个公共位置。这允许YARN缓存它到节点上，而不用在每次运行应用程序时都需要分配。指向HDFS中的jar包，可以这个参数为"hdfs:///some/path"
spark.yarn.access.namenodes | (none) | 你的Spark应用程序访问的HDFS namenode列表。例如，`spark.yarn.access.namenodes=hdfs://nn1.com:8032,hdfs://nn2.com:8032`，Spark应用程序必须访问namenode列表，Kerberos必须正确配置来访问它们。Spark获得namenode的安全令牌，这样Spark应用程序就能够访问这些远程的HDFS集群。
spark.yarn.containerLauncherMaxThreads | 25 | 为了启动执行者容器，应用程序master用到的最大线程数
spark.yarn.appMasterEnv.[EnvironmentVariableName] | (none) | 添加通过`EnvironmentVariableName`指定的环境变量到Application Master处理YARN上的启动。用户可以指定多个该设置，从而设置多个环境变量。在yarn-cluster模式下，这控制Spark driver的环境。在yarn-client模式下，这仅仅控制执行器启动者的环境。

## 在YARN上启动Spark

确保`HADOOP_CONF_DIR`或`YARN_CONF_DIR`指向的目录包含Hadoop集群的（客户端）配置文件。这些配置用于写数据到dfs和连接到YARN ResourceManager。

有两种部署模式可以用来在YARN上启动Spark应用程序。在yarn-cluster模式下，Spark driver运行在application master进程中，这个进程被集群中的YARN所管理，客户端会在初始化应用程序
之后关闭。在yarn-client模式下，driver运行在客户端进程中，application master仅仅用来向YARN请求资源。

和Spark单独模式以及Mesos模式不同，在这些模式中，master的地址由"master"参数指定，而在YARN模式下，ResourceManager的地址从Hadoop配置得到。因此master参数是简单的`yarn-client`和`yarn-cluster`。

在yarn-cluster模式下启动Spark应用程序。

```shell
./bin/spark-submit --class path.to.your.Class --master yarn-cluster [options] <app jar> [app options]
```

例子：
```shell
$ ./bin/spark-submit --class org.apache.spark.examples.SparkPi \
    --master yarn-cluster \
    --num-executors 3 \
    --driver-memory 4g \
    --executor-memory 2g \
    --executor-cores 1 \
    --queue thequeue \
    lib/spark-examples*.jar \
    10
```

以上启动了一个YARN客户端程序用来启动默认的 Application Master，然后SparkPi会作为Application Master的子线程运行。客户端会定期的轮询Application Master用于状态更新并将
更新显示在控制台上。一旦你的应用程序运行完毕，客户端就会退出。

在yarn-client模式下启动Spark应用程序，运行下面的shell脚本

```shell
$ ./bin/spark-shell --master yarn-client
```

### 添加其它的jar

在yarn-cluster模式下，driver运行在不同的机器上，所以离开了保存在本地客户端的文件，`SparkContext.addJar`将不会工作。为了使`SparkContext.addJar`用到保存在客户端的文件，
在启动命令中加上`--jars`选项。
```shell
$ ./bin/spark-submit --class my.main.Class \
    --master yarn-cluster \
    --jars my-other-jar.jar,my-other-other-jar.jar
    my-main-jar.jar
    app_arg1 app_arg2
```

## 注意事项

- 在Hadoop 2.2之前，YARN不支持容器核的资源请求。因此，当运行早期的版本时，通过命令行参数指定的核的数量无法传递给YARN。在调度决策中，核请求是否兑现取决于用哪个调度器以及
如何配置调度器。
- Spark executors使用的本地目录将会是YARN配置（yarn.nodemanager.local-dirs）的本地目录。如果用户指定了`spark.local.dir`，它将被忽略。
- `--files`和`--archives`选项支持指定带 * # * 号文件名。例如，你能够指定`--files localtest.txt#appSees.txt`，它上传你在本地命名为`localtest.txt`的文件到HDFS，但是将会链接为名称`appSees.txt`。当你的应用程序运行在YARN上时，你应该使用`appSees.txt`去引用该文件。
- 如果你在yarn-cluster模式下运行`SparkContext.addJar`，并且用到了本地文件， `--jars`选项允许`SparkContext.addJar`函数能够工作。如果你正在使用 HDFS, HTTP, HTTPS或FTP，你不需要用到该选项

