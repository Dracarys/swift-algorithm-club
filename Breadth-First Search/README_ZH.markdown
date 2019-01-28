# 广度优先搜索（Breadth-First Search）

> 相关教程已发在[这里](https://www.raywenderlich.com/155801/swift-algorithm-club-swift-breadth-first-search)

广度优先搜索（BFS）是对[树](../Tree/) 或 [图](../Graph/)等数据结构进行遍历或搜索时应用的一种算法。它从起始节点开始，先遍历最近的节点，之后在遍历下一级的临近节点。

广度优先搜索即可用于有向图也可用于无向图。

## 动画示例

下面的动画展示了广度优先算法是如何搜索一个图的：

![Animated example of a breadth-first search](Images/AnimatedExample.gif)

当访问一个节点时，将其标黑。同时将它的临近节点放入[队列](../Queue/)中。那些已入队但尚未被访问的节点，在动画中以灰色标记。

让我们跟着动画（来熟悉一下过程），首先从起始节点 `A` 开始，将其添加到队列中，此时 `A` 节点变成灰色。

```swift
queue.enqueue(A)
```

队列变成 `[ A ]`。（之后的）思路是，随着节点的入队，在访问位于队列首的节点的同时，将其尚未被访问的临近节点入队，（以此类推，如此往复）。

在开始遍历图之前，先将起始节点 `A` 出队，并将其标黑。紧接着入队它的两个临近节点 `B` 和 `C`，与此同时将他们标灰。

```swift
queue.dequeue()   // A
queue.enqueue(B)
queue.enqueue(C)
```

此时队列变为 `[ B, C ]`。接着将 `B` 出队，在将 `B` 的临近节点 `D` 和 `E` 入队。

```swift
queue.dequeue()   // B
queue.enqueue(D)
queue.enqueue(E)
```

这时队列变为 `[ C, D, E ]`。之后将 `C` 出队，并将 `C` 的临近节点 `F` 和 `G` 入队。

```swift
queue.dequeue()   // C
queue.enqueue(F)
queue.enqueue(G)
```

队列变成了 `[ D, E, F, G ]`。在将 `D` 出队，它没有（为被访问）临近节点（，因此无需入队操作）。

```swift
queue.dequeue()   // D
```

此时队列变成 `[ E, F, G ]`。将 `E` 出队，并将它唯一的临近节点 `H` 入队。注意节点 `B` 虽然也是 `E` 的临近节点，但是由于已经被访问过，所以不再重复入队。

```swift
queue.dequeue()   // E
queue.enqueue(H)
```

这时队列变成 `[ F, G, H ]`。将 `F` 出队，它已经没有未被访问的临近节点了（，因此无需入队操作）。

```swift
queue.dequeue()   // F
```

之后队列变成 `[ G, H ]`。将 `G` 出队，它也没有未被访问过的临近节点了（，因此无需入队操作）。

```swift
queue.dequeue()   // G
```

队列变成了 `[ H ]`。将 `H` 出队，同样它也没有未被访问的临近节点了（，因此无需入队操作）。

```swift
queue.dequeue()   // H
```
至此，队列为空，这意味着所有的节点都已经被遍历过。各节点遍历的顺序依次为 `A`, `B`, `C`, `D`, `E`, `F`, `G`, `H`。

以树来表示形式如下：

![The BFS tree](Images/TraversalTree.png)

正是通过父节点“发现”其（子）节点。而根节点则是广度优先查询的起始节点。

对于未加权的图而言，该树定义了从起始到树上各个子节点间最短的路径。因此广度优先查询正是寻找图中两个节点间最短路径的算法之一。

## 代码

下面是一个通过队列实现的广度优先查询示例：

```swift
func breadthFirstSearch(_ graph: Graph, source: Node) -> [String] {
  var queue = Queue<Node>()
  queue.enqueue(source)

  var nodesExplored = [source.label]
  source.visited = true

  while let node = queue.dequeue() {
    for edge in node.neighbors {
      let neighborNode = edge.neighbor
      if !neighborNode.visited {
        queue.enqueue(neighborNode)
        neighborNode.visited = true
        nodesExplored.append(neighborNode.label)
      }
    }
  }

  return nodesExplored
}
```

当队列中存在节点时，我们先访问队首节点，紧接着在将其尚未被访问的临近节点入队。

将下面的代码拷贝到 playground 中尝试一下：

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

let nodesExplored = breadthFirstSearch(graph, source: nodeA)
print(nodesExplored)
```

会输出结果为：`["a", "b", "c", "d", "e", "f", "g", "h"]`
   
## 哪里适用广度优先算法呢？

广度优先算法适用范围很广，下面举两个简单的应用：

* 求[最短路径](../Shortest%20Path%20(Unweighted)/)
* Calculating the [minimum spanning tree](../Minimum%20Spanning%20Tree%20(Unweighted)/) on an unweighted graph.

## 最后（该节待删除，以保持与英文的一致性）

该系列文章翻译自 [Raywenderlich](https://www.raywenderlich.com) 的开源项目：[swift-algorithm-club](https://github.com/raywenderlich/swift-algorithm-club)，意在帮助有一定基础的同学进行回顾，如果你才接触，那么建议移步[详细教程](https://www.raywenderlich.com/155801/swift-algorithm-club-swift-breadth-first-search)

*Written by [Chris Pilcher](https://github.com/chris-pilcher) and Matthijs Hollemans*

*由 William Han 翻译*
