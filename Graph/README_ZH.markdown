# 图（Graph）

> 相关教程发表在 [这里](https://www.raywenderlich.com/152046/swift-algorithm-club-graphs-adjacency-list)

下面是一个图：

![A graph](Images/Graph.png)

在计算机科学中，图是指由一组*边*连接起来的一组*点*。点以圆形呈现，边以圆之间的线段表示。边将一个点与另一个点连接起来。
In computer science, a graph is defined as a set of *vertices* paired with a set of *edges*. The vertices are represented by circles, and the edges are the lines between them. Edges connect a vertex to other vertices.

> **注意：** 点也被称为“节点”，边也可称为“链接”
> **Note:** Vertices are sometimes called "nodes", and edges are called "links".

图可以被喻为一个社交网络。每个人都是一个点，相互认识的人以边相连。下面以历史的大概人物关系举例：
A graph can represent a social network. Each person is a vertex, and people who know each other are connected by edges. Here is a somewhat historically inaccurate example:

![Social network](Images/SocialNetwork.png)

图的大小和形状可以有很多中。边可以用一个正或负的数值表示 *量*，例如这个航线的例子。城市作为点，航线作为边。那么航线上的量即可以表示飞行时间，也可以表示航线的机票价格。
Graphs have various shapes and sizes. The edges can have a *weight*, where a positive or negative numeric value is assigned to each edge. Consider an example of a graph representing airplane flights. Cities can be vertices, and flights can be edges. Then, an edge weight could describe flight time or the price of a ticket.

![Airplane flights](Images/Flights.png)

如上面所示，从旧金山经纽约转墨西哥是最便宜的线路。

边还可以是*有向的*。之前例子中的边是无向的。举例来说，如果 Ada 认识 Charles，那么 Charles 也认识 Ada。反之，有向边仅表示单一关系。即只能从点 X 到点 Y，而*不能*从 Y 到 X。

继续以之前的航线为例，一个从旧金山到阿拉斯加朱诺的有向边，表示可以从旧金山飞朱诺，但是没有从朱诺飞旧金山的飞机。

![One-way flights](Images/FlightsDirected.png)

下面的（结构）同样属于图：

![Tree and linked list](Images/TreeAndList.png)

左边是一个[树（tree）](../Tree/)结构，右边是一个[链表（linked list）](../Linked%20List/)结构。它们都可以被视为一种简单的图。它们都有点（节点）和边（连接）。

最先展示点图中包含了一个*环*，你可以从某点出发，沿着路径，最终又回到出发点。树是没有环的图。
The first graph includes *cycles*, where you can start off at a vertex, follow a path, and come back to the original vertex. A tree is a graph without such cycles.

另一种常见的图就是*有向无环图（directed acyclic graph）*，简称DAG：

![DAG](Images/DAG.png)

与树类似，这个图中没有环（无论你从何处开始，都无法会到起始点），
Like a tree, this graph does not have any cycles (no matter where you start, there is no path back to the starting vertex), but this graph has directional edges with the shape that does not necessarily form a hierarchy.

## 为什么要用图呢？

可能你已经开始耸着肩膀（译者：现在都是黑线了）想了，这有个毛用啊？随着牵出的就是图这个有用的数据结构。
Maybe you're shrugging your shoulders and thinking, what's the big deal? Well, it turns out that a graph is a useful data structure.

如果你再遇到可以用点和线表示的编程问题，就可以用图将其画出来，然后在借助如：[广度优先搜索（breadth-first search）](../Breadth-First%20Search/) 或 [深度优先搜索（depth-first search）](../Depth-First%20Search)等图的算法知识找到相应的解决方案。

例如，假设有个任务列表，其中的某些任务必须等其它某些任务完毕后才能开始。那么就可以通过一个有向无环图来为其建模：

![Tasks as a graph](Images/Tasks.png)

每个点表示一个任务。两个点之间的边表示任务的顺序依赖关系。例如，任务 C 只能在 B 和 D 都结束后才能开始，而 B 或 D 则只能在 A 结束之后开始。

现在问题已经用图表示出来了，之后便可通过深度优先算法执行[拓扑排序（topological sort）](../Topological%20Sort/)。这将对任务的顺序进行优化，以减少等待的时间。（一种排序方案：A, B, D, E, C, F, G, H, I, J, K.）

无论何时遇到多么棘手的编程问题，都问自己，“怎么用图来表示这个问题呢？” 图总是能呈现出数据之间的关系。关键在于你如何定义“关系”
Whenever you are faced with a tough programming problem, ask yourself, "how can I express this problem using a graph?" Graphs are all about representing relationships between your data. The trick is in how you define "relationships".

If you are a musician you might appreciate this graph:

![Chord map](Images/ChordMap.png)

The vertices are chords from the C major scale. The edges -- the relationships between the chords -- represent how [likely one chord is to follow another](http://mugglinworks.com/chordmaps/genmap.htm). This is a directed graph, so the direction of the arrows shows how you can go from one chord to the next. It is also a weighted graph, where the weight of the edges -- portrayed here by line thickness -- shows a strong relationship between two chords. As you can see, a G7-chord is very likely to be followed by a C chord, and much less likely by a Am chord.

You are probably already using graphs without even knowing it. Your data model is also a graph (from Apple's Core Data documentation):

![Core Data model](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/CoreDataVersioning/Art/recipe_version2.0.jpg)

Another common graph used by programmers is the state machine, where edges depict the conditions for transitioning between states. Here is a state machine that models my cat:

![State machine](Images/StateMachine.png)

Graphs are awesome. Facebook made a fortune from their social graph. If you are going to learn any data structure, you must choose the graph and the vast collection of standard graph algorithms.

## Vertices and edges, oh my!

理论上，图就是一堆点和边对象，那么用代码这么表示呢？
In theory, a graph is just a bunch of vertex and edge objects, but how do you describe this in code?

有两种主要方式：邻接表（adjacency list）和邻接矩阵（adjacency matrix）。

**邻接表（Adjacency List）** 在邻接表的视线中，每个点都有一张表，用于存储所有从该点发出的边。例如，如果点 A 有连接 B，C，
和D的边，那么点 A 的表中就会存有 3 条边。
**Adjacency List.** In an adjacency list implementation, each vertex stores a list of edges that originate from that vertex. For example, if vertex A has an edge to vertices B, C, and D, then vertex A would have a list containing 3 edges.

![Adjacency list](Images/AdjacencyList.png)

邻接表表示的是发出的边。A 有到 B 的边，但是 B 并没有到 A 的边，所以 A 不会出现在 B 的邻接表中。要查找某两个点之间的边或者边的量是非常昂贵的，因为它的访问不是随机的。必须所有邻接表直至找到。
The adjacency list describes outgoing edges. A has an edge to B, but B does not have an edge back to A, so A does not appear in B's adjacency list. Finding an edge or weight between two vertices can be expensive because there is no random access to edges. You must traverse the adjacency lists until it is found.

**邻接矩阵（Adjacency Matrix）** 在一个邻接矩阵的实现中，通过行列上的量来表示点之间是否连接，以及以什么量连接。例如，一个从 A 到 B 的有向边的量是 5.6，那么 A 行 B 列交点处的值即为 5.6：
**Adjacency Matrix.** In an adjacency matrix implementation, a matrix with rows and columns representing vertices stores a weight to indicate if vertices are connected and by what weight. For example, if there is a directed edge of weight 5.6 from vertex A to vertex B, then the entry with row for vertex A and column for vertex B would have the value 5.6:

![Adjacency matrix](Images/AdjacencyMatrix.png)

想图中额外增加一个点是昂贵的，这是因为一个新的矩阵结构必须创建足够的空间以存储新的行/列，并且已有的结构还必须拷贝进来。
Adding another vertex to the graph is expensive, because a new matrix structure must be created with enough space to hold the new row/column, and the existing structure must be copied into the new one.

那么应该用那种呢？大多数时间，邻接表比较好。下面是两种方式的更多细节表交。
So which one should you use? Most of the time, the adjacency list is the right approach. What follows is a more detailed comparison between the two.

*V* 表示图中点的书里阿妮个，*E* 表示边的数量。如下表：

| 操作             |   邻接表        |    邻接矩阵       |
|-----------------|----------------|------------------|
| 存储空间         | O(V + E)       | O(V^2)           |
| 添加点           | O(1)           | O(V^2)           |
| 添加边           | O(1)           | O(1)             |
| 邻检             | O(V)           | O(1)             |

“邻检”表示，检查一个给定的点是不是另一个点的近邻。邻接表的邻检时间杂度是 **O(V)**，因为最坏时，一个点与其它*每个*点都有连接。
"Checking adjacency" means that we try to determine that a given vertex is an immediate neighbor of another vertex. The time to check adjacency for an adjacency list is **O(V)**, because in the worst case a vertex is connected to *every* other vertex.

在一个*稀疏*的图中，每个点仅与少数几个点相连，那么此时邻接表就是最好的方式，如果是一个*浓密*的图，每个点与其它大多数点都有连接，那么矩阵将是一个更好的选择。
In the case of a *sparse* graph, where each vertex is connected to only a handful of other vertices, an adjacency list is the best way to store the edges. If the graph is *dense*, where each vertex is connected to most of the other vertices, then a matrix is preferred.

下面是邻接表和邻接矩阵的简单实现：

## 代码: 边和点

每个点的邻接表都存有一系列的 `Edge` 对象：
The adjacency list for each vertex consists of `Edge` objects:

```swift
public struct Edge<T>: Equatable where T: Equatable, T: Hashable {

  public let from: Vertex<T>
  public let to: Vertex<T>

  public let weight: Double?

}
```

This struct describes the "from" and "to" vertices and a weight value. Note that an `Edge` object is always directed, a one-way connection (shown as arrows in the illustrations above). If you want an undirected connection, you also need to add an `Edge` object in the opposite direction. Each `Edge` optionally stores a weight, so they can be used to describe both weighted and unweighted graphs.

The `Vertex` looks like this:

```swift
public struct Vertex<T>: Equatable where T: Equatable, T: Hashable {

  public var data: T
  public let index: Int

}
```

It stores arbitrary data with a generic type `T`, which is `Hashable` to enforce uniqueness, and also `Equatable`. Vertices themselves are also `Equatable`.

## The code: graphs

> **Note:** There are many ways to implement graphs. The code given here is just one possible implementation. You probably want to tailor the graph code to each individual problem you are trying to solve. For instance, your edges may not need a `weight` property, or you may not have the need to distinguish between directed and undirected edges.

Here is an example of a simple graph:

![Demo](Images/Demo1.png)

We can represent it as an adjacency matrix or adjacency list. The classes implementing those concepts both inherit a common API from `AbstractGraph`, so they can be created in an identical fashion, with different optimized data structures behind the scenes.

Let's create some directed, weighted graphs, using each representation, to store the example:

```swift
for graph in [AdjacencyMatrixGraph<Int>(), AdjacencyListGraph<Int>()] {

  let v1 = graph.createVertex(1)
  let v2 = graph.createVertex(2)
  let v3 = graph.createVertex(3)
  let v4 = graph.createVertex(4)
  let v5 = graph.createVertex(5)

  graph.addDirectedEdge(v1, to: v2, withWeight: 1.0)
  graph.addDirectedEdge(v2, to: v3, withWeight: 1.0)
  graph.addDirectedEdge(v3, to: v4, withWeight: 4.5)
  graph.addDirectedEdge(v4, to: v1, withWeight: 2.8)
  graph.addDirectedEdge(v2, to: v5, withWeight: 3.2)

}
```

As mentioned earlier, to create an undirected edge you need to make two directed edges. For undirected graphs, we call the following method instead:

```swift
  graph.addUndirectedEdge(v1, to: v2, withWeight: 1.0)
  graph.addUndirectedEdge(v2, to: v3, withWeight: 1.0)
  graph.addUndirectedEdge(v3, to: v4, withWeight: 4.5)
  graph.addUndirectedEdge(v4, to: v1, withWeight: 2.8)
  graph.addUndirectedEdge(v2, to: v5, withWeight: 3.2)
```

We could provide `nil` as the values for the `withWeight` parameter in either case to make unweighted graphs.

## The code: adjacency list

To maintain the adjacency list, there is a class that maps a list of edges to a vertex. The graph simply maintains an array of such objects and modifies them as necessary.

```swift
private class EdgeList<T> where T: Equatable, T: Hashable {

  var vertex: Vertex<T>
  var edges: [Edge<T>]? = nil

  init(vertex: Vertex<T>) {
    self.vertex = vertex
  }

  func addEdge(_ edge: Edge<T>) {
    edges?.append(edge)
  }

}
```

They are implemented as a class as opposed to structs, so we can modify them by reference, in place, like when adding an edge to a new vertex, where the source vertex already has an edge list:

```swift
open override func createVertex(_ data: T) -> Vertex<T> {
  // check if the vertex already exists
  let matchingVertices = vertices.filter() { vertex in
    return vertex.data == data
  }

  if matchingVertices.count > 0 {
    return matchingVertices.last!
  }

  // if the vertex doesn't exist, create a new one
  let vertex = Vertex(data: data, index: adjacencyList.count)
  adjacencyList.append(EdgeList(vertex: vertex))
  return vertex
}
```

The adjacency list for the example looks like this:

```
v1 -> [(v2: 1.0)]
v2 -> [(v3: 1.0), (v5: 3.2)]
v3 -> [(v4: 4.5)]
v4 -> [(v1: 2.8)]
```

where the general form `a -> [(b: w), ...]` means an edge exists from `a` to `b` with weight `w` (with possibly more edges connecting `a` to other vertices as well).

## The code: adjacency matrix

We will keep track of the adjacency matrix in a two-dimensional `[[Double?]]` array. An entry of `nil` indicates no edge, while any other value indicates an edge of the given weight. If `adjacencyMatrix[i][j]` is not nil, then there is an edge from vertex `i` to vertex `j`.

To index into the matrix using vertices, we use the `index` property in `Vertex`, which is assigned when creating the vertex through the graph object. When creating a new vertex, the graph must resize the matrix:

```swift
open override func createVertex(_ data: T) -> Vertex<T> {
  // check if the vertex already exists
  let matchingVertices = vertices.filter() { vertex in
    return vertex.data == data
  }

  if matchingVertices.count > 0 {
    return matchingVertices.last!
  }

  // if the vertex doesn't exist, create a new one
  let vertex = Vertex(data: data, index: adjacencyMatrix.count)

  // Expand each existing row to the right one column.
  for i in 0 ..< adjacencyMatrix.count {
    adjacencyMatrix[i].append(nil)
  }

  // Add one new row at the bottom.
  let newRow = [Double?](repeating: nil, count: adjacencyMatrix.count + 1)
  adjacencyMatrix.append(newRow)

  _vertices.append(vertex)

  return vertex
}
```

Then the adjacency matrix looks like this:

	[[nil, 1.0, nil, nil, nil]    v1
	 [nil, nil, 1.0, nil, 3.2]    v2
	 [nil, nil, nil, 4.5, nil]    v3
	 [2.8, nil, nil, nil, nil]    v4
	 [nil, nil, nil, nil, nil]]   v5

	  v1   v2   v3   v4   v5


## 参见

该本讲述了什么是图，以及如果实现一个基本的数据结构。此外还有其它涉及图应用的文章，请一并参考吧！
This article described what a graph is, and how you can implement the basic data structure. We have other articles on practical uses of graphs, so check those out too!

*Written by Donald Pinckney and Matthijs Hollemans*
