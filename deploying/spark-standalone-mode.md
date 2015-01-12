# Spark独立部署模式

## 安装Spark独立模式集群

安装Spark独立模式，你只需要将Spark的编译版本简单的放到集群的每个节点。你可以获得每个稳定版本的预编译版本，也可以自己编译。

## 手动启动集群

你能够通过下面的方式启动独立的master服务器。

```shell
./sbin/start-master.sh
```

一旦启动，master将会为自己打印出`spark://HOST:PORT` URL，你能够用它连接到workers或者作为"master"参数传递给`SparkContext`。你也可以在master web UI上发现这个URL，
master web UI默认的地址是`http://localhost:8080`。

相同的，你也可以启动一个或者多个workers或者将它们连接到master。

```shell
./bin/spark-class org.apache.spark.deploy.worker.Worker spark://IP:PORT
```

一旦你启动了一个worker，查看master web UI。你可以看到新的节点列表以及节点的CPU数以及内存。

下面的配置参数可以传递给master和worker。

Argument | Meaning
--- | ---
-h HOST, --host HOST | 监听的主机名
-i HOST, --ip HOST | 同上，已经被淘汰
-p PORT, --port PORT | 监听的服务的端口（master默认是7077，worker随机）
--webui-port PORT | web UI的端口(master默认是8080，worker默认是8081)
-c CORES, --cores CORES | Spark应用程序可以使用的CPU核数（默认是所有可用）；这个选项仅在worker上可用
-m MEM, --memory MEM | Spark应用程序可以使用的内存数（默认情况是你的机器内存数减去1g）；这个选项仅在worker上可用
-d DIR, --work-dir DIR | 用于暂存空间和工作输出日志的目录（默认是SPARK_HOME/work）；这个选项仅在worker上可用
--properties-file FILE | 自定义的Spark配置文件的加载目录（默认是conf/spark-defaults.conf）

## 集群启动脚本

为了用启动脚本启动Spark独立集群，你应该在你的Spark目录下建立一个名为`conf/slaves`的文件，这个文件必须包含所有你要启动的Spark worker所在机器的主机名，一行一个。如果
`conf/slaves`不存在，启动脚本默认为单个机器（localhost），这台机器对于测试是有用的。注意，master机器通过ssh访问所有的worker。在默认情况下，SSH是并行运行，需要设置无密码（采用私有密钥）的访问。
如果你没有设置为无密码访问，你可以设置环境变量`SPARK_SSH_FOREGROUND`，为每个worker提供密码。

一旦你设置了这个文件，你就可以通过下面的shell脚本启动或者停止你的集群。

- sbin/start-master.sh：在机器上启动一个master实例
- sbin/start-slaves.sh：在每台机器上启动一个slave实例
- sbin/start-all.sh：同时启动一个master实例和所有slave实例
- sbin/stop-master.sh：停止master实例
- sbin/stop-slaves.sh：停止所有slave实例
- sbin/stop-all.sh：停止master实例和所有slave实例

注意，这些脚本必须在你的Spark master运行的机器上执行，而不是在你的本地机器上面。

你可以在`conf/spark-env.sh`中设置环境变量进一步配置集群。利用`conf/spark-env.sh.template`创建这个文件，然后将它复制到所有的worker机器上使设置有效。下面的设置可以起作用：

Environment Variable | Meaning
--- | ---
SPARK_MASTER_IP | 绑定master到一个指定的ip地址
SPARK_MASTER_PORT | 在不同的端口上启动master（默认是7077）
SPARK_MASTER_WEBUI_PORT | master web UI的端口（默认是8080）
SPARK_MASTER_OPTS | 应用到master的配置属性，格式是 "-Dx=y"（默认是none），查看下面的表格的选项以组成一个可能的列表
SPARK_LOCAL_DIRS | Spark中暂存空间的目录。包括map的输出文件和存储在磁盘上的RDDs(including map output files and RDDs that get stored on disk)。这必须在一个快速的、你的系统的本地磁盘上。它可以是一个逗号分隔的列表，代表不同磁盘的多个目录
SPARK_WORKER_CORES | Spark应用程序可以用到的核心数（默认是所有可用）
SPARK_WORKER_MEMORY | Spark应用程序用到的内存总数（默认是内存总数减去1G）。注意，每个应用程序个体的内存通过`spark.executor.memory`设置
SPARK_WORKER_PORT | 在指定的端口上启动Spark worker(默认是随机)
SPARK_WORKER_WEBUI_PORT | worker UI的端口（默认是8081）
SPARK_WORKER_INSTANCES | 每台机器运行的worker实例数，默认是1。如果你有一台非常大的机器并且希望运行多个worker，你可以设置这个数大于1。如果你设置了这个环境变量，确保你也设置了`SPARK_WORKER_CORES`环境变量用于限制每个worker的核数或者每个worker尝试使用所有的核。
SPARK_WORKER_DIR | Spark worker运行目录，该目录包括日志和暂存空间（默认是SPARK_HOME/work）
SPARK_WORKER_OPTS | 应用到worker的配置属性，格式是 "-Dx=y"（默认是none），查看下面表格的选项以组成一个可能的列表
SPARK_DAEMON_MEMORY | 分配给Spark master和worker守护进程的内存（默认是512m）
SPARK_DAEMON_JAVA_OPTS | Spark master和worker守护进程的JVM选项，格式是"-Dx=y"（默认为none）
SPARK_PUBLIC_DNS | Spark master和worker公共的DNS名（默认是none）

注意，启动脚本还不支持windows。为了在windows上启动Spark集群，需要手动启动master和workers。

`SPARK_MASTER_OPTS`支持一下的系统属性：

Property Name | Default | Meaning
--- | --- | ---
spark.deploy.retainedApplications | 200 | 展示完成的应用程序的最大数目。老的应用程序会被删除以满足该限制
spark.deploy.retainedDrivers | 200 | 展示完成的drivers的最大数目。老的应用程序会被删除以满足该限制
spark.deploy.spreadOut | true | 这个选项控制独立的集群管理器是应该跨节点传递应用程序还是应努力将程序整合到尽可能少的节点上。在HDFS中，传递程序是数据本地化更好的选择，但是，对于计算密集型的负载，整合会更有效率。
spark.deploy.defaultCores | (infinite) | 在Spark独立模式下，给应用程序的默认核数（如果没有设置`spark.cores.max`）。如果没有设置，应用程序总数获得所有可用的核，除非设置了`spark.cores.max`。在共享集群上设置较低的核数，可用防止用户默认抓住整个集群。
spark.worker.timeout | 60 | 独立部署的master认为worker失败（没有收到心跳信息）的间隔时间。

`SPARK_WORKER_OPTS`支持的系统属性：

Property Name | Default | Meaning
--- | --- | ---
spark.worker.cleanup.enabled | false | 周期性的清空worker/应用程序目录。注意，这仅仅影响独立部署模式。不管应用程序是否还在执行，用于程序目录都会被清空
spark.worker.cleanup.interval | 1800 (30分) | 在本地机器上，worker清空老的应用程序工作目录的时间间隔
spark.worker.cleanup.appDataTtl | 7 * 24 * 3600 (7天) | 每个worker中应用程序工作目录的保留时间。这个时间依赖于你可用磁盘空间的大小。应用程序日志和jar包上传到每个应用程序的工作目录。随着时间的推移，工作目录会很快的填满磁盘空间，特别是如果你运行的作业很频繁。

## 连接一个应用程序到集群中

为了在Spark集群中运行一个应用程序，简单地传递`spark://IP:PORT` URL到[SparkContext](http://spark.apache.org/docs/latest/programming-guide.html#initializing-spark)

为了在集群上运行一个交互式的Spark shell，运行一下命令：

```shell
./bin/spark-shell --master spark://IP:PORT
```
你也可以传递一个选项`--total-executor-cores <numCores>`去控制spark-shell的核数。

## 启动Spark应用程序

[spark-submit脚本](submitting-applications.md)支持最直接的提交一个Spark应用程序到集群。对于独立部署的集群，Spark目前支持两种部署模式。在`client`模式中，driver启动进程与
客户端提交应用程序所在的进程是同一个进程。然而，在`cluster`模式中，driver在集群的某个worker进程中启动，只有客户端进程完成了提交任务，它不会等到应用程序完成就会退出。

如果你的应用程序通过Spark submit启动，你的应用程序jar包将会自动分发到所有的worker节点。对于你的应用程序依赖的其它jar包，你应该用`--jars`符号指定（如` --jars jar1,jar2`）。

另外，`cluster`模式支持自动的重启你的应用程序（如果程序一非零的退出码退出）。为了用这个特征，当启动应用程序时，你可以传递`--supervise`符号到`spark-submit`。如果你想杀死反复失败的应用，
你可以通过如下的方式：

```shell
./bin/spark-class org.apache.spark.deploy.Client kill <master url> <driver ID>
```

你可以在独立部署的Master web UI（http://<master url>:8080）中找到driver ID。

## 资源调度

独立部署的集群模式仅仅支持简单的FIFO调度器。然而，为了允许多个并行的用户，你能够控制每个应用程序能用的最大资源数。在默认情况下，它将获得集群的所有核，这只有在某一时刻只
允许一个应用程序才有意义。你可以通过`spark.cores.max`在[SparkConf](http://spark.apache.org/docs/latest/configuration.html#spark-properties)中设置核数。

```scala
val conf = new SparkConf()
             .setMaster(...)
             .setAppName(...)
             .set("spark.cores.max", "10")
val sc = new SparkContext(conf)
```
另外，你可以在集群的master进程中配置`spark.deploy.defaultCores`来改变默认的值。在`conf/spark-env.sh`添加下面的行：

```properties
export SPARK_MASTER_OPTS="-Dspark.deploy.defaultCores=<value>"
```

这在用户没有配置最大核数的共享集群中是有用的。

## 高可用

默认情况下，独立的调度集群对worker失败是有弹性的（在Spark本身的范围内是有弹性的，对丢失的工作通过转移它到另外的worker来解决）。然而，调度器通过master去执行调度决定，
这会造成单点故障：如果master死了，新的应用程序就无法创建。为了避免这个，我们有两个高可用的模式。

### 用ZooKeeper的备用master

利用ZooKeeper去支持领导选举以及一些状态存储，你能够在你的集群中启动多个master，这些master连接到同一个ZooKeeper实例上。一个被选为“领导”，其它的保持备用模式。如果当前
的领导死了，另一个master将会被选中，恢复老master的状态，然后恢复调度。整个的恢复过程大概需要1到2分钟。注意，这个恢复时间仅仅会影响调度新的应用程序-运行在失败master中的
应用程序不受影响。

#### 配置

为了开启这个恢复模式，你可以用下面的属性在`spark-env`中设置`SPARK_DAEMON_JAVA_OPTS`。

System property | Meaning
--- | ---
spark.deploy.recoveryMode | 设置ZOOKEEPER去启动备用master模式（默认为none）
spark.deploy.zookeeper.url | zookeeper集群url(如192.168.1.100:2181,192.168.1.101:2181)
spark.deploy.zookeeper.dir | zookeeper保存恢复状态的目录（默认是/spark）

可能的陷阱：如果你在集群中有多个masters，但是没有用zookeeper正确的配置这些masters，这些masters不会发现彼此，会认为它们都是leaders。这将会造成一个不健康的集群状态（因为所有的master都会独立的调度）。

#### 细节

zookeeper集群启动之后，开启高可用是简单的。在相同的zookeeper配置（zookeeper URL和目录）下，在不同的节点上简单地启动多个master进程。master可以随时添加和删除。

为了调度新的应用程序或者添加worker到集群，它需要知道当前leader的IP地址。这可以通过简单的传递一个master列表来完成。例如，你可能启动你的SparkContext指向`spark://host1:port1,host2:port2`。
这将造成你的SparkContext同时注册这两个master-如果`host1`死了，这个配置文件将一直是正确的，因为我们将找到新的leader-`host2`。

"registering with a Master"和正常操作之间有重要的区别。当启动时，一个应用程序或者worker需要能够发现和注册当前的leader master。一旦它成功注册，它就在系统中了。如果
错误发生，新的leader将会接触所有之前注册的应用程序和worker，通知他们领导关系的变化，所以它们甚至不需要事先知道新启动的leader的存在。

由于这个属性的存在，新的master可以在任何时候创建。你唯一需要担心的问题是新的应用程序和workers能够发现它并将它注册进来以防它成为leader master。

### 用本地文件系统做单节点恢复

zookeeper是生产环境下最好的选择，但是如果你想在master死掉后重启它，`FILESYSTEM`模式可以解决。当应用程序和worker注册，它们拥有足够的状态写入提供的目录，以至于在重启master
进程时它们能够恢复。

#### 配置

为了开启这个恢复模式，你可以用下面的属性在`spark-env`中设置`SPARK_DAEMON_JAVA_OPTS`。

System property | Meaning
--- | ---
spark.deploy.recoveryMode | 设置为FILESYSTEM开启单节点恢复模式（默认为none）
spark.deploy.recoveryDirectory | 用来恢复状态的目录

#### 细节

- 这个解决方案可以和监控器/管理器（如[monit](http://mmonit.com/monit/)）相配合，或者仅仅通过重启开启手动恢复。
- 虽然文件系统的恢复似乎比没有做任何恢复要好，但对于特定的开发或实验目的，这种模式可能是次优的。特别是，通过`stop-master.sh`杀掉master不会清除它的恢复状态，所以，不管你何时启动一个新的master，它都将进入恢复模式。这可能使启动时间增加到1分钟。
- 虽然它不是官方支持的方式，你也可以创建一个NFS目录作为恢复目录。如果原始的master节点完全死掉，你可以在不同的节点启动master，它可以正确的恢复之前注册的所有应用程序和workers。未来的应用程序会发现这个新的master。