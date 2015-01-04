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


