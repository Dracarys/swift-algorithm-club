# 二叉搜索树（Binary Search Tree （BST））

> 相关教程发表在[这里](https://www.raywenderlich.com/139821/swift-algorithm-club-swift-binary-search-tree-data-structure)


二叉搜索树是一种特殊的[二叉树](../Binary%20Tree/)（每个节点最多拥有 2 个子节点），无论插入还是删除，它总是有序的。

要了更多关于树的知识，[请先阅读这篇](../Tree/)。

## 始终有序性

下面是一个二叉搜索树的示例:

![A binary search tree](Images/Tree1.png)

注意每个左子节点都比父节点小，每个右子节点都比父节点大。这是一个二叉搜索树的关键特性。

例如, `2` 小于 `7`, 所以它在左侧; `5` 大于 `2`, 所以它在右侧.

## 节点的插入

当执行插入操作时，首先将要插入的值与根节点比较。如果该值较小，就取左分支；如果该值较大，就取右分支。如此沿着树向下比较，直至找到一个空位，将新增值插入。

假设我们要插入数值 `9`:

- 从树的根开始 (该节点值为 `7`) 将其与 `9` 进行比较。
- `9 > 7`，所以取右分支，并继续与右子节点 `10` 进行比较。
- 由于 `9 < 10`, 取左分支.
- 现在已抵达无值可比的空位，所以将新节点 `9` 插入该位置。

下面是插入 `9` 以后的树：

![After adding 9](Images/Tree2.png)

树中有且仅有一个位置可以插入该值。查找这个位置同行很快。时间杂度是 **O(h)**，其中 **h** 代表树的高度。

> **注意：** 节点的**高度**是指从该节点到其最底层叶子节点的跳数。整个树的高度是指从根节点到最底层叶子节点到跳数。二叉搜索树的很多操作都通过树的高度来表示。

小值居左，大值居右，记住这个简单的规则，就可以让树始终保持有序，所以任何时候查询，都可以判断某个值是否包含在树中。

## 树的搜索

在树中搜索某个值，与插入操作步骤相同：

- 如果该值小于当前节点，取左分支。
- 如果该值大于当前节点，取右分支.
- 如果该值等于当前节点，找到了!

与大多数树操作类似，这也递归操作，直至找到或者遍历完所有节点。

下面是一个搜索 `5` 的例子:

![Searching the tree](Images/Searching.png)

通过树结构进行搜索非常快捷。耗时仅 **O(h)**。那怕你有一百万个节点，只要平衡性够好，也只需要 20 步便可找到。（思路非常类似数组中的[二分搜索算法（binary search）](../Binary%20Search)）

## 树的遍历

有时需要查看所有节点，而不是某一个。

遍历一个二叉树主要有三种方法:

1. *中序遍历（In-order）* (或者 *深度优先遍历（depth-first）*)：先查左子节点，在查父节点，最后查右子节点。
2. *前序遍历（Pre-order）*：先查父节点，在查左、右子节点。
3. *后序遍历（Post-order）*：先查左、右子节点，在查父节点。

树的遍历也是递归进行的。

如果以中序遍历一个二叉搜索树，那么所有节点就像是从小到大排列一样。例如下图，将依次打印 `1, 2, 5, 7, 9, 10`：

![Traversing the tree](Images/Traversing.png)

## 节点的移除

移除节点非常简单。移除后，需要用其左分支上最大的子节点或右分支上最小的子节点填充其位置。如此就能在移除后依然保持有序性。例如下图，移除 10 后需要用 9（图2） 或者 11（图3） 填充其位置。

![Deleting a node with two children](Images/DeleteTwoChildren.png)

注意，只有被移除的节点有子节点时才需要替换填充。否则直接断开其与父节点的连接即可：

![Deleting a leaf node](Images/DeleteLeaf.png)


## 上码 (方案 1)

理论差不多了，接下来我们看看怎么通过 Swift 实现一个二叉搜索树。有两种方案可以实现。首先，展示一个类（class）实现的版本，之后在展示如何通过枚举（enum）来实现。

下面是一个 `BinarySearchTree` 的类实现:

```swift
public class BinarySearchTree<T: Comparable> {
  private(set) public var value: T
  private(set) public var parent: BinarySearchTree?
  private(set) public var left: BinarySearchTree?
  private(set) public var right: BinarySearchTree?

  public init(value: T) {
    self.value = value
  }

  public var isRoot: Bool {
    return parent == nil
  }

  public var isLeaf: Bool {
    return left == nil && right == nil
  }

  public var isLeftChild: Bool {
    return parent?.left === self
  }

  public var isRightChild: Bool {
    return parent?.right === self
  }

  public var hasLeftChild: Bool {
    return left != nil
  }

  public var hasRightChild: Bool {
    return right != nil
  }

  public var hasAnyChild: Bool {
    return hasLeftChild || hasRightChild
  }

  public var hasBothChildren: Bool {
    return hasLeftChild && hasRightChild
  }

  public var count: Int {
    return (left?.count ?? 0) + 1 + (right?.count ?? 0)
  }
}
```

该类仅描述了一个节点而不是整个树。类型是泛型，所以节点可以存储任何类型的数据。此外它还有对其 `left` 、`right` 以及 `parent` 节点的引用。

用法如下:

```swift
let tree = BinarySearchTree<Int>(value: 7)
```

The `count` property determines how many nodes are in the subtree described by this node. This does not just count the node's immediate children but also their children and their children's children, and so on. If this particular object is the root node, then it counts how many nodes are in the entire tree. Initially, `count = 0`.

> **Note:** Because `left`, `right`, and `parent` are optional, we can make good use of Swift's optional chaining (`?`) and nil-coalescing operators (`??`). You could also write this sort of thing with `if let`, but that is less concise.

### Inserting nodes

A tree node by itself is useless, so here is how you would add new nodes to the tree:

```swift
  public func insert(value: T) {
    if value < self.value {
      if let left = left {
        left.insert(value: value)
      } else {
        left = BinarySearchTree(value: value)
        left?.parent = self
      }
    } else {
      if let right = right {
        right.insert(value: value)
      } else {
        right = BinarySearchTree(value: value)
        right?.parent = self
      }
    }
  }
```

Like so many other tree operations, insertion is easiest to implement with recursion. We compare the new value to the values of the existing nodes and decide whether to add it to the left branch or the right branch.

If there is no more left or right child to look at, we create a `BinarySearchTree` object for the new node and connect it to the tree by setting its `parent` property.

> **Note:** Because the whole point of a binary search tree is to have smaller nodes on the left and larger ones on the right, you should always insert elements at the root to make sure this remains a valid binary tree!

To build the complete tree from the example you can do:

```swift
let tree = BinarySearchTree<Int>(value: 7)
tree.insert(2)
tree.insert(5)
tree.insert(10)
tree.insert(9)
tree.insert(1)
```

> **Note:** For reasons that will become clear later, you should insert the numbers in a random order. If you insert them in a sorted order, the tree will not have the right shape.

For convenience, let's add an init method that calls `insert()` for all the elements in an array:

```swift
  public convenience init(array: [T]) {
    precondition(array.count > 0)
    self.init(value: array.first!)
    for v in array.dropFirst() {
      insert(value: v)
    }
  }
```

Now you can simply do this:

```swift
let tree = BinarySearchTree<Int>(array: [7, 2, 5, 10, 9, 1])
```

The first value in the array becomes the root of the tree.

### Debug output

When working with complicated data structures, it is useful to have human-readable debug output.

```swift
extension BinarySearchTree: CustomStringConvertible {
  public var description: String {
    var s = ""
    if let left = left {
      s += "(\(left.description)) <- "
    }
    s += "\(value)"
    if let right = right {
      s += " -> (\(right.description))"
    }
    return s
  }
}
```

When you do a `print(tree)`, you should get something like this:

	((1) <- 2 -> (5)) <- 7 -> ((9) <- 10)

The root node is in the middle. With some imagination, you should see that this indeed corresponds to the following tree:

![The tree](Images/Tree2.png)

You may be wondering what happens when you insert duplicate items? We always insert those in the right branch. Try it out!

### Searching

What do we do now that we have some values in our tree? Search for them, of course! To find items quickly is the main purpose of a binary search tree. :-)

Here is the implementation of `search()`:

```swift
  public func search(value: T) -> BinarySearchTree? {
    if value < self.value {
      return left?.search(value)
    } else if value > self.value {
      return right?.search(value)
    } else {
      return self  // found it!
    }
  }
```

I hope the logic is clear: this starts at the current node (usually the root) and compares the values. If the search value is less than the node's value, we continue searching in the left branch; if the search value is greater, we dive into the right branch.

If there are no more nodes to look at -- when `left` or `right` is nil -- then we return `nil` to indicate the search value is not in the tree.

> **Note:** In Swift, that is conveniently done with optional chaining; when you write `left?.search(value)` it automatically returns nil if `left` is nil. There is no need to explicitly check for this with an `if` statement.

Searching is a recursive process, but you can also implement it with a simple loop instead:

```swift
  public func search(_ value: T) -> BinarySearchTree? {
    var node: BinarySearchTree? = self
    while let n = node {
      if value < n.value {
        node = n.left
      } else if value > n.value {
        node = n.right
      } else {
        return node
      }
    }
    return nil
  }
```

Verify that you understand these two implementations are equivalent. Personally, I prefer to use iterative code over recursive code, but your opinion may differ. ;-)

Here is how to test searching:

```swift
tree.search(5)
tree.search(2)
tree.search(7)
tree.search(6)   // nil
```

The first three lines return the corresponding `BinaryTreeNode` object. The last line returns `nil` because there is no node with value `6`.

> **Note:** If there are duplicate items in the tree, `search()` returns the "highest" node. That makes sense, because we start searching from the root downwards.

### Traversal

Remember there are 3 different ways to look at all nodes in the tree? Here they are:

```swift
  public func traverseInOrder(process: (T) -> Void) {
    left?.traverseInOrder(process: process)
    process(value)
    right?.traverseInOrder(process: process)
  }

  public func traversePreOrder(process: (T) -> Void) {
    process(value)
    left?.traversePreOrder(process: process)
    right?.traversePreOrder(process: process)
  }

  public func traversePostOrder(process: (T) -> Void) {
    left?.traversePostOrder(process: process)
    right?.traversePostOrder(process: process)
    process(value)
  }
```

They all work the same but in different orders. Notice that all the work is done recursively. The Swift's optional chaining makes it clear that the calls to `traverseInOrder()` etc are ignored when there is no left or right child.

To print out all the values of the tree sorted from low to high you can write:

```swift
tree.traverseInOrder { value in print(value) }
```

This prints the following:

	1
	2
	5
	7
	9
	10

You can also add things like `map()` and `filter()` to the tree. For example, here is an implementation of map:

```swift

  public func map(formula: (T) -> T) -> [T] {
    var a = [T]()
    if let left = left { a += left.map(formula: formula) }
    a.append(formula(value))
    if let right = right { a += right.map(formula: formula) }
    return a
  }
```

This calls the `formula` closure on each node in the tree and collects the results in an array. `map()` works by traversing the tree in-order.

An extremely simple example of how to use `map()`:

```swift
  public func toArray() -> [T] {
    return map { $0 }
  }
```

This turns the contents of the tree back into a sorted array. Try it out in the playground:

```swift
tree.toArray()   // [1, 2, 5, 7, 9, 10]
```

As an exercise, see if you can implement filter and reduce.

### Deleting nodes

We can make the code more readable by defining some helper functions.

```swift
  private func reconnectParentTo(node: BinarySearchTree?) {
    if let parent = parent {
      if isLeftChild {
        parent.left = node
      } else {
        parent.right = node
      }
    }
    node?.parent = parent
  }
```

Making changes to the tree involves changing a bunch of `parent` and `left` and `right` pointers. This function helps with this implementation. It takes the parent of the current node -- that is `self` -- and connects it to another node which will be one of the children of `self`.

We also need a function that returns the minimum and maximum of a node:

```swift
  public func minimum() -> BinarySearchTree {
    var node = self
    while let next = node.left {
      node = next
    }
    return node
  }

  public func maximum() -> BinarySearchTree {
    var node = self
    while let next = node.right {
      node = next
    }
    return node
  }

```

The rest of the code is self-explanatory:

```swift
  @discardableResult public func remove() -> BinarySearchTree? {
    let replacement: BinarySearchTree?

    // Replacement for current node can be either biggest one on the left or
    // smallest one on the right, whichever is not nil
    if let right = right {
      replacement = right.minimum()
    } else if let left = left {
      replacement = left.maximum()
    } else {
      replacement = nil
    }

    replacement?.remove()

    // Place the replacement on current node's position
    replacement?.right = right
    replacement?.left = left
    right?.parent = replacement
    left?.parent = replacement
    reconnectParentTo(node:replacement)

    // The current node is no longer part of the tree, so clean it up.
    parent = nil
    left = nil
    right = nil

    return replacement
  }
```

### Depth and height

Recall that the height of a node is the distance to its lowest leaf. We can calculate that with the following function:

```swift
  public func height() -> Int {
    if isLeaf {
      return 0
    } else {
      return 1 + max(left?.height() ?? 0, right?.height() ?? 0)
    }
  }
```

We look at the heights of the left and right branches and take the highest one. Again, this is a recursive procedure. Since this looks at all children of this node, performance is **O(n)**.

> **Note:** Swift's null-coalescing operator is used as shorthand to deal with `left` or `right` pointers that are nil. You could write this with `if let`, but this is more concise.

Try it out:

```swift
tree.height()  // 2
```

You can also calculate the *depth* of a node, which is the distance to the root. Here is the code:

```swift
  public func depth() -> Int {
    var node = self
    var edges = 0
    while let parent = node.parent {
      node = parent
      edges += 1
    }
    return edges
  }
```

It steps upwards through the tree, following the `parent` pointers until we reach the root node (whose `parent` is nil). This takes **O(h)** time. Here is an example:

```swift
if let node9 = tree.search(9) {
  node9.depth()   // returns 2
}
```

### Predecessor and successor

The binary search tree is always "sorted" but that does not mean that consecutive numbers are actually next to each other in the tree.

![Example](Images/Tree2.png)

Note that you cannot find the number that comes before `7` by just looking at its left child node. The left child is `2`, not `5`. Likewise for the number that comes after `7`.

The `predecessor()` function returns the node whose value precedes the current value in sorted order:

```swift
  public func predecessor() -> BinarySearchTree<T>? {
    if let left = left {
      return left.maximum()
    } else {
      var node = self
      while let parent = node.parent {
        if parent.value < value { return parent }
        node = parent
      }
      return nil
    }
  }
```

It is easy if we have a left subtree. In that case, the immediate predecessor is the maximum value in that subtree. You can verify in the above picture that `5` is indeed the maximum value in `7`'s left branch.

If there is no left subtree, then we have to look at our parent nodes until we find a smaller value. If we want to know what the predecessor is of node `9`, we keep going up until we find the first parent with a smaller value, which is `7`.

The code for `successor()` works the same way but mirrored:

```swift
  public func successor() -> BinarySearchTree<T>? {
    if let right = right {
      return right.minimum()
    } else {
      var node = self
      while let parent = node.parent {
        if parent.value > value { return parent }
        node = parent
      }
      return nil
    }
  }
```

Both these methods run in **O(h)** time.

> **Note:** There is a variation called a ["threaded" binary tree](../Threaded%20Binary%20Tree) where "unused" left and right pointers are repurposed to make direct links between predecessor and successor nodes. Very clever!

### Is the search tree valid?

If you were intent on sabotage you could turn the binary search tree into an invalid tree by calling `insert()` on a node that is not the root. Here is an example:

```swift
if let node1 = tree.search(1) {
  node1.insert(100)
}
```

The value of the root node is `7`, so a node with value `100`must be in the tree's right branch. However, you are not inserting at the root but at a leaf node in the tree's left branch. So the new `100` node is in the wrong place in the tree!

As a result, doing `tree.search(100)` gives nil.

You can check whether a tree is a valid binary search tree with the following method:

```swift
  public func isBST(minValue minValue: T, maxValue: T) -> Bool {
    if value < minValue || value > maxValue { return false }
    let leftBST = left?.isBST(minValue: minValue, maxValue: value) ?? true
    let rightBST = right?.isBST(minValue: value, maxValue: maxValue) ?? true
    return leftBST && rightBST
  }
```

This verifies the left branch contains values that are less than the current node's value, and that the right branch only contains values that are larger.

Call it as follows:

```swift
if let node1 = tree.search(1) {
  tree.isBST(minValue: Int.min, maxValue: Int.max)  // true
  node1.insert(100)                                 // EVIL!!!
  tree.search(100)                                  // nil
  tree.isBST(minValue: Int.min, maxValue: Int.max)  // false
}
```

## The code (solution 2)

We have implemented the binary tree node as a class, but you can also use an enum.

The difference is reference semantics versus value semantics. Making a change to the class-based tree will update that same instance in memory, but the enum-based tree is immutable -- any insertions or deletions will give you an entirely new copy of the tree. Which one is best, totally depends on what you want to use it for.

Here is how you can make a binary search tree using an enum:

```swift
public enum BinarySearchTree<T: Comparable> {
  case Empty
  case Leaf(T)
  indirect case Node(BinarySearchTree, T, BinarySearchTree)
}
```

The enum has three cases:

- `Empty` to mark the end of a branch (the class-based version used `nil` references for this).
- `Leaf` for a leaf node that has no children.
- `Node` for a node that has one or two children. This is marked `indirect` so that it can hold `BinarySearchTree` values. Without `indirect` you can't make recursive enums.

> **Note:** The nodes in this binary tree do not have a reference to their parent node. It is not a major impediment, but it will make certain operations more cumbersome to implement.

This implementation is recursive, and each case of the enum will be treated differently. For example, this is how you can calculate the number of nodes in the tree and the height of the tree:

```swift
  public var count: Int {
    switch self {
    case .Empty: return 0
    case .Leaf: return 1
    case let .Node(left, _, right): return left.count + 1 + right.count
    }
  }

  public var height: Int {
    switch self {
    case .Empty: return -1
    case .Leaf: return 0
    case let .Node(left, _, right): return 1 + max(left.height, right.height)
    }
  }
```

Inserting new nodes looks like this:

```swift
  public func insert(newValue: T) -> BinarySearchTree {
    switch self {
    case .Empty:
      return .Leaf(newValue)

    case .Leaf(let value):
      if newValue < value {
        return .Node(.Leaf(newValue), value, .Empty)
      } else {
        return .Node(.Empty, value, .Leaf(newValue))
      }

    case .Node(let left, let value, let right):
      if newValue < value {
        return .Node(left.insert(newValue), value, right)
      } else {
        return .Node(left, value, right.insert(newValue))
      }
    }
  }
```

Try it out in a playground:

```swift
var tree = BinarySearchTree.Leaf(7)
tree = tree.insert(2)
tree = tree.insert(5)
tree = tree.insert(10)
tree = tree.insert(9)
tree = tree.insert(1)
```

Notice that for each insertion, you get back a new tree object, so you need to assign the result back to the `tree` variable.

Here is the all-important search function:

```swift
  public func search(x: T) -> BinarySearchTree? {
    switch self {
    case .Empty:
      return nil
    case .Leaf(let y):
      return (x == y) ? self : nil
    case let .Node(left, y, right):
      if x < y {
        return left.search(x)
      } else if y < x {
        return right.search(x)
      } else {
        return self
      }
    }
  }
```

Most of these functions have the same structure.

Try it out in a playground:

```swift
tree.search(10)
tree.search(1)
tree.search(11)   // nil
```

To print the tree for debug purposes, you can use this method:

```swift
extension BinarySearchTree: CustomDebugStringConvertible {
  public var debugDescription: String {
    switch self {
    case .Empty: return "."
    case .Leaf(let value): return "\(value)"
    case .Node(let left, let value, let right):
      return "(\(left.debugDescription) <- \(value) -> \(right.debugDescription))"
    }
  }
}
```

When you do `print(tree)`, it will look like this:

	((1 <- 2 -> 5) <- 7 -> (9 <- 10 -> .))

The root node is in the middle, and a dot means there is no child at that position.

## When the tree becomes unbalanced...

A binary search tree is *balanced* when its left and right subtrees contain the same number of nodes. In that case, the height of the tree is *log(n)*, where *n* is the number of nodes. That is the ideal situation.

If one branch is significantly longer than the other, searching becomes very slow. We end up checking more values than we need. In the worst case, the height of the tree can become *n*. Such a tree acts like a [linked list](../Linked%20List/) than a binary search tree, with performance degrading to **O(n)**. Not good!

One way to make the binary search tree balanced is to insert the nodes in a totally random order. On average that should balance out the tree well, but it not guaranteed, nor is it always practical.

The other solution is to use a *self-balancing* binary tree. This type of data structure adjusts the tree to keep it balanced after you insert or delete nodes. To see examples, check [AVL tree](../AVL%20Tree) and [red-black tree](../Red-Black%20Tree).

## See also

[Binary Search Tree on Wikipedia](https://en.wikipedia.org/wiki/Binary_search_tree)

*Written for the Swift Algorithm Club by [Nicolas Ameghino](http://www.github.com/nameghino) and Matthijs Hollemans*
