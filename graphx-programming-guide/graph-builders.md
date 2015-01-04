# 图构造者

GraphX提供了几种方式从RDD或者磁盘上的顶点和边集合构造图。默认情况下，没有哪个图构造者为图的边重新分区，而是把边保留在默认的分区中（例如HDFS中它们的原始块）。[Graph.groupEdges](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@groupEdges((ED,ED)⇒ED):Graph[VD,ED])
需要重新分区图，因为它假定相同的边将会被分配到同一个分区，所以你必须在调用groupEdges之前调用[Graph.partitionBy](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph@partitionBy(PartitionStrategy):Graph[VD,ED])

```scala
object GraphLoader {
  def edgeListFile(
      sc: SparkContext,
      path: String,
      canonicalOrientation: Boolean = false,
      minEdgePartitions: Int = 1)
    : Graph[Int, Int]
}
```

[GraphLoader.edgeListFile](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.GraphLoader$@edgeListFile(SparkContext,String,Boolean,Int):Graph[Int,Int])
提供了一个方式从磁盘上的边列表中加载一个图。它解析如下形式（源顶点ID，目标顶点ID）的连接表，跳过以`#`开头的注释行。

```scala
# This is a comment
2 1
4 1
1 2
```

它从指定的边创建一个图，自动地创建边提及的所有顶点。所有的顶点和边的属性默认都是1。`canonicalOrientation`参数允许重定向正方向(srcId < dstId)的边。这在[connected components](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.lib.ConnectedComponents$)
算法中需要用到。`minEdgePartitions`参数指定生成的边分区的最少数量。边分区可能比指定的分区更多，例如，一个HDFS文件包含更多的块。

```scala
object Graph {
  def apply[VD, ED](
      vertices: RDD[(VertexId, VD)],
      edges: RDD[Edge[ED]],
      defaultVertexAttr: VD = null)
    : Graph[VD, ED]
  def fromEdges[VD, ED](
      edges: RDD[Edge[ED]],
      defaultValue: VD): Graph[VD, ED]
  def fromEdgeTuples[VD](
      rawEdges: RDD[(VertexId, VertexId)],
      defaultValue: VD,
      uniqueEdges: Option[PartitionStrategy] = None): Graph[VD, Int]
}
```
[Graph.apply](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph$@apply[VD,ED](RDD[(VertexId,VD)],RDD[Edge[ED]],VD)(ClassTag[VD],ClassTag[ED]):Graph[VD,ED])
允许从顶点和边的RDD上创建一个图。重复的顶点可以任意的选择其中一个，在边RDD中而不是在顶点RDD中发现的顶点分配默认的属性。

[Graph.fromEdges](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph$@fromEdges[VD,ED](RDD[Edge[ED]],VD)(ClassTag[VD],ClassTag[ED]):Graph[VD,ED])
允许仅仅从一个边RDD上创建一个图，它自动地创建边提及的顶点，并分配这些顶点默认的值。

[Graph.fromEdgeTuples](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.Graph$@fromEdgeTuples[VD](RDD[(VertexId,VertexId)],VD,Option[PartitionStrategy])(ClassTag[VD]):Graph[VD,Int])
允许仅仅从一个边元组组成的RDD上创建一个图。分配给边的值为1。它自动地创建边提及的顶点，并分配这些顶点默认的值。它还支持删除边。为了删除边，需要传递一个[PartitionStrategy](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.PartitionStrategy)
为值的`Some`作为`uniqueEdges`参数（如uniqueEdges = Some(PartitionStrategy.RandomVertexCut)）。分配相同的边到同一个分区从而使它们可以被删除，一个分区策略是必须的。
