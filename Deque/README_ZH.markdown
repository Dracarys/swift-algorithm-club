# 双端队列（Deque）

双端队列（Deque）。由于某些原因通常读作 "deck"。（译者：避免跟“鸡儿”的发音一样）

常规 [队列（queue）](../Queue/)允许在后端添加元素，从前端移除元素。双端队列不仅如此还支持前端入队和后端出队，以及双端查看。
下面是一个双端队列的 SWift 基本实现：

```swift
public struct Deque<T> {
  private var array = [T]()

  public var isEmpty: Bool {
    return array.isEmpty
  }

  public var count: Int {
    return array.count
  }

  public mutating func enqueue(_ element: T) {
    array.append(element)
  }

  public mutating func enqueueFront(_ element: T) {
    array.insert(element, at: 0)
  }

  public mutating func dequeue() -> T? {
    if isEmpty {
      return nil
    } else {
      return array.removeFirst()
    }
  }

  public mutating func dequeueBack() -> T? {
    if isEmpty {
      return nil
    } else {
      return array.removeLast()
    }
  }

  public func peekFront() -> T? {
    return array.first
  }

  public func peekBack() -> T? {
    return array.last
  }
}
```

这里在内部使用了数组。出队和入队就是简单地从数组前端或后端添加和移除对象。

一个在 playground 中应用的例子：

```swift
var deque = Deque<Int>()
deque.enqueue(1)
deque.enqueue(2)
deque.enqueue(3)
deque.enqueue(4)

deque.dequeue()       // 1
deque.dequeueBack()   // 4

deque.enqueueFront(5)
deque.dequeue()       // 5
```

上面对 `Deque` 的实现比较简单，但是不够高效。除了 `enqueueFront()` 和 `dequeue()`，其它操作的时间杂度都是 **O(n)**。这里主要是为了向大家展示双端队列的工作原理。

## 高效版（A more efficient version）

之所以 `dequeue()` 和 `enqueueFront()` 的时间杂度是 **O(n)**，是因为它们都是在数组的前端进行（操作）。一旦从数组前端移除一个元素，那么数组中剩余的元素在内存中都会整体向前移动。

假设双端队列的数组包含以下元素：

	[ 1, 2, 3, 4 ]

当数组中 `dequeue()` 移除 `1` 时，那么剩余的元素 `2`, `3`, 和 `4` 都会向前移动一个位置：

	[ 2, 3, 4 ]

由于所有元素在内存中都必须向前移动一个位置，所以这是一个 **O(n)** 操作。

同理，在数组前端插入一个元素同样昂贵，因为数组中已有元素必须全部向后移动一个位置。`enqueueFront(5)` 后的数组如下：

	[ 5, 2, 3, 4 ]

首先，元素 `2`, `3`, 和 `4` 会在内存中会移动一个位置，之后在将新元素 `5` 插入到原来 `2` 的位置。

那么为什么 `enqueue()` 和 `dequeueBack()` 就不存在这个问题呢？这是因为这些操作恰好是在数组后端执行的。Swift 中的数组在末尾保留了一定的空闲空间。

我们初始化的数组 `[ 1, 2, 3, 4]` 其在内存中实际类似于下面这样：

	[ 1, 2, 3, 4, x, x, x ]

其中的 `x` 表示数组中的额外空闲位置。此时调用 `enqueue(6)` ，只是将新对象拷贝到紧邻的下一个空位而已：

	[ 1, 2, 3, 4, 6, x, x ]

而 `dequeueBack()` 方法是通过 `array.removeLast()` 来删除对象的。它仅仅是将 `array.count` 减去一，不会减小数组的实际内存空间。由于没有昂贵的内存拷贝需求，所以在数组尾端的操作都非常之快， **O(1)**。

数组是有可能耗尽末尾的空闲内存的。此时 Swift 会开辟一块新的内存，并将原来所有的数据拷贝到新数组中。这虽然是一个 **O(n)** 操作，但是由于在一段时间内仅会发生一次，所以平均下来，在数组末尾添加元素的时间杂度仍然是  **O(1)**。 

当然，我们也可以将这个技巧应用到数组的 *前端（beginning）* ，以便双端队列的前端操作更加高效。这时的数组看起来是这样的：

	[ x, x, x, 1, 2, 3, 4, x, x, x ]

这样数组前端也有了一大块空闲空间，再在前端进行添加或移除操作的时间杂度也将是 **O(1)**。

下面是新版的 `Deque`：

```swift
public struct Deque<T> {
  private var array: [T?]
  private var head: Int
  private var capacity: Int
  private let originalCapacity:Int

  public init(_ capacity: Int = 10) {
    self.capacity = max(capacity, 1)
    originalCapacity = self.capacity
    array = [T?](repeating: nil, count: capacity)
    head = capacity
  }

  public var isEmpty: Bool {
    return count == 0
  }

  public var count: Int {
    return array.count - head
  }

  public mutating func enqueue(_ element: T) {
    array.append(element)
  }

  public mutating func enqueueFront(_ element: T) {
    // this is explained below
  }

  public mutating func dequeue() -> T? {
    // this is explained below
  }

  public mutating func dequeueBack() -> T? {
    if isEmpty {
      return nil
    } else {
      return array.removeLast()
    }
  }

  public func peekFront() -> T? {
    if isEmpty {
      return nil
    } else {
      return array[head]
    }
  }

  public func peekBack() -> T? {
    if isEmpty {
      return nil
    } else {
      return array.last!
    }
  }  
}
```

变化不大， `enqueue()` 和 `dequeueBack()` 甚至都没改。其中最关键的变化是，数组的存储类型从 `T` 变成了 `T?`。因为需要将数组的元素标记为空。

 `init` 会初始化一个包涵了若干 `nil` 值的数组。默认的参数 10 正是我们在数组前端所预留的空位。

 `head` 变量用来标识数组真实的起始索引。队列当前为空，`head` 指向了一个越界的数组索引位置。

	[ x, x, x, x, x, x, x, x, x, x ]
	                                 |
	                                 head
每当需要在前端入队时，我们就将 `head` 向左移动一位并将新的对象拷贝到数组的 `head` 索引位。`enqueueFront(5)` 如下：

	[ x, x, x, x, x, x, x, x, x, 5 ]
	                             |
	                             head

紧接着 `enqueueFront(7)`：

	[ x, x, x, x, x, x, x, x, 7, 5 ]
	                          |
	                          head

依此类推， `head` 继续向左移动并始终指向队列中的第一个元素。由于仅需向数组中拷入新值，所以`enqueueFront()` 的操作耗时是一个常数，**O(1)**。

代码如下：

```swift
  public mutating func enqueueFront(element: T) {
    head -= 1
    array[head] = element
  }
```

在队尾添加元素没什么变化（与之前的代码实现相同）。例如，`enqueue(1)` ：

	[ x, x, x, x, x, x, x, x, 7, 5, 1, x, x, x, x, x, x, x, x, x ]
	                          |
	                          head

注意数组是如何进行扩容的。当发现已经没空位放值 `1` 时，Swift 会将创建一个更大的数组并在末尾添加一定量的空位。如果此时再次入队一个对象，它将被添加到下一个紧邻的空位中，例如， `enqueue(2)`：

	[ x, x, x, x, x, x, x, x, 7, 5, 1, 2, x, x, x, x, x, x, x, x ]
	                          |
	                          head

> **注意：** 当你调用 `print(deque.array)` 时，是看不见这些空位的。这是因为 Swift 为你隐藏了它们，只有前面（我们）添加的会被打印出来。

 `dequeue()` 方法恰好与 `enqueueFront()` 相反，它从 `head` 读取一个值，并其重设为 `nil` ，之后在将 `head` 向右移动一位：

```swift
  public mutating func dequeue() -> T? {
    guard head < array.count, let element = array[head] else { return nil }

    array[head] = nil
    head += 1

    return element
  }
```

还有一点小问题。。。如果在前端插入大量对象，那么前端预留的空位就可能会耗光。当这种情况发生在后端时，Swift 会自动为我们扩容。但是在前端我们就必须自己手动处理了，在 `enqueueFront()` 添加一些额外的处理逻辑：

```swift
  public mutating func enqueueFront(element: T) {
    if head == 0 {
      capacity *= 2
      let emptySpace = [T?](repeating: nil, count: capacity)
      array.insert(contentsOf: emptySpace, at: 0)
      head = capacity
    }

    head -= 1
    array[head] = element
  }
```

如果 `head` 等于 0，前端已经没有空闲空间。此时我们就在数组（前端插入）一定量的 `nil`。虽然这是一个 **O(n)** 操作，但是将其耗时均摊到所有的 `enqueueFront()` 操作中，那么平均下来每次的 `enqueueFront()`  的时间杂度仍是 **O(1)**。

> **注意:** 每次我们都将容量扩充一倍，因此队列会变的越来越大，随之扩容发生的频率也会越来越低。这也是为什么 Swift 数组会自动在后台（扩容）的原因。（译者：可以自行查看Swift[数组源码（Array.swift）](https://github.com/apple/swift/blob/master/stdlib/public/core/Array.swift)）

同样我们也比对 `dequeue()` 进行相应处理。如果你总是在队尾入队，队首出队，如此操作大量元素之后，那么数组可会呈现如下样式：

	[ x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, 1, 2, 3 ]
	                                                              |
	                                                              head

队首空位只有在调用 `enqueueFront()` 时才会被使用。但如果很少在队首入队，那么就会导致浪费大量内存空间。所以我们来添加一些代码在出队时将（多余的）空位清理掉：

```swift
  public mutating func dequeue() -> T? {
    guard head < array.count, let element = array[head] else { return nil }

    array[head] = nil
    head += 1

    if capacity >= originalCapacity && head >= capacity*2 {
      let amountToRemove = capacity + capacity/2
      array.removeFirst(amountToRemove)
      head -= amountToRemove
      capacity /= 2
    }
    return element
  }
```

回想一下， `capacity` 是我们在队首预留的空位数。如果 `head` 已经向右超过了该数值的两倍，那么就触发空位裁切。将其裁剪到 25%。

> **注意：** 队列会比较 `capacity` 和 `originalCapacity`，以保证（裁切后的）空位不少于初始。

例如下面这样:

	[ x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, x, 1, 2, 3 ]
	                                |                             |
	                                capacity                      head

裁切后:

	[ x, x, x, x, x, 1, 2, 3 ]
	                 |
	                 head
	                 capacity

如此我们就可以在前端出入队和内存有效控制中取得一个平衡。

> **注意：** 太小对数组不会进行裁切，节省几个比特的内存没必要。

## 参见（See also）

此外队列还可以通过[双链表（doubly linked list）](../Linked%20List/), [环形缓冲器（circular buffer）](../Ring%20Buffer/)或者一对反向[栈（stacks）](../Stack/)来实现。

[完整的SWift双端队列实现](https://github.com/lorentey/Deque)

*由 Matthijs Hollemans 发表于 Swift 算法社区*

*由 William Han 翻译*