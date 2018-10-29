# 栈（Stack）

> 详细教程发表在[这里](https://www.raywenderlich.com/149213/swift-algorithm-club-swift-stack-data-structure)（译者：栈概念相对简单，所以仅简单翻译了这篇ReadMe，详细教程便不在翻译。原本该篇都不想翻译的，但原计划是翻译该仓库的所有算法文章，如果单独少一两篇会显得不完整，所以还是顺手补上。）

栈类似一个功能首先的数组。仅可以将新元素从栈的顶端 *push（压入）* ，或着从顶端 *pop（弹出）* ，如果不弹出元素，那么只能看到栈顶元素。

那为什么要这么做呢？这是因为，在众多算法问题中，经常会遇到将某些元素添加到一个临时的列表中，而后又将它们移除，且顺序不能乱的情况。

而栈恰好满足了这种后进先出（LIFO）的顺序需求。最后一个压入栈的元素也最先出被弹出。（另一个类似的数据结构是[队列(queue)](../Queue/)，它是先进先出）
例如，压入一个数：

```swift
stack.push(10)
```

现在栈是 `[ 10 ]`。压入下一个数：

```swift
stack.push(3)
```

此时栈是 `[ 10, 3 ]`。在压入一个数：

```swift
stack.push(57)
```

栈变成了 `[ 10, 3, 57 ]` 。现在我们来弹出最顶端的数：

```swift
stack.pop()
```

返回了  `57`，正是我们刚刚压入的数。现在栈又变成了`[ 10, 3 ]`。

```swift
stack.pop()
```

这一次返回 `3`，以此类推。如果栈空了，会返回 `nil` 或者在其它一些实现中返回错误消息 "栈下溢出(stack underflow)"。

通过 Swift 创建栈非常简单。对数组进行封装以满足 push，pop，以及读取栈顶元素等操作即可：

```swift
public struct Stack<T> {
  fileprivate var array = [T]()

  public var isEmpty: Bool {
    return array.isEmpty
  }

  public var count: Int {
    return array.count
  }

  public mutating func push(_ element: T) {
    array.append(element)
  }

  public mutating func pop() -> T? {
    return array.popLast()
  }

  public var top: T? {
    return array.last
  }
}
```

注意，push 是新元素添加到数组的末尾，而不是开头。在开头插入元素是一个非常昂贵的操作，时间复杂度为 **O(n)**，之所以如此，是因为需要将数组中已有的元素在内存中做整体移动。而尾部追加的时间杂度是  **O(1)**；它总是消耗固定的时间，与数组的大小无关。

有趣的栈：每次调用函数或方法，CPU都会将返回地址放入栈中。一旦抵达函数结尾，CPU 就会通过返回地址跳回到调用者。这也是为什么当你调用太多函数，例如：无限递归，会导致“栈溢出”，因为 CPU 的栈的空间已经耗尽了。

*由 Matthijs Hollemans 为 Swift 算法社区撰写*
*由 William Han 翻译*
