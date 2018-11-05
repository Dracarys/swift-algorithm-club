# 树（Trees）

> 相关教程发表在[这里](https://www.raywenderlich.com/138190/swift-algorithm-club-swift-tree-data-structure)

树可以呈现对象间的层级关系。如下图：

![A tree](Images/Tree.png)

树包涵一系列的节点，各个节点由于其它节点相连。

节点通常连接有子节点和父节点。子节点指当前节点的下一级节点；而父节点是指其连接的上一级节点。一个节点只能有一个父节点，但可以有多个子节点。

![A tree](Images/ParentChildren.png)

没有父节点的节点称为 *根（root）* 节点。没有子节点的节点称为 *叶子（leaf）* 节点。

树的关键在于没有闭环。下面展示的就不是树：

![Not a tree](Images/Cycles.png)

这样的结构被称为[图（graph）](../Graph/)。树实际上是一种非常简单的图（一脉相承的还有——[链表（linked list）](../Linked%20List/) 试想一下，它实际上是一个非常简单的树）

本文将讨论的是多用途树。既无子节点个数限制，也无节点顺序限制。

下面是一个简单的 Swift 实现：

```swift
public class TreeNode<T> {
  public var value: T

  public weak var parent: TreeNode?
  public var children = [TreeNode<T>]()

  public init(value: T) {
    self.value = value
  }

  public func addChild(_ node: TreeNode<T>) {
    children.append(node)
    node.parent = self
  }
}
```

它描述里树中的一个节点。有一个类型为 `T` 的值，一个 `parent` 节点引用，以及一个包涵了子节点的数组。

添加一个 `description` 方法，以便打印整个树：

```swift
extension TreeNode: CustomStringConvertible {
  public var description: String {
    var s = "\(value)"
    if !children.isEmpty {
      s += " {" + children.map { $0.description }.joined(separator: ", ") + "}"
    }
    return s
  }
}
```

在 playground 中尝试一下：

```swift
let tree = TreeNode<String>(value: "beverages")

let hotNode = TreeNode<String>(value: "hot")
let coldNode = TreeNode<String>(value: "cold")

let teaNode = TreeNode<String>(value: "tea")
let coffeeNode = TreeNode<String>(value: "coffee")
let chocolateNode = TreeNode<String>(value: "cocoa")

let blackTeaNode = TreeNode<String>(value: "black")
let greenTeaNode = TreeNode<String>(value: "green")
let chaiTeaNode = TreeNode<String>(value: "chai")

let sodaNode = TreeNode<String>(value: "soda")
let milkNode = TreeNode<String>(value: "milk")

let gingerAleNode = TreeNode<String>(value: "ginger ale")
let bitterLemonNode = TreeNode<String>(value: "bitter lemon")

tree.addChild(hotNode)
tree.addChild(coldNode)

hotNode.addChild(teaNode)
hotNode.addChild(coffeeNode)
hotNode.addChild(chocolateNode)

coldNode.addChild(sodaNode)
coldNode.addChild(milkNode)

teaNode.addChild(blackTeaNode)
teaNode.addChild(greenTeaNode)
teaNode.addChild(chaiTeaNode)

sodaNode.addChild(gingerAleNode)
sodaNode.addChild(bitterLemonNode)
```

打印 `tree` 中的值，结果如下：
If you print out the value of `tree`, you'll get:

	beverages {hot {tea {black, green, chai}, coffee, cocoa}, cold {soda {ginger ale, bitter lemon}, milk}}

恰好与下图相对应:

![Example tree](Images/Example.png)

由于没有父节点，所以 `beverages` 是根。而 `black`, `green`, `chai`, `coffee`, `cocoa`, `ginger ale`, `bitter lemon`, `milk` 由于没有子节点，所以他们都是叶子节点。

对于任何节点来说，都可以沿着它的 `parent` 属性访问到根：

```swift
teaNode.parent           // this is the "hot" node
teaNode.parent!.parent   // this is the root
```

在讨论树时，经常会用到一下术语:

- **树的高度** 指从根到最底层的叶子的连接数。在我们的例子中，树的高度是 3，只需 3 步即可从根跳转到底。

- **节点深度** 当前节点与根之间的连接数。例如, `tea` 的节点深度是 2，因为它只需 2 步即可调转至根。（根的节点深度是 0。）

有许多不同的方法去构建一棵树。例如，有时可以没有 `parent` 属性，或者每个节点最多只能有两个子节点———这种树称为[二叉树（binary tree）](../Binary%20Tree/)。树最常见的类型是[二叉搜索树（binary search tree）](../Binary%20Search%20Tree/) (或者 BST)，一种更严格的二叉树，节点有特定的排序，以提高搜索效率。

我们这里展示的多用途树主要是为了展示数据之间的层级关系，具体需要哪些额外功能最终还是取决于你的应用需求。

下面的例子将展示，如果通过 `TreeNode` 类查明一个树是否包涵某个特定值。首先查看节点自身的 `value` 是否匹配，随后查看所有的子节点。当然，这些子节点也都是 `TreeNodes`，从而可以继续：查看自己的值，然后在查看子节点的值。如此环环相扣，循环递归。

代码:

```swift
extension TreeNode where T: Equatable {
  func search(_ value: T) -> TreeNode? {
    if value == self.value {
      return self
    }
    for child in children {
      if let found = child.search(value) {
        return found
      }
    }
    return nil
  }
}
```

下面是应用的例子:

```swift
tree.search("cocoa")    // returns the "cocoa" node
tree.search("chai")     // returns the "chai" node
tree.search("bubbly")   // nil
```

仅使用数组也可以描述一棵树。在数组中保存相应的索引，之后在建立各节点之间的连接。例如，假设我们有如下对象：

	0 = beverage		5 = cocoa		9  = green
	1 = hot			6 = soda		10 = chai
	2 = cold		7 = milk		11 = ginger ale
	3 = tea			8 = black		12 = bitter lemon
	4 = coffee				

那么我们可以用下面的数组来保存这棵树:

	[ -1, 0, 0, 1, 1, 1, 2, 2, 3, 3, 3, 6, 6 ]

数组中的每个元素都指向它的父节点。如索引为 3 的 `tea`，因为它的父节点是 `hot`，所以它的值是 1。跟节点指向 `-1`，是因为它没有父节点。这种方式仅支持从某个节点向根查找，其它查找方式则不行。

如果有时你看到算法中提及  *森林（forest）*。也不要奇怪，这只是对一组独立树对象集合的称呼。例如，[联合查询(union-find)](../Union-Find/)

*由 Matthijs Hollemans 发表于 Swift 算法社区*

*由 William Han 翻译*
