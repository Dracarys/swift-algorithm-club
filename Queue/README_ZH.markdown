# 队列（Queue）

> 相关教程已发表在[Raywenderlich](https://www.raywenderlich.com/148141/swift-algorithm-club-swift-queue-data-structure)
> This topic has been tutorialized [here](https://www.raywenderlich.com/148141/swift-algorithm-club-swift-queue-data-structure)

队列是一个仅能尾进头出的列表。且保证第一个入队的对象也是第一个出队。先入先出！

为什么需要队列呢？在很多算法问题中，经常会遇到需要将一系列的对象添加到临时列表中，之后又按照添加到顺序逐个移除的情况。且顺序不能乱。（译者：其实跟生活中一样，比如排队购物付款，对，就是为保证先到先付款的顺序）

队列允许先进先出。第一个添加的元素也第一出队。绝对公平！（与之相似的数据结构是[栈（Stack）](../Stack/)，只不过它是后进先出）

下面想队列中添加一个数：

```swift
queue.enqueue(10)
```

此时队列是 `[ 10 ]`. 添加下一个数：

```swift
queue.enqueue(3)
```

队列变成了 `[ 10, 3 ]`。在添加一个数：

```swift
queue.enqueue(57)
```
此时队列变成了  `[ 10, 3, 57 ]`。让我们从头取出最初添加的数：

```swift
queue.dequeue()
```

返回 `10` 正好是我们第一次添加的数。现在队列变成了 `[ 3, 57 ]`。所有的元素都向前移动了一个位置。

```swift
queue.dequeue()
```

返回 `3`，在出队返回 `57`，以此类推。如果队列为空，会返回 `nil`，活着其它实现中返回错误信息。

> **注意:** 队列并不总是最好的选择，如果元素添加删除的顺序不重要，完全可以用[栈](../Stack/)来替代队列，它更简单高效。

## 上码（The code）

这是一个极度简化的队列 Swift 实现。通过对数组的简单封装来实现入队，出队，以及查看队首元素等功能：

```swift
public struct Queue<T> {
  fileprivate var array = [T]()

  public var isEmpty: Bool {
    return array.isEmpty
  }
  
  public var count: Int {
    return array.count
  }

  public mutating func enqueue(_ element: T) {
    array.append(element)
  }
  
  public mutating func dequeue() -> T? {
    if isEmpty {
      return nil
    } else {
      return array.removeFirst()
    }
  }
  
  public var front: T? {
    return array.first
  }
}
```

功能正常，但是不够效率。

由于是从数组末尾添加新元素，所以耗费的时间总是相同的，且于数组大小无关，是一个 **O(1)** 操作。

你可能会想：为什么向数组追加元素的操作耗时是恒定的 **O(1)** 呢？这是因为，在 Swift 中，数组的末尾总是保留一些空位。如果我们只想一些操作：

```swift
var queue = Queue<String>()
queue.enqueue("Ada")
queue.enqueue("Steve")
queue.enqueue("Tim")
```

之后数组会变成类似这样：

	[ "Ada", "Steve", "Tim", xxx, xxx, xxx ]

其中的 `xxx` 就是被保留但是还没有添加内容的内存。此时向数组中追加新元素，就会添加到空位中：

	[ "Ada", "Steve", "Tim", "Grace", xxx, xxx ]

这就导致在从一个地方想另一个地方拷贝内存是一个耗时恒定的操作。

数组末尾的空位是有限的。当最后一个空位被占用时，如果你还想在追加元素，那么数组就需要重设，以便有足够的空间。

重设（数组）需要开辟内存空间，并将原有的内容拷贝到新数组中。这是一个 **O(n)** 操作，相当耗时。

出队就不同了。要出队，首先要移除数组 *beginning（头部）* 的元素。这始终是一个 **O(n)** 操作，因为它需要将数组中所有的元素整体向前移动。

例如，出队元素`"Ada"`，需要将 `"Steve"` 拷贝到原 `"Ada"` 到位置，`"Tim"` 拷贝到 `"Steve"` 的位置, 而 `"Grace"` 要拷贝到原 `"Tim"` 的位置：

	before   [ "Ada", "Steve", "Tim", "Grace", xxx, xxx ]
	                   /       /      /
	                  /       /      /
	                 /       /      /
	                /       /      /
	 after   [ "Steve", "Tim", "Grace", xxx, xxx, xxx ]
 
在内存中移动这些元素始终是 **O(n)** 操作。所以在外面实现的这个简单队列，入队很高效，但出队的实现就不那么可取了。。。

## 更高效的队列（A more efficient queue）

为了使出队更高效，我们需要保留一些额外的自由空间，只是这次需要在前面保留。由于 Swift 内建的数组不支持，所以我们必须自己来实现这部分功能。

主要思路就是，无论怎么出队元素，都不在将数组中的元素向前移动，而是将该位置标记为空。将 `"Ada"` 出队后的数组是这样的：

	[ xxx, "Steve", "Tim", "Grace", xxx, xxx ]

在将 `"Steve"` 出队，此时数组：

	[ xxx, xxx, "Tim", "Grace", xxx, xxx ]

由于数组前面的空位不会在使用，所以可以周期性地对数组进行调整，将剩余元素想前移动：

	[ "Tim", "Grace", xxx, xxx, xxx, xxx ]

数组的调整还是产生内存移动的 **O(n)** 操作。但是因为它单位时间内仅发生一次，所以平均出队耗时变成了 **O(1)**。

下面是新版  `Queue` 的实现：

```swift
public struct Queue<T> {
  fileprivate var array = [T?]()
  fileprivate var head = 0
  
  public var isEmpty: Bool {
    return count == 0
  }

  public var count: Int {
    return array.count - head
  }
  
  public mutating func enqueue(_ element: T) {
    array.append(element)
  }
  
  public mutating func dequeue() -> T? {
    guard head < array.count, let element = array[head] else { return nil }

    array[head] = nil
    head += 1

    let percentage = Double(head)/Double(array.count)
    if array.count > 50 && percentage > 0.25 {
      array.removeFirst(head)
      head = 0
    }
    
    return element
  }
  
  public var front: T? {
    if isEmpty {
      return nil
    } else {
      return array[head]
    }
  }
}
```

现在数组的存储对象类型由 `T` 变成了 `T?`，这是因为我们需要将数组元素标记为空。`head` 变量用来标记数组前端。

绝大多数的逻辑都位于 `dequeue()` 中。当出队一个对象时，首先将 `array[head]` 设为 `nil` 以将其从数组中移除。之后增加 `head` ，因为之后的对象已经变成首对象。

变化前：

	[ "Ada", "Steve", "Tim", "Grace", xxx, xxx ]
	  head

变化后：

	[ xxx, "Steve", "Tim", "Grace", xxx, xxx ]
	        head

就像是超市里排队付款一样，只是排队的人不动，而收银台沿着队伍移动。

如果不移除前端空位，那么数组将随着我们的入队和出队不断增长。这就需要像下面这样，对数组进行裁切：

```swift
    let percentage = Double(head)/Double(array.count)
    if array.count > 50 && percentage > 0.25 {
      array.removeFirst(head)
      head = 0
    }
```

这里计算了数组前端空位与数组自身大小的比例。如果空位超过了25%，那么就将浪费的空位砍掉（回收），并且要求在裁切前数组中至少要有50个对象。

> **注意:** 这些数值都是凭空设定的，实际生产环境中具体设定还结合你应用的实际情况而定。

在 playground 中测试一下：

```swift
var q = Queue<String>()
q.array                   // [] empty array

q.enqueue("Ada")
q.enqueue("Steve")
q.enqueue("Tim")
q.array             // [{Some "Ada"}, {Some "Steve"}, {Some "Tim"}]
q.count             // 3

q.dequeue()         // "Ada"
q.array             // [nil, {Some "Steve"}, {Some "Tim"}]
q.count             // 2

q.dequeue()         // "Steve"
q.array             // [nil, nil, {Some "Tim"}]
q.count             // 1

q.enqueue("Grace")
q.array             // [nil, nil, {Some "Tim"}, {Some "Grace"}]
q.count             // 2
```

为了测试裁切效果，将这行代码替换，

```swift
    if array.count > 50 && percentage > 0.25 {
```

换成:

```swift
    if head > 2 {
```

现在如果你出队一个对象，数组将会变成下面这样：

```swift
q.dequeue()         // "Tim"
q.array             // [{Some "Grace"}]
q.count             // 1
```

前端的 `nil` 对象被移除了，数组也不在浪费（存储）空间。新版的 `Queue` 虽然相对之前版本变化不大，但由于我们更好的运用了数组，所以出队操作也变成了 **O(1)** 操作。

## 参见（Also see）

实现一个队列的方式还有很多。其它方案如：[链表（linked list）](../Linked%20List/)，[循环缓冲器（circular buffer）](../Ring%20Buffer/)，或者[堆（heap）](../Heap/)

此外还有变种[双端队列（deque）](../Deque/)，两端均可以执行出队和入队操作，以及[优先级队列（priority queue）](../Priority%20Queue/)，让最终重要的对象是中处于队列前端。

*由 Matthijs Hollemans 为 Swift 算法社区编写*

*由 William Han 翻译*
