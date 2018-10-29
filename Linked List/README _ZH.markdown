# 链表（Linked List）
# Linked List

> 对应的[教程](https://www.raywenderlich.com/144083/swift-algorithm-club-swift-linked-list-data-structure)
> This topic has been tutorialized [here](https://www.raywenderlich.com/144083/swift-algorithm-club-swift-linked-list-data-structure)

链表是一连串的数据对象，像数组一样。但与数组不同的是，数组需要在内存中开辟一大块内存用来存储对象，而链表中的对象则是分散存储在内存中，它们通过一条链链接在一起：

	+--------+    +--------+    +--------+    +--------+
	|        |    |        |    |        |    |        |
	| node 0 |--->| node 1 |--->| node 2 |--->| node 3 |
	|        |    |        |    |        |    |        |
	+--------+    +--------+    +--------+    +--------+

链表中的元素被引用为 *nodes（节点）*。上图展示了一个 *singly linked list（单向链表）*，这里每个节点都有一个指向下一个节点的引用或者说是“指针”。但是在如下图所示的 *doubly linked list（双向链表）*中，每个节点还有一个指向前一节点的指针：

	+--------+    +--------+    +--------+    +--------+
	|        |--->|        |--->|        |--->|        |
	| node 0 |    | node 1 |    | node 2 |    | node 3 |
	|        |<---|        |<---|        |<---|        |
	+--------+    +--------+    +--------+    +--------+

通常一个链表的首节点包含一个成为 *head（头）* 的指针：

	         +--------+    +--------+    +--------+    +--------+
	head --->|        |--->|        |--->|        |--->|        |---> nil
	         | node 0 |    | node 1 |    | node 2 |    | node 3 |
	 nil <---|        |<---|        |<---|        |<---|        |<--- tail
	         +--------+    +--------+    +--------+    +--------+

对尾部的引用也非常重要，被称为 *tail（尾）*。注意，最后一个节点的“next”指针和首节点的“pevious”指针一样，均为“nil”

## 链表的执行效率（Performance of linked lists）
## Performance of linked lists

对于大多数链表来说，它的时间复杂度都是 **O(n)** 。所以链表通常要比数组慢。但是相对于数组动则就要拷贝整块内存来说，链表的操作更具灵活领，大多仅需要改变几个指针即可。

之所以（链表的）时间杂度是 **O(n)**，是因为你不能简单地通过 `list[2]` 去访问 2 号节点。如果你现在没有 2 号节点的引用，那么就必须从 `head` 开始，通过 `next` 指针，一路查找下去（或者从 `tail`开始，通过 `previous` 指针，一路查找下去）。

一旦你拥有了某个节点的引用，那么无论是执行插入还是删除操作均非常便捷。只是查找节点较慢。

也就是说，操作链表时，始终应该在前面插入新节点，这一操作的时间杂度是 **O(1)**。相反如果你持有的是链表的 `tail` 指针，那么你应该是中在后面插入新节点。

## 单向 VS 双向链表（Singly vs doubly linked lists）
## Singly vs doubly linked lists

相对于双向链表，由于单向链表无需保存 `previous`指针，所以它消耗的内存更少。

但是，如果你需要查找一个已有节点的前一个节点，那就麻烦了。你必须从头开始遍历，直到找到正确的节点。

针对大多数应用场景，双向链表更容易一些。

## 为什么采用链表？(Why use a linked list?)
## Why use a linked list?

链表的一个典型应用场景就是[queue(队列)](../Queue/)。用数组实现的队列，移除前端的一个元素相对缓慢，这是因为你必须先将其他元素出栈。队列就不同了，你只要将 `head` 指向第二个元素即可，操作非常之快。

但是说实话，工作中你几乎没多少机会去实现自己的链表。但是理解链表的原理还是非常有帮助的；其中涉及到的对象链接原理同样适用于[trees(数)](../Tree/) 和 [graphs(图)](../Graph/)

## 代码（The code）
## The code

我们首先定义一个节点：

```swift
public class LinkedListNode<T> {
  var value: T
  var next: LinkedListNode?
  weak var previous: LinkedListNode?

  public init(value: T) {
    self.value = value
  }
}
```

这里运用了泛型，所以 `T` 可以表示任何你即将存储于节点中的数据类型。后续都将以字符串为例。

我们采用双向链表，每个节点都拥有一个 `next` 和一个 `previous` 指针（译者注：原文为pointer，但 swift 中实际是一个引用）。因为它可能没有下一个或者前一个节点，所以采用可选类型。（后续会逐个指出，那些在单向链表中需要改动的函数）

> **Note:** 为了避免循环持有，这里我们将 `previous` 声明为弱引用。假设有一个节点 `A` 其后跟着一个节点 `B`, 此时 `A` 指向 `B` 但 `B` 也同样指向 `A`。这就出现了循环持有的情况，即使你删除了这个两个节点，由于他们互相持有也将持续存活（于内存中）。这不是我们期望的，所以我们将其中的一个引用（译者注：原文为‘pointer’，但Swift中应为引用）设为 `weak` 以打破这种循环。

让我们开始构建链表。下面是第一步：

```swift
public class LinkedList<T> {
  public typealias Node = LinkedListNode<T>

  private var head: Node?

  public var isEmpty: Bool {
    return head == nil
  }

  public var first: Node? {
    return head
  }
}
```

我们期望一个类的命名越具描述性越好，但是我们又不想每次使用该类时都打很多的字，因此在 `LinkedList` 内部，我们使用类型别名 `Node` 来取代 `LinkedListNode<T>` 。

该链表中仅有一个 `head` 引用，没有尾部。这里就把添加尾部指针作为练习留给读者。（教程中会指出，如果存在尾指针，函数会有哪些不同）

当 `head` 为 nil 时，该链表为空。由于 `head` 是一个私有变量，此处还添加了一个 `first` 属性，以便返回链表的第一个节点。

你可以在Playground中尝试一下，类似这样：

```swift
let list = LinkedList<String>()
list.isEmpty   // true
list.first     // nil
```

我们在添加一个属性，用于返回链表的尾节点。有趣的地方开始了：

```swift
  public var last: Node? {
    guard var node = head else {
      return nil
    }
  
    while let next = node.next {
      node = next
    }
    return node
  }
```

如果你刚接触 Swift，你可能对 `if let` 有印象，但是对 `if var` 较陌生。他们的作用是相同的，对可选类型 `head` 进行解封，并将解封的结果赋值给一个名为 `node` 的局部变量。唯一不同就是这里的 `node` 是一个变量而非常量，因此可以在循环的内部对其进行修改。

这个循环也施了一些 Swift 魔法。 `while let next = node.next` 持续循环直到 `node.next` 为 nil。你可以写成下面这样：

```swift
      var node: Node? = head
      while node != nil && node!.next != nil {
        node = node!.next
      }
```

但这样给我感觉不够 Swift。我们尽可能使用 Swift 内建支持的解封方式。之后的代码中你还会看到很多类似对循环。

> **Note:** 如果我们持有一个尾部指针，那么 `last` 直接返回 `return tail` 即可。但如果没有，那么就必须从头部开始遍历整条链直到尾部。这种操作是非常昂贵的，尤其是当链很长时。（译者注：持有尾部，获取尾部的时间复杂度为 O(1)，否则将变为O(n)）

当然，`last` 只会返回 nil，因为链表中还没有任何节点。我们来添加一个可以向链表尾部添加新节点的方法：

```swift
  public func append(value: T) {
    let newNode = Node(value: value)
    if let lastNode = last {
      newNode.previous = lastNode
      lastNode.next = newNode
    } else {
      head = newNode
    }
  }
```

 `append()` 方法首先创建一个节点，之后访问 `last` 属性。如果不存节点，链表为空，那么就将 `head` 指向刚刚创建的新 `Node`。但是如果找到了一个有效的节点，就更新 `next` 和 `previous`指针，将新创建的节点添加到链中。链表中的大量代码都是针对 `next` 和 `previous` 的指针操作。

我们在 palyground 中测试一下：

```swift
list.append("Hello")
list.isEmpty         // false
list.first!.value    // "Hello"
list.last!.value     // "Hello"
```

此时链表如下图所示：

	         +---------+
	head --->|         |---> nil
	         | "Hello" |
	 nil <---|         |
	         +---------+

现在添加第二个节点：

```swift
list.append("World")
list.first!.value    // "Hello"
list.last!.value     // "World"
```

此时链表是这样的：

	         +---------+    +---------+
	head --->|         |--->|         |---> nil
	         | "Hello" |    | "World" |
	 nil <---|         |<---|         |
	         +---------+    +---------+

你可以通过查看 `next` 和 `previous` 来进行校验：

```swift
list.first!.previous          // nil
list.first!.next!.value       // "World"
list.last!.previous!.value    // "Hello"
list.last!.next               // nil
```

我们在来添加一个统计链表中节点数的方法。跟前面做法非常类似：

```swift
  public var count: Int {
    guard var node = head else {
      return 0
    }
  
    var count = 1
    while let next = node.next {
      node = next
      count += 1
    }
    return count
  }
```

以同样的方式遍历整个链，不同是同时进行了计数。

> **Note:** 一个将 `count` 的效率从 **O(n)** 提升 **O(1)** 的方式是，维护一个变量，无论你添加还是删除一个节点，都对该变量进行更新。

如果我们想查找某个索引节点怎么办？在数组中我们可以通过下标 `array[index]` 去访问，该访问耗时 **O(1)**。链表虽相对复杂一些，但又一次应用了相同的模式。

```swift
  public func node(atIndex index: Int) -> Node {
    if index == 0 {
      return head!
    } else {
      var node = head!.next
      for _ in 1..<index {
        node = node?.next
        if node == nil { //(*1)
          break
        }
      }
      return node!
    }
  }
```

首先，我们检查索引是否为 0。如果是 0，直接返回 head(头) 即可。但如果索引大于 0，那么还是从头开始，沿着 next 指针对链表进行遍历。与之前计数方法不同的是，这一次拥有两个终止条件。一个是 for 循环表达式已抵达指定的索引，并且我们可以去的相应索引的节点。另一个是 for 循环表达式中的 `node.next` 返回空值退出循环。也就是说给定的索引越界并崩溃。

试一下：

```swift
list.nodeAt(0)!.value    // "Hello"
list.nodeAt(1)!.value    // "World"
// list.nodeAt(2)           // crash
```

出于好玩，我们还可以实现其 `subscript` 方法:

```swift
  public subscript(index: Int) -> T {
    let node = node(atIndex: index)
    return node.value
  }
```

现在你可以这样访问了：

```swift
list[0]   // "Hello"
list[1]   // "World"
list[2]   // crash!
```
因为没有（2号）节点，所以 `list[2]` 崩溃了。

截止目前我们已经实现了想链表尾部添加节点的方法，但是很低效，因为你需要在添加前先找到尾部（如果持有尾部指针的话将非常高效）。所以，如果链表内元素顺序无关紧要的话，你就应该在链表的头部插入节点。这也是一个 **O(1)** 的操作。

我们来实现一个可以在任意位置插入新节点的方法：

```swift
  public func insert(_ node: Node, atIndex index: Int) {
   let newNode = node
   if index == 0 {
     newNode.next = head                      
     head?.previous = newNode
     head = newNode
   } else {
     let prev = self.node(atIndex: index-1)
     let next = prev.next

     newNode.previous = prev
     newNode.next = prev.next
     prev.next = newNode
     next?.previous = newNode
   }
}
```

像 `node(atIndex :)` 方法一样，`insert(_: at:)` 方法同样需要判断给定索引是否为 0.

首先我们回顾一下之前的例子。假定我们拥有如下的一个链表和一个新节点(C)。

             +---------+     +---------+
    head --->|         |---->|         |-----//----->
             |    A    |     |    B    |
     nil <---|         |<----|         |<----//------
             +---------+     +---------+ 
                 [0]             [1]
                  
                  
             +---------+ 
     new --->|         |----> nil
             |    C    |
             |         |
             +---------+
    
现在将新节点添加首节点之前，像这样：

    new.next = head
    head.previous = new
    
             +---------+            +---------+     +---------+
     new --->|         |--> head -->|         |---->|         |-----//----->
             |    C    |            |    A    |     |    B    |
             |         |<-----------|         |<----|         |<----//------
             +---------+            +---------+     +---------+ 


最后，让 head 指向新节点。

    head = new
    
             +---------+    +---------+     +---------+
    head --->|         |--->|         |---->|         |-----//----->
             |    C    |    |    A    |     |    B    |
     nil <---|         |<---|         |<----|         |<----//------
             +---------+    +---------+     +---------+ 
                 [0]            [1]             [2]


然而，当制定的索引大于 0 时，就必须获同时获取 privious 和 next 的索引，并在其间插入新节点。（译者：为什么必须获取两个呢？）

通过 `node(atIndex:)` 同样可以获取 privious 和 next 索引，如下：

             +---------+         +---------+     +---------+    
    head --->|         |---//--->|         |---->|         |----
             |         |         |    A    |     |    B    |    
     nil <---|         |---//<---|         |<----|         |<---
             +---------+         +---------+     +---------+    
                 [0]              [index-1]        [index]      
                                      ^               ^ 
                                      |               | 
                                     prev            next
    
    prev = node(at: index-1)
    next = prev.next

现在在 privious 和 next 之间插入新节点。

    new.prev = prev; prev.next = new  // connect prev and new.
    new.next = next; next.prev = new  // connect new and next.

             +---------+         +---------+     +---------+     +---------+
    head --->|         |---//--->|         |---->|         |---->|         |
             |         |         |    A    |     |    C    |     |    B    |
     nil <---|         |---//<---|         |<----|         |<----|         |
             +---------+         +---------+     +---------+     +---------+
                 [0]              [index-1]        [index]        [index+1]
                                      ^               ^               ^
                                      |               |               |
                                     prev            new             next


验证一下：

```swift
list.insert("Swift", atIndex: 1)
list[0]     // "Hello"
list[1]     // "Swift"
list[2]     // "World"
```

同样地，验证下分别在首尾添加新节点，已确定一切工作正常。

> **Note:** `node(atIndex:)` 和 `insert(_: atIndex:)` 方法同样适用于单向链表，因为它并不赖节点的 `previous` 指针来查找前一个元素。

我们还需要些什么呢？当然是移除节点！首先，我们实现 `removeAll()`方法，很简单：

```swift
  public func removeAll() {
    head = nil
  }
```

如果你设置了 tail 指针，此时你必须也将其置为 nil。

接下来，我们来添加一些方法，用于益处制定节点。如果你已经持有某个节点的引用，无疑直接 `remove()` 是最效率的，因为省去了遍历查找的过程。

```swift
  public func remove(node: Node) -> T {
    let prev = node.previous
    let next = node.next

    if let prev = prev {
      prev.next = next
    } else {
      head = next
    }
    next?.previous = prev

    node.previous = nil
    node.next = nil
    return node.value
  }
```

当从链中移除一个节点时，我们也破坏了对前后节点的引用。要恢复成完整的链，我们必须将上下节点重新链接起来。

别忘了 `head` 指针！如果（移除的）刚好时首节点，那么此时必须更新 `head` ，让它指向下一个节点。（反之亦然，如果你持有的是 `tail` 指针，同样需要让它指向尾节点。）。当然，如果没有节点了，那么 `head` 就应该置为 nil 。

验证一下：

```swift
list.remove(list.first!)   // "Hello"
list.count                     // 2
list[0]                        // "Swift"
list[1]                        // "World"
```

如果未持有一个节点，那么你可以使用 `removeLast()` 或者 `removeAt()`方法：

```swift
  public func removeLast() -> T {
    assert(!isEmpty)
    return remove(node: last!)
  }

  public func removeAt(_ index: Int) -> T {
    let node = nodeAt(index)
    assert(node != nil)
    return remove(node: node!)
  }
```

所有的移除方法都会返回被移除的元素。

```swift
list.removeLast()              // "World"
list.count                     // 1
list[0]                        // "Swift"

list.removeAt(0)          // "Swift"
list.count                     // 0
```

> **Note:** 对单向链表来说，移除尾部元素稍显复杂。不能简单的通过 `last` 来查找链的尾端，因为你还需要倒数第二个节点的引用，需要用 `nodesBeforeAndAfter()` 替代。如果链表有一个 tail 指针，那么 `removeLast()` 就非常便捷了，但是务必记得，移除后要让 `tail` 指向前一个节点。
> 
还可以为 `LinkedList` 类添加一些有趣的方法。比如让 debug 输出更具可能性。

```swift
extension LinkedList: CustomStringConvertible {
  public var description: String {
    var s = "["
    var node = head
    while node != nil {
      s += "\(node!.value)"
      node = node!.next
      if node != nil { s += ", " }
    }
    return s + "]"
  }
}
```

这会让输出变成一下格式：

	[Hello, Swift, World]

翻转链表，让它首位互换如何？下面是一种非常高效的反转算法：

```swift
  public func reverse() {
    var node = head
    tail = node // If you had a tail pointer
    while let currentNode = node {
      node = currentNode.next
      swap(&currentNode.next, &currentNode.previous)
      head = currentNode
    }
  }
```

循环遍历真个链并交换每个节点的 `next` 和 `previous` 指针。同时将 `head` 指向最后一个元素。（如果你设置了 tail 指针，此时需要同步更新）反转后的链表是这样的：

	         +--------+    +--------+    +--------+    +--------+
	tail --->|        |<---|        |<---|        |<---|        |---> nil
	         | node 0 |    | node 1 |    | node 2 |    | node 3 |
	 nil <---|        |--->|        |--->|        |--->|        |<--- head
	         +--------+    +--------+    +--------+    +--------+

数组拥有 `map()` 和 `filter()` 方法，没理由链表没有啊。

```swift
  public func map<U>(transform: T -> U) -> LinkedList<U> {
    let result = LinkedList<U>()
    var node = head
    while node != nil {
      result.append(transform(node!.value))
      node = node!.next
    }
    return result
  }
```

可以这样用：

```swift
let list = LinkedList<String>()
list.append("Hello")
list.append("Swifty")
list.append("Universe")

let m = list.map { s in s.characters.count }
m  // [5, 6, 8]
```

这是 filter（的实现）：

```swift
  public func filter(predicate: T -> Bool) -> LinkedList<T> {
    let result = LinkedList<T>()
    var node = head
    while node != nil {
      if predicate(node!.value) {
        result.append(node!.value)
      }
      node = node!.next
    }
    return result
  }
```

还有一个比较笨的例子：

```swift
let f = list.filter { s in s.count > 5 }
f    // [Universe, Swifty]
```

读者练习：这里实现的 `map()` and `filter()` 不够高效，因为他们都是通过 `append()` 方法向尾端添加新节点。而调用一次添加的时间复杂度是 **O(n)**，每次都要扫描整条链来查找最后一个节点。你能让它更高效一些吗？（提示：试着持有你刚刚添加的那个节点）。

## 替代方案（An alternative approach）
## An alternative approach

目前为止， `LinkedList` 是通过类引用来实现的，这没什么错，但是相对 Swift 中的其它集合诸如`Array` 和 `Dictionary` 而言，显得有些偏重。

其实它是可以以值的方式通过枚举来实现，大概这样：


```swift
enum ListNode<T> {
  indirect case node(T, next: ListNode<T>)
  case end
}
```

枚举实现（相对类引用实现）最大的不同点在于，针对链作出的任何改动都会产生一个 *new copy(新的拷贝)*，这是由 [Swift 值语义](https://developer.apple.com/swift/blog/?id=10)（自身特点）决定的，与你的意愿或者应用无关。

[作者：如果有必要，我会针对这个部分补充更多细节]
[译者：会随着作者的补充进行更新]
[I might fill out this section in more detail if there's a demand for it.]

## 遵循 Collection 协议（Conforming to the Collection protocol）
## Conforming to the Collection protocol

遵循 Sequence 协议的类型，它所包涵的元素可以被无损地多次遍历，可以通过下标访问，并实现了 Swift 标准库中的 Collection 协议。

处理数据集合时，通常需要进行数量庞大的属性访问和操作。不仅如此，对 Swift 开发者来说，它通常还允许自定义类型匹配

为了实现该协议，类需要实现以下内容：
  1 `startIndex` 和 `endIndex` 属性.
  2 Subscript access to elements as O(1). Diversions of this time complexity need to be documented.
In order to conform to this protocol, classes need to provide:
  
```swift
/// The position of the first element in a nonempty collection.
public var startIndex: Index {
  get {
    return LinkedListIndex<T>(node: head, tag: 0)
  }
}
  
/// The collection's "past the end" position---that is, the position one
/// greater than the last valid subscript argument.
/// - Complexity: O(n), where n is the number of elements in the list.
///   This diverts from the protocol's expectation.
public var endIndex: Index {
  get {
    if let h = self.head {
      return LinkedListIndex<T>(node: h, tag: count)
    } else {
      return LinkedListIndex<T>(node: nil, tag: startIndex.tag)
    }
  }
}
```

```swift
public subscript(position: Index) -> T {
  get {
    return position.node!.value
  }
}
```

由于集合需要自己管理索引，所以下面的实现保留了一个节点引用。同时一个tag属性用来指示节点在链中的所处位置。

```swift
/// Custom index type that contains a reference to the node at index 'tag'
public struct LinkedListIndex<T> : Comparable
{
  fileprivate let node: LinkedList<T>.LinkedListNode<T>?
  fileprivate let tag: Int

  public static func==<T>(lhs: LinkedListIndex<T>, rhs: LinkedListIndex<T>) -> Bool {
    return (lhs.tag == rhs.tag)
  }

  public static func< <T>(lhs: LinkedListIndex<T>, rhs: LinkedListIndex<T>) -> Bool {
    return (lhs.tag < rhs.tag)
  }
}
```

最终，随着下面的实现，链表已经可以通过给定索引计算索引了。

```swift
public func index(after idx: Index) -> Index {
  return LinkedListIndex<T>(node: idx.node?.next, tag: idx.tag+1)
}
```

## 时刻留意（Some things to keep in mind）
## Some things to keep in mind

链表灵活性高，但是操作时间杂度是 **O(n)** 。

当对链表执行操作时，务必妥善处理相关的 `next` 和 `previous` 指针，以及 `head` and `tail` 指针。一旦遗漏，链表就会出错，而工程也可能因此而崩溃，所以务必小心！

处理链表时，尽量使用递归：找到首元素，然后递归调用方法。直到没有下一个元素为止。这也是为什么链表能成为 LISP 这种函数式编程言的基础。

*原文由Matthijs Hollemans撰写，发表于Ray Wenderlich的 Swift 算法社区*
*译文由William Han翻译*
