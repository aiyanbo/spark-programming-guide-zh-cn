# 部署应用程序

## Requirements

运行一个Spark Streaming应用程序，有下面一些步骤

- 有管理器的集群-这是任何Spark应用程序都需要的需求，详见[部署指南](../../deploying/README.md)
- 将应用程序打为jar包-你必须编译你的应用程序为jar包。如果你用[spark-submit](../../deploying/submitting-applications.md)启动应用程序，你不需要将Spark和Spark Streaming打包进这个jar包。
如果你的应用程序用到了高级源（如kafka，flume），你需要将它们关联的外部artifact以及它们的依赖打包进需要部署的应用程序jar包中。例如，一个应用程序用到了`TwitterUtils`，那么就需要将`spark-streaming-twitter_2.10`
以及它的所有依赖打包到应用程序jar中。
- 为executors配置足够的内存-因为接收的数据必须存储在内存中，executors必须配置足够的内存用来保存接收的数据。注意，如果你正在做10分钟的窗口操作，系统的内存要至少能保存10分钟的数据。所以，应用程序的内存需求依赖于使用
它的操作。
- 配置checkpointing-如果stream应用程序需要checkpointing，然后一个与Hadoop API兼容的容错存储目录必须配置为检查点的目录，流应用程序将checkpoint信息写入该目录用于错误恢复。
- 配置应用程序driver的自动重启-为了自动从driver故障中恢复，运行流应用程序的部署设施必须能监控driver进程，如果失败了能够重启它。不同的集群管理器，有不同的工具得到该功能
    - Spark Standalone：一个Spark应用程序driver可以提交到Spark独立集群运行，也就是说driver运行在一个worker节点上。进一步来看，独立的集群管理器能够被指示用来监控driver，并且在driver失败（或者是由于非零的退出代码如exit(1)，
    或者由于运行driver的节点的故障）的情况下重启driver。
    - YARN：YARN为自动重启应用程序提供了类似的机制。
    - Mesos： Mesos可以用[Marathon](https://github.com/mesosphere/marathon)提供该功能
- 配置write ahead logs-在Spark 1.2中，为了获得极强的容错保证，我们引入了一个新的实验性的特性-预写日志（write ahead logs）。如果该特性开启，从receiver获取的所有数据会将预写日志写入配置的checkpoint目录。
这可以防止driver故障丢失数据，从而保证零数据丢失。这个功能可以通过设置配置参数`spark.streaming.receiver.writeAheadLogs.enable`为true来开启。然而，这些较强的语义可能以receiver的接收吞吐量为代价。这可以通过
并行运行多个receiver增加吞吐量来解决。另外，当预写日志开启时，Spark中的复制数据的功能推荐不用，因为该日志已经存储在了一个副本在存储系统中。可以通过设置输入DStream的存储级别为`StorageLevel.MEMORY_AND_DISK_SER`获得该功能。


## 升级应用程序代码

如果运行的Spark Streaming应用程序需要升级，有两种可能的方法

- 启动升级的应用程序，使其与未升级的应用程序并行运行。一旦新的程序（与就程序接收相同的数据）已经准备就绪，旧的应用程序就可以关闭。这种方法支持将数据发送到两个不同的目的地（新程序一个，旧程序一个）
- 首先，平滑的关闭（`StreamingContext.stop(...)`或`JavaStreamingContext.stop(...)`）现有的应用程序。在关闭之前，要保证已经接收的数据完全处理完。然后，就可以启动升级的应用程序，升级
的应用程序会接着旧应用程序的点开始处理。这种方法仅支持具有源端缓存功能的输入源（如flume，kafka），这是因为当旧的应用程序已经关闭，升级的应用程序还没有启动的时候，数据需要被缓存。

