# 离散流（DStreams）

离散流或者DStreams是Spark Streaming提供的基本的抽象，它代表一个连续的数据流。它要么是从源中获取的输入流，要么是输入流通过转换算子生成的处理后的数据流。在内部，DStreams由一系列连续的
RDD组成。DStreams中的每个RDD都包含确定时间间隔内的数据，如下图所示：

![DStreams](../../img/streaming-dstream.png)

任何对DStreams的操作都转换成了对DStreams隐含的RDD的操作。在前面的[例子](../a-quick-example.md)中，`flatMap`操作应用于`lines`这个DStreams的每个RDD，生成`words`这个DStreams的
RDD。过程如下图所示：

![DStreams](../../img/streaming-dstream-ops.png)

通过Spark引擎计算这些隐含RDD的转换算子。DStreams操作隐藏了大部分的细节，并且为了更便捷，为开发者提供了更高层的API。下面几节将具体讨论这些操作的细节。