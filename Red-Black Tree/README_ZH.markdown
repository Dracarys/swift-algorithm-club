# 红黑树（Red-Black Tree）

> **注意:** 虽然这个项目的翻译仅仅是为了帮助自己复习提高算法知识水平，但是本着严谨，尽量避免误导的态度，鉴于译者自己对红黑树的理解上还有欠缺，所以暂停翻译，待后序更新。
> 
红黑树（RBTs）是一个平衡版的 [二叉搜索树（Binary Search Tree）](../Binary%20Search%20Tree)，它可以保证那些基础操作（搜索，前驱，后继，最小，最大，插入和删除）的时间杂度即使最坏也只是对数。

二叉搜素树（BSTs）的缺点是在执行插入或删除操作后可能变的不在平衡。最坏时，会导致整棵树变成一个如下的链表：

```
a
  \
   b
    \
     c
      \
       d
```
为了阻止该问题的发生，红黑树会在插入或删除后及性能在平衡，并且在每个节点上存储一个额外的红或黑的颜色属性。每次操作后，红黑树均具备一下特征：

## 特征

1. 每个节点非红即黑。 
2. 根节点是黑色的。
3. 每个叶子节点是黑色的。
4. 如果一个节点是红色的，那么它的两个子节点都是黑色的。
5. 对任何一个节点而言，位于该节点与其叶子节点之间路径上的黑色节点数相同。

特征 5 恰好描述某个节点 x 的“黑高度”的定义，bh(x)，它表示位于该节点向下与其叶子节点之间路径上的黑色节点数，不包含该节点自身。来自[算法导论（CLRS）]
Property 5 includes the definition of the black-height of a node x, bh(x), which is the number of black nodes on a path from this node down to a leaf not counting the node itself.
From [CLRS]

## 方法

节点的：

* `nodeX.getSuccessor()` 返回 nodeX 的有序后继者
* `nodeX.minimum()` 返回 Nodex 子树中包含最小键的节点。
* `nodeX.maximum()` 返回 Nodex 子树中包含最大键的节点。

树的：

* `search(input:)` 返回指定键值的节点
* `minValue()` 返回整棵树中最小的键值
* `maxValue()` 返回整棵树中最大的键值
* `insert(key:)` 向树中插入键值
* `delete(key:)` 删除含有指定键值的节点
* `verify()` 校验给定的树是否符合红色树特征

旋转、插入和删除算法来自[算法导论（CLRS）]，均基于伪代码实现，

## 实现细节

For convenience, all nil-pointers to children or the parent (except the parent of the root) of a node are exchanged with a nullLeaf. This is an ordinary node object, like all other nodes in the tree, but with a black color, and in this case a nil value for its children, parent and key. Therefore, an empty tree consists of exactly one nullLeaf at the root.

## Rotation

Left rotation (around x):
Assumes that x.rightChild y is not a nullLeaf, rotates around the link from x to y, makes y the new root of the subtree with x as y's left child and y's left child as x's right child, where n = a node, [n] = a subtree
```
      |                |
      x                y
    /   \     ~>     /   \
  [A]    y          x    [C]
        / \        / \
      [B] [C]    [A] [B]
```
Right rotation (around y):
Assumes that y.leftChild x is not a nullLeaf, rotates around the link from y to x, makes x the new root of the subtree with y as x's right child and x's right child as y's left child, where n = a node, [n] = a subtree
```
      |                |
      x                y
    /   \     <~     /   \
  [A]    y          x    [C]
        / \        / \
      [B] [C]    [A] [B]
```
As rotation is a local operation only exchanging pointers it's runtime is O(1).

## Insertion

We create a node with the given key and set its color to red. Then we insert it into the the tree by performing a standard insert into a BST. After this, the tree might not be a valid RBT anymore, so we fix the red-black properties by calling the insertFixup algorithm.
The only violation of the red-black properties occurs at the inserted node z and its parent. Either both are red, or the parent does not exist (so there is a violation since the root must be black).
We have three distinct cases:
**Case 1:** The uncle of z is red. If z.parent is left child, z's uncle is z.grandparent's right child. If this is the case, we recolor and move z to z.grandparent, then we check again for this new z. In the following cases, we denote a red node with (x) and a black node with {x}, p = parent, gp = grandparent and u = uncle
```
         |                   |
        {gp}               (newZ)
       /    \     ~>       /    \
    (p)     (u)         {p}     {u}
    / \     / \         / \     / \
  (z)  [C] [D] [E]    (z) [C] [D] [E]
  / \                 / \
[A] [B]             [A] [B]

```
**Case 2a:** The uncle of z is black and z is right child. Here, we move z upwards, so z's parent is the newZ and then we rotate around this newZ. After this, we have Case 2b.
```
         |                   |
        {gp}                {gp}
       /    \     ~>       /    \
    (p)      {u}         (z)     {u}
    / \      / \         / \     / \
  [A]  (z)  [D] [E]  (newZ) [C] [D] [E]
       / \            / \
     [B] [C]        [A] [B]

```
**Case 2b:** The uncle of z is black and z is left child. In this case, we recolor z.parent to black and z.grandparent to red. Then we rotate around z's grandparent. Afterwards we have valid red-black tree.
```
         |                   |
        {gp}                {p}
       /    \     ~>       /    \
    (p)      {u}         (z)    (gp)
    / \      / \         / \     / \
  (z)  [C] [D] [E]    [A] [B]  [C]  {u}
  / \                               / \
[A] [B]                           [D] [E]

```
Running time of this algorithm: 
* Only case 1 may repeat, but this only h/2 steps, where h is the height of the tree
* Case 2a -> Case 2b -> red-black tree
* Case 2b -> red-black tree
As we perform case 1 at most O(log n) times and all other cases at most once, we have O(log n) recolorings and at most 2 rotations. 
The overall runtime of insert is O(log n).

## Deletion

We search for the node with the given key, and if it exists we delete it by performing a standard delete from a BST. If the deleted nodes' color was red everything is fine, otherwise red-black properties may be violated so we call the fixup procedure deleteFixup.
Violations can be that the parent and child of the deleted node x where red, so we now have two adjacent red nodes, or if we deleted the root, the root could now be red, or the black height property is violated.
We have 4 cases: We call deleteFixup on node x
**Case 1:** The sibling of x is red. The sibling is the other child of x's parent, which is not x itself. In this case, we recolor the parent of x and x.sibling then we left rotate around x's parent. In the following cases s = sibling of x, (x) = red, {x} = black, |x| = red/black.
```
        |                   |
       {p}                 {s}
      /    \     ~>       /    \
    {x}    (s)         (p)     [D]
    / \    / \         / \     
  [A] [B] [C] [D]    {x} [C]
                     / \  
                   [A] [B]

```
**Case 2:** The sibling of x is black and has two black children. Here, we recolor x.sibling to red, move x upwards to x.parent and check again for this newX.
```
        |                    |
       |p|                 |newX|
      /    \     ~>       /     \
    {x}     {s}          {x}     (s)
    / \    /   \          / \   /   \
  [A] [B] {l}  {r}     [A] [B] {l}  {r}
          / \  / \             / \  / \
        [C][D][E][F]         [C][D][E][F]

```
**Case 3:** The sibling of x is black with one black child to the right. In this case, we recolor the sibling to red and sibling.leftChild to black, then we right rotate around the sibling. After this we have case 4.
```
        |                    |
       |p|                  |p|
      /    \     ~>       /     \
    {x}     {s}          {x}     {l}
    / \    /   \         / \    /   \
  [A] [B] (l)  {r}     [A] [B] [C]  (s)
          / \  / \                  / \
        [C][D][E][F]               [D]{e}
                                      / \
                                    [E] [F]

```
**Case 4:** The sibling of x is black with one red child to the right. Here, we recolor the sibling to the color of x.parent and x.parent and sibling.rightChild to black. Then we left rotate around x.parent. After this operation we have a valid red-black tree. Here, ||x|| denotes that x can have either color red or black, but that this can be different to |x| color. This is important, as s gets the same color as p.
```
        |                        |
       ||p||                   ||s||
      /    \     ~>           /     \
    {x}     {s}              {p}     {r}
    / \    /   \            /  \     /  \
  [A] [B] |l|  (r)        {x}  |l|  [E] [F]
          / \  / \        / \  / \
        [C][D][E][F]    [A][B][C][D]

```
Running time of this algorithm:
* Only case 2 can repeat, but this only h many times, where h is the height of the tree
* Case 1 -> case 2 -> red-black tree
  Case 1 -> case 3 -> case 4 -> red-black tree
  Case 1 -> case 4 -> red-black tree
* Case 3 -> case 4 -> red-black tree
* Case 4 -> red-black tree
As we perform case 2 at most O(log n) times and all other steps at most once, we have O(log n) recolorings and at most 3 rotations.
The overall runtime of delete is O(log n).

## Resources:
[CLRS]  T. Cormen, C. Leiserson, R. Rivest, and C. Stein. "Introduction to Algorithms", Third Edition. 2009

*Written for Swift Algorithm Club by Ute Schiehlen. Updated from Jaap Wijnen and Ashwin Raghuraman's contributions.*
