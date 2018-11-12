# 优先队列（Priority Queue）

优先队列是一种其内最重要的元素总是位于前端的[queue](../Queue/)。

优先队列又分为*大元优先*队列（最大的元素在首）和*小元优先*队列（最小元素在首）。

## 为什么？

优先级队列通常用在那些需要处理大量元素，并且在处理过程中需要重复标记最大或最小值元素亦或是自定义个“重要元素”的算法中。

优先队列的主要应用场景：
- 模拟事件驱动。各个事件均带有时间戳，且需要按时间戳顺序执行。此时优先队列就非常容易找到下一个需要模拟的事件。
- Dijkstra算法就是通过优先队列求得图中的最短路径。
- 用于数据压缩的[霍夫曼编码（Huffman coding）](../Huffman%20Coding/)，该算法会构建一个压缩树。它需要反复查询两个尚无父节点的节点的最小频率。（译者：这里翻译不够准确。）
- 人工智能的 A* 算法
- 其它大量应用

对与常规队列或者传统的数组来说，需要不断地对整个序列进行查询，以便能找到下一个最大值。而优先队列正式为此而优化的。

## 做什么？

优先队列的常见操作：

- **入队**：向队列中插入一个新元素
- **出队**：移除并返回重要的那个元素
- **查找最小值** 或 **查找最大值**：返回但不删除最重要元素
- **修改优先级**: 调整已入队元素的优先级等级

## 如何实现

几种实现方式:

- [有序数组（sorted array）](../Ordered%20Array/)。最终要的元素位于数组末尾。缺点：由于必须有序插入，所以插入新元素较慢。
- 平衡的[二叉搜索树（binary search tree）](../Binary%20Search%20Tree/)。对于双端优先队列来说是最佳的，因为它同时实现了高效地“查询最大”和“查询最小”。 
- [堆（heap）](../Heap/)。 堆是优先队列最常用的数据结构，实际上，它俩经常作为同义词。堆比有序数组更高效，因为它仅需要部分有序。堆的操作堆时间杂度均为 **O(log n)**。

下面是一个基于堆的优先队列的 Swift 实现：

```swift
public struct PriorityQueue<T> {
  fileprivate var heap: Heap<T>

  public init(sort: (T, T) -> Bool) {
    heap = Heap(sort: sort)
  }

  public var isEmpty: Bool {
    return heap.isEmpty
  }

  public var count: Int {
    return heap.count
  }

  public func peek() -> T? {
    return heap.peek()
  }

  public mutating func enqueue(element: T) {
    heap.insert(element)
  }

  public mutating func dequeue() -> T? {
    return heap.remove()
  }

  public mutating func changePriority(index i: Int, value: T) {
    return heap.replace(index: i, value: value)
  }
}
```

如上所示，完全一样。如果一斤更有了[堆](../Heap/)，那么实现优先队列非常简单，因为堆*就是*一个优先队列。

## 参见

[优先队列——维基百科](https://en.wikipedia.org/wiki/Priority_queue)

*由 Matthijs Hollemans 发表于 Swift 算法社区*

*由 William Han 翻译*
