# 队列（Queue）

> 相关教程已发表在[Raywenderlich](https://www.raywenderlich.com/148141/swift-algorithm-club-swift-queue-data-structure)
> This topic has been tutorialized [here](https://www.raywenderlich.com/148141/swift-algorithm-club-swift-queue-data-structure)

队列是一个只能从尾添加新元素，从头移除元素的列表。确保第一个入队的元素也是第一个出队。先入先出。
A queue is a list where you can only insert new items at the back and remove items from the front. This ensures that the first item you enqueue is also the first item you dequeue. First come, first serve!

为什么需要队列呢？在很多算法问题中，经常会遇到需要将一系列的对象添加到临时列表中，之后又按照添加到顺序逐个移除的情况。且顺序不能乱。（译者：这给的理由还真是简单，咋不直接说因为需要队列所以有了队列呢。队列在计算机中的产生跟生活中一样，比如排队购物付款，对，就是为保证先到先付款的顺序）
Why would you need this? Well, in many algorithms you want to add objects to a temporary list and pull them off this list later. Often the order in which you add and remove these objects matters.

队列允许先进先出。第一个添加的元素也第一出队。绝对公平！（与之相似的数据结构是[栈](../Stack/)，只不过它是后进先出）
A queue gives you a FIFO or first-in, first-out order. The element you inserted first is the first one to come out. It is only fair! (A similar data structure, the [stack](../Stack/), is LIFO or last-in first-out.)

下面想队列中添加一个数：
Here is an example to enqueue a number:

```swift
queue.enqueue(10)
```

此时队列是 `[ 10 ]`. 添加下一个数：
The queue is now `[ 10 ]`. Add the next number to the queue:

```swift
queue.enqueue(3)
```

队列变成了 `[ 10, 3 ]`。在添加一个数：
The queue is now `[ 10, 3 ]`. Add one more number:

```swift
queue.enqueue(57)
```
此时队列变成了  `[ 10, 3, 57 ]`。让我们从头取出最初添加的数：
The queue is now `[ 10, 3, 57 ]`. Let's dequeue to pull the first element off the front of the queue:

```swift
queue.dequeue()
```

返回 `10` 正好是我们第一次添加的数。现在队列变成了 `[ 3, 57 ]`。所有的元素都向前移动了一个位置。
This returns `10` because that was the first number we inserted. The queue is now `[ 3, 57 ]`. Everyone moved up by one place.

```swift
queue.dequeue()
```

返回 `3`，在出队返回 `57`，以此类推。如果队列为空，会返回 `nil`，活着其它实现中返回错误信息。
This returns `3`, the next dequeue returns `57`, and so on. If the queue is empty, dequeuing returns `nil` or in some implementations it gives an error message.

> **Note:** 队列并不总是最好的选择，如果元素添加删除的顺序不重要，完全可以用[栈](../Stack/)来替代队列，它更简单高效。
> **Note:** A queue is not always the best choice. If the order in which the items are added and removed from the list is not important, you can use a [stack](../Stack/) instead of a queue. Stacks are simpler and faster.

## 上码（The code）

这是一个极度简化的队列 Swift 实现。通过对数组的简单封装来实现入队，出队，以及查看队首元素等功能：
Here is a simplistic implementation of a queue in Swift. It is a wrapper around an array to enqueue, dequeue, and peek at the front-most item:

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
This queue works well, but it is not optimal.

由于是从数组末尾添加新元素，所以耗费的时间总是相同的，且于数组大小无关，是一个  an **O(1)** 操作。
Enqueuing is an **O(1)** operation because adding to the end of an array always takes the same amount of time regardless of the size of the array.

你可能会想：为什么向数组追加元素的操作耗时是恒定的 **O(1)** 呢？这是因为，在 Swift 中，数组的末尾总是保留一些空位。如果我们只想一些操作：

You might be wondering why appending items to an array is **O(1)** or a constant-time operation. That is because an array in Swift always has some empty space at the end. If we do the following:

```swift
var queue = Queue<String>()
queue.enqueue("Ada")
queue.enqueue("Steve")
queue.enqueue("Tim")
```

之后数组会变成类似这样：
then the array might actually look like this:

	[ "Ada", "Steve", "Tim", xxx, xxx, xxx ]

其中的 `xxx` 就是被保留但是还没有添加内容的内存。此时向数组中追加新元素，就会添加到空位中：
where `xxx` is memory that is reserved but not filled in yet. Adding a new element to the array overwrites the next unused spot:

	[ "Ada", "Steve", "Tim", "Grace", xxx, xxx ]

这就导致在从一个地方想另一个地方拷贝内存是一个耗时恒定的操作。
This results by copying memory from one place to another which is a constant-time operation.

数组末尾的空位是有限的。当最后一个空位被占用时，如果你还想在追加元素，那么数组就需要重设，以便有足够的空间。
There are only a limited number of unused spots at the end of the array. When the last `xxx` gets used, and you want to add another item, the array needs to resize to make more room.

重设（数组）需要开辟内存空间，并将原有的内容拷贝到新数组中。这是一个 **O(n)** 操作，相当耗时。
Resizing includes allocating new memory and copying all the existing data over to the new array. This is an **O(n)** process which is relatively slow. Since it happens occasionally, the time for appending a new element to the end of the array is still **O(1)** on average or **O(1)** "amortized".

出队就不同了。要出队，首先要移除数组 *beginning（头部）* 的元素。这始终是一个 **O(n)** 操作，因为它需要将数组中所有的元素整体向前移动。
The story for dequeueing is different. To dequeue, we remove the element from the *beginning* of the array. This is always an **O(n)** operation because it requires all remaining array elements to be shifted in memory.

例如，出队元素`"Ada"`，需要将 `"Steve"` 拷贝到原 `"Ada"` 到位置，`"Tim"` 拷贝到 `"Steve"` 的位置, 而 `"Grace"` 要拷贝到原 `"Tim"` 的位置：

In our example, dequeuing the first element `"Ada"` copies `"Steve"` in the place of `"Ada"`, `"Tim"` in the place of `"Steve"`, and `"Grace"` in the place of `"Tim"`:

	before   [ "Ada", "Steve", "Tim", "Grace", xxx, xxx ]
	                   /       /      /
	                  /       /      /
	                 /       /      /
	                /       /      /
	 after   [ "Steve", "Tim", "Grace", xxx, xxx, xxx ]
 
在内存中移动这些元素始终是 **O(n)** 操作。所以在外面实现的这个简单队列，入队很高效，但出队的实现就不那么可取了。。。

Moving all these elements in memory is always an **O(n)** operation. So with our simple implementation of a queue, enqueuing is efficient, but dequeueing leaves something to be desired...

## 更高效的队列（A more efficient queue）

为了使出队更高效，我们需要保留一些额外的自由空间，只是这次需要在前面保留。由于 Swift 内建的数组不支持，所以我们必须自己来实现这部分功能。

To make dequeuing efficient, we can also reserve some extra free space but this time at the front of the array. We must write this code ourselves because the built-in Swift array does not support it.

主要思路就是，无论怎么出队元素，都不在将数组中的元素向前移动，而是将该位置标记为空。将 `"Ada"` 出队后的数组是这样的：

The main idea is whenever we dequeue an item, we do not shift the contents of the array to the front (slow) but mark the item's position in the array as empty (fast). After dequeuing `"Ada"`, the array is:

	[ xxx, "Steve", "Tim", "Grace", xxx, xxx ]

在将 `"Steve"` 出队，此时数组：
After dequeuing `"Steve"`, the array is:

	[ xxx, xxx, "Tim", "Grace", xxx, xxx ]

由于数组前面的空位不会在使用，所以可以周期性地对数组进行调整，将剩余元素想前移动：
Because these empty spots at the front never get reused, you can periodically trim the array by moving the remaining elements to the front:

	[ "Tim", "Grace", xxx, xxx, xxx, xxx ]

数组的调整还是产生内存移动的 **O(n)** 操作。但是因为它单位时间内仅发生一次，所以平均出队耗时变成了 **O(1)**。
This trimming procedure involves shifting memory which is an **O(n)** operation. Because this only happens once in a while, dequeuing is **O(1)** on average.

下面是新版  `Queue` 的实现：
Here is how you can implement this version of `Queue`:

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

现在数组的存储对象类型由 `T` 变成了 `T?`，这是因为我们需要将数组元素标记为空。`head` 变量用来标记数组前端
The array now stores objects of type `T?` instead of just `T` because we need to mark array elements as being empty. The `head` variable is the index in the array of the front-most object.

Most of the new functionality sits in `dequeue()`. When we dequeue an item, we first set `array[head]` to `nil` to remove the object from the array. Then, we increment `head` because the next item has become the front one.

We go from this:

	[ "Ada", "Steve", "Tim", "Grace", xxx, xxx ]
	  head

to this:

	[ xxx, "Steve", "Tim", "Grace", xxx, xxx ]
	        head

It is like if in a supermarket the people in the checkout lane do not shuffle forward towards the cash register, but the cash register moves up the queue.

If we never remove those empty spots at the front then the array will keep growing as we enqueue and dequeue elements. To periodically trim down the array, we do the following:

```swift
    let percentage = Double(head)/Double(array.count)
    if array.count > 50 && percentage > 0.25 {
      array.removeFirst(head)
      head = 0
    }
```

This calculates the percentage of empty spots at the beginning as a ratio of the total array size. If more than 25% of the array is unused, we chop off that wasted space. However, if the array is small we do not resize it all the time, so there must be at least 50 elements in the array before we try to trim it. 

> **Note:** I just pulled these numbers out of thin air -- you may need to tweak them based on the behavior of your app in a production environment.

To test this in a playground, do the following:

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

To test the trimming behavior, replace the line,

```swift
    if array.count > 50 && percentage > 0.25 {
```

with:

```swift
    if head > 2 {
```

Now if you dequeue another object, the array will look as follows:

```swift
q.dequeue()         // "Tim"
q.array             // [{Some "Grace"}]
q.count             // 1
```

The `nil` objects at the front have been removed, and the array is no longer wasting space. This new version of `Queue` is not more complicated than the first one but dequeuing is now also an **O(1)** operation, just because we were aware about how we used the array.

## See also

There are many ways to create a queue. Alternative implementations use a [linked list](../Linked%20List/), a [circular buffer](../Ring%20Buffer/), or a [heap](../Heap/). 

Variations on this theme are [deque](../Deque/), a double-ended queue where you can enqueue and dequeue at both ends, and [priority queue](../Priority%20Queue/), a sorted queue where the "most important" item is always at the front.

*Written for Swift Algorithm Club by Matthijs Hollemans*
