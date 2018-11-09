# 平衡树（AVL Tree）

平衡树是一个具有自平衡能力的[二叉搜索树（binary search tree）](../Binary%20Search%20Tree/)，其左右子树的高度差距最大不超过 1。

当一个二叉树左右子树的节点数近乎相同时，我们就可以说这个二叉树是 *平衡的*。正是这一点使的树的搜索非常快。一旦平衡被打破，搜索的便随之恶化。

下面是一棵不平衡的树：

![Unbalanced tree](Images/Unbalanced.png)

所有的子节点都位于左分支，而右分支没有。实际上它几乎与[链表（linked list）](../Linked%20List/)无异。因此，搜索的时间杂度也随之由二叉搜索树的 **O(log n)** 将为（链表）  **O(n)**。

平衡的的树看上去应该是这样的：

![Balanced tree](Images/Balanced.png)

以完全随机的方式插入每个节点是可以保证树的平衡性的，但是这是没法保证的，也不是现实。

另一种解决方案就是使用 *自平衡* 的二叉树。这种数据结构会在插入和移除节点后对树进行调整，以保证其平衡性。此种树的高度始终保证是 *log(n)*，*n* 为节点数。在一个平衡的树上插入、移除、和搜索的操作时间杂度都是 **O(log n)**。非常高效。 ;-)

## 关于平衡树

平衡树可以通过向左或右旋转，来会恢复一棵树的平衡性。

位于平衡树中的节点，如果该节点的子树“高度”差不超过 1，那么就认为它是平衡的，而如果一棵树的所有子节点都是平衡的，那么整棵树就是平衡的。

节点的 *高度* 是指从该节点到其最底端的叶子节点的跳数。例如下面这棵树，从 A 到 E 需要 3 跳，所以 A 的高度是 3。B 的高度是 2，C 的高度是 1，其它节点因为都是叶子节点，所以高度是 0。

![Node height](Images/Height.png)

如前所述，如果一个平衡树中的节点的左右子树拥有相同的高度，那么它就是平衡的。不一定要绝对相等，但是差距不能超过 1。下面的例子中全部都是平衡树：

![Balanced trees](Images/BalanceOK.png)

但这里的就不是平衡树了，因为他们的左子树相比右子树高处太多：

![Unbalanced trees](Images/BalanceNotOK.png)

左右子树的高度差被称为 *平衡因子*。可以通过下面的公式求的：

	balance factor = abs(height(left subtree) - height(right subtree))

如果插入或移除后平衡因子大于 1，那么我们就需要对平衡树的这个部分（子树）进行旋转在平衡。

## 旋转

树的每个节点都拥有一个变量，以标识其当前的平衡因子。当插入一个新节点后，我们就需要对其父节点的平衡因子进行维护。一旦平衡因子大于 1，那么我们就对该部分进行“旋转”以恢复平衡。

![Rotation0](Images/RotationStep0.jpg)

旋转过程涉及到的术语（译者：因为配图用的是原配图，所以为了保持一致性，此处术语依然沿用英文）：

* *Root* - 待旋转子树的父节点；
* *Pivot* - 即将称为父节点的的节点（也就是说它在旋转后会取代 *Root* 的位置）
* *RotationSubtree* - 位于 *Pivot* 旋转一侧的子树
* *OppositeSubtree* - 位于 *Pivot* 旋转对策的子树（译者：这两个术语相对抽象，注意与下面的插图结合）

下面看一则通过 *右旋* （顺时针）恢复平衡的例子：

![Rotation1](Images/RotationStep1.jpg) ![Rotation2](Images/RotationStep2.jpg) ![Rotation3](Images/RotationStep3.jpg)

旋转大致分为一下三步：

1. 将 *RotationSubtree* 作为新的 *OppositeSubtree* 赋给 *Pivot*
2. 将 *Root* 作为新的 *RotationSubtree* 赋给 *Pivot*;
3. 最终检查

相应的伪代码如下：

```
Root.OS = Pivot.RS
Pivot.RS = Root
Root = Pivot
```

这一操作的时间是恒定的 **O(1)**，插入最多旋转 2 次。移除可能最坏则需要 **log(n)** 次旋转。

## 代码

[AVLTree.swift](AVLTree.swift) 的大部分代码都与普通[二叉搜索树（binary search tree）](../Binary%20Search%20Tree/) 相同。均可在二叉搜索树的实现中找到。例如，搜索就完全相同。唯一略有不同的就在于插入和移除节点。

> **注意：** 如果你对二叉搜索的常规操作还有点模糊，那么我建议再[复习一下这里](../Binary%20Search%20Tree/)。之后在看平衡树余下内容会更容易理解。

关键有趣的部分位于 `balance()` 方法中，它会在每次插入或移除节点后被调用。

## 参见

[维基百科-平衡树（英）](https://en.wikipedia.org/wiki/AVL_tree)

平衡树是最早出现的自平衡树。近来[红黑树（red-black tree）](../Red-Black%20Tree/)开始变的比较火。

*由 Mike Taghavi 和 Matthijs Hollemans 联合发表于 Swift 算法社区*

*由 William Han 翻译*