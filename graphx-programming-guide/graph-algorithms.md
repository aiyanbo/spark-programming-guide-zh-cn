# 图算法

GraphX包括一组图算法来简化分析任务。这些算法包含在`org.apache.spark.graphx.lib`包中，可以被直接访问。

## PageRank算法

PageRank度量一个图中每个顶点的重要程度，假定从u到v的一条边代表v的重要性标签。例如，一个Twitter用户被许多其它人粉，该用户排名很高。GraphX带有静态和动态PageRank的实现方法
，这些方法在[PageRank object](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.lib.PageRank$)中。静态的PageRank运行固定次数
的迭代，而动态的PageRank一直运行，直到收敛。[GraphOps]()允许直接调用这些算法作为图上的方法。

GraphX包含一个我们可以运行PageRank的社交网络数据集的例子。用户集在`graphx/data/users.txt`中，用户之间的关系在`graphx/data/followers.txt`中。我们通过下面的方法计算
每个用户的PageRank。

```scala
// Load the edges as a graph
val graph = GraphLoader.edgeListFile(sc, "graphx/data/followers.txt")
// Run PageRank
val ranks = graph.pageRank(0.0001).vertices
// Join the ranks with the usernames
val users = sc.textFile("graphx/data/users.txt").map { line =>
  val fields = line.split(",")
  (fields(0).toLong, fields(1))
}
val ranksByUsername = users.join(ranks).map {
  case (id, (username, rank)) => (username, rank)
}
// Print the result
println(ranksByUsername.collect().mkString("\n"))
```

## 连通体算法

连通体算法用id标注图中每个连通体，将连通体中序号最小的顶点的id作为连通体的id。例如，在社交网络中，连通体可以近似为集群。GraphX在[ConnectedComponents object](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.lib.ConnectedComponents$)
中包含了一个算法的实现，我们通过下面的方法计算社交网络数据集中的连通体。

```scala
/ Load the graph as in the PageRank example
val graph = GraphLoader.edgeListFile(sc, "graphx/data/followers.txt")
// Find the connected components
val cc = graph.connectedComponents().vertices
// Join the connected components with the usernames
val users = sc.textFile("graphx/data/users.txt").map { line =>
  val fields = line.split(",")
  (fields(0).toLong, fields(1))
}
val ccByUsername = users.join(cc).map {
  case (id, (username, cc)) => (username, cc)
}
// Print the result
println(ccByUsername.collect().mkString("\n"))
```

## 三角形计数算法

一个顶点有两个相邻的顶点以及相邻顶点之间的边时，这个顶点是一个三角形的一部分。GraphX在[TriangleCount object](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.graphx.lib.TriangleCount$)
中实现了一个三角形计数算法，它计算通过每个顶点的三角形的数量。需要注意的是，在计算社交网络数据集的三角形计数时，`TriangleCount`需要边的方向是规范的方向(srcId < dstId),
并且图通过`Graph.partitionBy`分片过。

```scala
// Load the edges in canonical order and partition the graph for triangle count
val graph = GraphLoader.edgeListFile(sc, "graphx/data/followers.txt", true).partitionBy(PartitionStrategy.RandomVertexCut)
// Find the triangle count for each vertex
val triCounts = graph.triangleCount().vertices
// Join the triangle counts with the usernames
val users = sc.textFile("graphx/data/users.txt").map { line =>
  val fields = line.split(",")
  (fields(0).toLong, fields(1))
}
val triCountByUsername = users.join(triCounts).map { case (id, (username, tc)) =>
  (username, tc)
}
// Print the result
println(triCountByUsername.collect().mkString("\n"))
```