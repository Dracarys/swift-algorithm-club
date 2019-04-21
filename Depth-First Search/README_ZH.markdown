# 深度优先搜索（Depth-First Search）

> 相关教程发表在[这里](https://www.raywenderlich.com/157949/swift-algorithm-club-depth-first-search)

深度优先搜索(DFS)是遍历或搜索[树](../Tree/) 或 [图](../Graph/)等数据结构的一种算法。从源点开始，在回溯之前尽量遍历较远的每一个分支。

深度优先搜索既适用于有向图也适用于无向图。

## 动画示例

下面演示了深度优先搜索是如何搜索一张图的：

![Animated example](Images/AnimatedExample.gif)

首先从 `A` 节点开始。我们先寻找该节点的第一个临近节点，并访问它，即示例中的节点 `B`。之后，在查找 `B` 节点的第一个临近节点，并访问它即节点 `D`。至此，`D` 节点已经没有未被访问过的临近节点，此时回溯到节点 `B`并访问它的另一个临近节点 `E`。依此类推，直至访问完图中所有节点。

依次访问它的第一个临近节点，直至无法继续，然后在回溯到可继续的那个节点（即存在未被访问的临近节点）。到回溯完所有路径并返回起始节点 `A` 时，搜索完毕。

以上图为例，依次访问的节点顺序为：`A`, `B`, `D`, `E`, `H`, `F`, `G`, `C`。
For the example, the nodes were visited in the order `A`, `B`, `D`, `E`, `H`, `F`, `G`, `C`.

深度优先搜索的过程也可以以树的形式呈现：

![Traversal tree](Images/TraversalTree.png)

父节点即是那个通过它“发现”该节点的节点。根节是深度优先算法的起始节点。每一个分支都代表一次回溯。
## 代码

一个用递归实现的深度优先搜索：

```swift
func depthFirstSearch(_ graph: Graph, source: Node) -> [String] {
  var nodesExplored = [source.label]
  source.visited = true

  for edge in source.neighbors {
    if !edge.neighbor.visited {
      nodesExplored += depthFirstSearch(graph, source: edge.neighbor)
    }
  }
  return nodesExplored
}
```

与[广度优先搜索](../Breadth-First%20Search/)优先访问临近节点不同，深度优先搜索是尽可能地深入访问（树或图）。

将下列代码复制到 playground 中尝试一下：
```swift
let graph = Graph()

let nodeA = graph.addNode("a")
let nodeB = graph.addNode("b")
let nodeC = graph.addNode("c")
let nodeD = graph.addNode("d")
let nodeE = graph.addNode("e")
let nodeF = graph.addNode("f")
let nodeG = graph.addNode("g")
let nodeH = graph.addNode("h")

graph.addEdge(nodeA, neighbor: nodeB)
graph.addEdge(nodeA, neighbor: nodeC)
graph.addEdge(nodeB, neighbor: nodeD)
graph.addEdge(nodeB, neighbor: nodeE)
graph.addEdge(nodeC, neighbor: nodeF)
graph.addEdge(nodeC, neighbor: nodeG)
graph.addEdge(nodeE, neighbor: nodeH)
graph.addEdge(nodeE, neighbor: nodeF)
graph.addEdge(nodeF, neighbor: nodeG)

let nodesExplored = depthFirstSearch(graph, source: nodeA)
print(nodesExplored)
```

输出结果应为：`["a", "b", "d", "e", "h", "f", "g", "c"]`。

## DFS适用哪些场景呢？

深度优先搜索可以用来解决很多问题，例如：

* 查找一个稀疏图（Sparse graph）的连接构成
* 对图中节点进行[拓扑排序（Topological sorting）](../Topological%20Sort/)
* 查询图中的[Bridges](https://en.wikipedia.org/wiki/Bridge_(graph_theory)#Bridge-finding_algorithm)
* 其它!

Depth-first search can be used to solve many problems, for example:

* Finding connected components of a sparse graph
* [Topological sorting](../Topological%20Sort/) of nodes in a graph
* Finding bridges of a graph (see: [Bridges](https://en.wikipedia.org/wiki/Bridge_(graph_theory)#Bridge-finding_algorithm))
* And lots of others!

*Written for Swift Algorithm Club by Paulo Tanaka and Matthijs Hollemans*

*由 William Han 翻译*
