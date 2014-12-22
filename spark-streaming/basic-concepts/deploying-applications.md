# 部署应用程序

## Requirements

运行一个Spark Streaming应用程序，有下面一些步骤

- 有管理器的集群-这是任何Spark应用程序都需要的需求，详见[部署指南](../../deploying/README.md)
- 将应用程序打为jar包-你必须编译你的应用程序为jar包。如果你用[spark-submit](../../deploying/submitting-applications.md)启动应用程序，你不需要将Spark和Spark Streaming打包进这个jar包。
如果你的应用程序用到了高级源（如kafka，flume），你需要将它们关联的外部artifact以及它们的依赖打包进需要部署的应用程序jar包中。例如，一个应用程序用到了`TwitterUtils`，那么就需要将`spark-streaming-twitter_2.10`
以及它的所有依赖打包到应用程序jar中。
- 


## 升级应用程序代码

如果运行的Spark Streaming应用程序需要升级，有两种可能的方法

- 启动升级的应用程序，使其与未升级的应用程序并行运行。一旦新的程序（与就程序接收相同的数据）已经准备就绪，旧的应用程序就可以关闭。这种方法支持将数据发送到两个不同的目的地（新程序一个，旧程序一个）
- 首先，平滑的关闭（`StreamingContext.stop(...)`或`JavaStreamingContext.stop(...)`）现有的应用程序。在关闭之前，要保证已经接收的数据完全处理完。然后，就可以启动升级的应用程序，升级
的应用程序会接着旧应用程序的点开始处理。这种方法仅支持具有源端缓存功能的输入源（如flume，kafka），这是因为当旧的应用程序已经关闭，升级的应用程序还没有启动的时候，数据需要被缓存。

