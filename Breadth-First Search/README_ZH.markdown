# 广度优先搜索（Breadth-First Search）

> 相关教程已发在[这里](https://www.raywenderlich.com/155801/swift-algorithm-club-swift-breadth-first-search)

广度优先搜索（BFS）是对[树](../Tree/) 或 [图](../Graph/)等数据结构进行遍历或搜索时应用的一种算法。它从源节点开始，先遍历临近节点，之后在遍历下一级的邻居节点。
Breadth-first search (BFS) is an algorithm for traversing or searching [tree](../Tree/) or [graph](../Graph/) data structures. It starts at a source node and explores the immediate neighbor nodes first, before moving to the next level neighbors.

广度优先搜索即可用于有向图也可用于无向图。
Breadth-first search can be used on both directed and undirected graphs.

## 动画示例（Animated example）

下面的动画展示了广度优先算法是如何搜索一个图的：
Here's how breadth-first search works on a graph:

![Animated example of a breadth-first search](Images/AnimatedExample.gif)

访问过的节点，将其标黑。同时将它的临近节点放入[队列](../Queue/)中。那些已入队但尚未被访问的节点，在动画中用灰色标记。
When we visit a node, we color it black. We also put its neighbor nodes into a [queue](../Queue/). In the animation the nodes that are enqueued but not visited yet are shown in gray.

动画中，首先从`A`节点开始，将其添加到队列中，此时`A`节点变成灰色。
Let's follow the animated example. We start with the source node `A` and add it to a queue. In the animation this is shown as node `A` becoming gray.

```swift
queue.enqueue(A)
```

此时队列变成 `[ A ]`。具体到思路是，随着节点的入队，开始访问位于队列前端的节点，同时将其那些尚未被访问的临近节点入队。
The queue is now `[ A ]`. The idea is that, as long as there are nodes in the queue, we visit the node that's at the front of the queue, and enqueue its immediate neighbor nodes if they have not been visited yet.

在开始遍历图之前，先将起始节点 `A` 出队，并将其标黑。紧接着入队它的两个临近节点 `B` 和 `C`，与此同时将他们标灰。
To start traversing the graph, we pull the first node off the queue, `A`, and color it black. Then we enqueue its two neighbor nodes `B` and `C`. This colors them gray.

```swift
queue.dequeue()   // A
queue.enqueue(B)
queue.enqueue(C)
```

此时队列变为 `[ B, C ]`。接着将 `B` 出队，在将 `B` 的临近节点 `D` 和 `E` 入队。
The queue is now `[ B, C ]`. We dequeue `B`, and enqueue `B`'s neighbor nodes `D` and `E`.

```swift
queue.dequeue()   // B
queue.enqueue(D)
queue.enqueue(E)
```

这是队列变为 `[ C, D, E ]`。之后将 `C`入队，并将 `C` 的临近节点 `F` 和 `G` 入队。
The queue is now `[ C, D, E ]`. Dequeue `C`, and enqueue `C`'s neighbor nodes `F` and `G`.

```swift
queue.dequeue()   // C
queue.enqueue(F)
queue.enqueue(G)
```

此时队列变成了 `[ D, E, F, G ]`。将 `D`，它已经没有（尚未被访问的）临近节点了。
The queue is now `[ D, E, F, G ]`. Dequeue `D`, which has no neighbor nodes.

```swift
queue.dequeue()   // D
```

队列变成 `[ E, F, G ]`。
The queue is now `[ E, F, G ]`. Dequeue `E` and enqueue its single neighbor node `H`. Note that `B` is also a neighbor for `E` but we've already visited `B`, so we're not adding it to the queue again.

```swift
queue.dequeue()   // E
queue.enqueue(H)
```

The queue is now `[ F, G, H ]`. Dequeue `F`, which has no unvisited neighbor nodes.

```swift
queue.dequeue()   // F
```

The queue is now `[ G, H ]`. Dequeue `G`, which has no unvisited neighbor nodes.

```swift
queue.dequeue()   // G
```

The queue is now `[ H ]`. Dequeue `H`, which has no unvisited neighbor nodes.

```swift
queue.dequeue()   // H
```

The queue is now empty, meaning that all nodes have been explored. The order in which the nodes were explored is `A`, `B`, `C`, `D`, `E`, `F`, `G`, `H`.

We can show this as a tree:

![The BFS tree](Images/TraversalTree.png)

The parent of a node is the one that "discovered" that node. The root of the tree is the node you started the breadth-first search from.

For an unweighted graph, this tree defines a shortest path from the starting node to every other node in the tree. So breadth-first search is one way to find the shortest path between two nodes in a graph.

## The code

Simple implementation of breadth-first search using a queue:

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

While there are nodes in the queue, we visit the first one and then enqueue its immediate neighbors if they haven't been visited yet.

Put this code in a playground and test it like so:

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

This will output: `["a", "b", "c", "d", "e", "f", "g", "h"]`
   
## 哪里适用广度优先算法呢？（What is BFS good for?）

广度优先算法适用范围很广，下面简单列举两个：

* 求[最短路径](../Shortest%20Path%20(Unweighted)/)

Breadth-first search can be used to solve many problems. A small selection:

* Computing the [shortest path](../Shortest%20Path%20(Unweighted)/) between a source node and each of the other nodes (only for unweighted graphs).
* Calculating the [minimum spanning tree](../Minimum%20Spanning%20Tree%20(Unweighted)/) on an unweighted graph.

*Written by [Chris Pilcher](https://github.com/chris-pilcher) and Matthijs Hollemans*
