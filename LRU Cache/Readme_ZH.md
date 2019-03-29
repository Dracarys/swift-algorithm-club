# LRU 缓存
缓存用来在内存中（临时）保存对象。缓存的大小是有限的；一旦系统没有足够的内存，缓存就必须被清理，否则将导致工程崩溃。[Least Recently Used][1] (LRU)是目前比较流行的一种缓存算法。

在这个 LRU 缓存实现中，在初始化的过程中就声明了一个容量大小，任何会导致容量溢出的添加，都会触发缓存清理最近经常使用的元素。通过 *优先级队列（priority queue）* 来确保这一行为。


## 优先级队列（The priority queue）

LRU 缓存的关键就在于优先级队列。为了简便，我们通过链表来构建它。所有与 LRU 缓存的交互都将转化为对链表维护，通过调用 `get` 和 `set` 更新优先级队列的状态，以此来反映出最近访问的元素。

### 技术点

#### 添加值

每次访问元素时，无论 `set` 还是 `get` 都要将元素插入到优先级连的头部，因此我们需要一个方法来协助完成这一操作：

```swift
private func insert(_ key: KeyType, val: Any) {
	cache[key] = val
	priority.insert(key, atIndex: 0)
	guard let first = priority.first else {
		return
	}
	key2node[key] = first
}
```

#### 清理缓存

缓存一旦满了，就必须对那些最近最少使用的元素进行清理，以便腾出空间。此时，我们就需要`remove` 那些最低有限的节点。操作如下：

```swift
private func remove(_ key: KeyType) {
	cache.removeValue(key)
	guard let node = key2node[key] else {
		return
	}
	priority.remove(node)
	key2node.removeValue(key)
}
```

### 效率优化

对于 LRU 缓存来说，从优先级队列中移除元素是一个常用操作。即便优先级队列是基于链表实现的，这也是一时间复杂度为 `O(n)` 的昂贵操作，它将成为 `set` 和 `get` 方法的瓶颈。

为了缓解这个问题，可以通过一张哈希表来保存对每个节点的引用：
```
private var key2node: [KeyType: LinkedList<KeyType>.LinkedListNode<KeyType>] = [:]
```

*由 Kai Chen，Kelvin Lau，联合发表于 Swift 算法社区*

*由 William Han 翻译*


[1]:	https://en.wikipedia.org/wiki/Cache_replacement_policies#LRU
