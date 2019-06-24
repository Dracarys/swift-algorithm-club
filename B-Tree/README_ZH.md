# B-Tree

B-Tree 是一个自平衡的搜索树，在它的节点中可以存在两个以上的子节点。
A B-Tree is a self-balancing search tree, in which nodes can have more than two children.

### 特征 Properties

一个节点数为 *n* 的 B-Tree 满足一下特征：
A B-Tree of order *n* satisfies the following properties:
 - 每个节点最多有 *2n* 个键；Every node has at most *2n* keys.
 - 每个节点（跟节点除外）至少有 *n* 个键；Every node (except root) has at least *n* keys.
 - 每个非叶子节点都有 *k* 个键以及 *k+1* 个子节点；Every non-leaf node with *k* keys has *k+1* children.
 - 所有节点中的键都以生序排列；The keys in all nodes are sorted in increasing order. 
 - 位于一个非叶子节点的 *k* 和 *l* 两个键之间的子树，包含 *k* 到 *l* 之间到所有键；The subtree between two keys *k* and *l* of a non-leaf node contains all the keys between *k* and *l*.
 - 所有到的叶子节点都出现在同一层中；All leaves appear at the same level.

A second order B-Tree with keys from 1 to 20 looks like this:

![A B-Tree with 20 keys.](Images/BTree20.png)

### 代码表示 B-Tree 节点的代码示例The representation of a B-Tree node in code

```swift
class BTreeNode<Key: Comparable, Value> {
  unowned var owner: BTree<Key, Value>
  
  fileprivate var keys = [Key]()
  var children: [BTreeNode]?
  
  ...
}
```

代码的主要有 2 个数组构成：

- 一个用来存储键
- 一个用来存储子节点；

The main parts of a node are two arrays:

 - An array containing the keys
 - An array containing the children

![Node.](Images/Node.png)

此外节点还有一个指向其所属树的引用。这是必不可少的，因为节点必需知晓其在树中的位置。
Nodes also have a reference to the tree they belong to.  
This is necessary because nodes have to know the order of the tree.

*注意：包含子节点的数组是可选类型，因为叶子节点没有自节点。*

*Note: The array containing the children is an Optional, because leaf nodes don't have children.*

## 搜索 Searching

1. 搜索 `k` 键首先从根节点开始；Searching for a key `k` begins at the root node.
2. 对节点上的键执行先行搜索，直至找到不小于 `k` 的 `l` 键，或者数组结尾；We perform a linear search on the keys of the node, until we find a key `l` that is not less than `k`,  
   or reach the end of the array.
3. `k == l`，即已找到；If `k == l` then we have found the key.
4. 当 `k < l`: 
    - 如果当前节点不是叶子节点，那么转至 `l` 的左子节点，并重复 3 - 5 步；If the node we are on is not a leaf, then we go to the left child of `l`, and perform the steps 3 - 5 again.
    - 如果当前节点是叶子节点，那么 `k` 不在树上；If we are on a leaf, then `k` is not in the tree.
5. 当抵达数组末尾时：If we have reached the end of the array:
    - 如果当前节点不是叶子节点，那么转至最后一个子节点，并重复 3 - 5 步；If we are on a non-leaf node, then we go to the last child of the node, and perform the steps 3 - 5 again.
    - 如果是叶子节点，那么 `k` 不在树上。If we are on a leaf, then `k` is not in the tree.

### 代码说明 The code
`value(for:)` 方法会搜索指定的键，如果存在则返回其所关联的值，否则返回 `nil`。
`value(for:)` method searches for the given key and if it's in the tree,  
it returns the value associated with it, else it returns `nil`.

## 插入Insertion
键只能被插入到叶子节点
Keys can only be inserted to leaf nodes.

1. 首先对我们想要插入到键 `k` 执行搜索；Perform a search for the key `k` we want to insert.
2. 如果不存在，并且最终是位于叶子节点，那么可以进行插入操作；If we haven't found it and we are on a leaf node, we can insert it.
 - 如果搜索结束时所处的键 `l` 比 `k`大：If after the search the key `l` which we are standing on is greater than `k`:  
   那么我们将 `k` 插入到 `l` 之前。We insert `k` to the position before `l`.
 - 反之:  
   将 `k` 插入到 `l` 之后。We insert `k` to the position after `l`.

插入完毕后，需要检查该节点中的键数是否位于正确范围之内。
After insertion we should check if the number of keys in the node is in the correct range. 
如果键的数量超过及节点序列的两倍 `order*2` ，那么需要对节点进行拆分。
If there are more keys in the node than `order*2`, we need to split the node.

#### 拆分节点 Splitting a node

1. 将待拆分节点的中间的键上移到它的父节点中（假设该父节点只有一个键）。Move up the middle key of the node we want to split, to its parent (if it has one).  
2. 如果没有父节点（例如：根节点），那么创建一个新的根节点，并插入其中。同时将旧的根节点添加为新根节点的左子节点。If it hasn't got a parent(it is the root), then create a new root and insert to it.  
   Also add the old root as the left child of the new root.
3. Split the node into two by moving the keys (and children, if it's a non-leaf) that were after the middle key
   to a new node.  
4. Add the new node as a right child for the key that we have moved up.  

进行节点拆分之后，很可能导致父节点包含过多的键，所以需要继续进行拆分。最糟糕的情况是逐层拆分直至根节点（这种情况会使树的高度增加）。
After splitting a node it is possible that the parent node will also contain too many keys, so we need to split it also.  
In the worst case the splitting goes up to the root (in this case the height of the tree increases).

An insertion to a first order tree looks like this:

![Splitting](Images/InsertionSplit.png)

### 代码说明 The code
方法 `insert(_:for:)` 会执行插入操作。执行插入之后，会向上逐个检查每个子节点中包含的键是否超量。如果一个节点的键过量，那么它的父节点就会执行方法 `split(child:atIndex:)`。
The method `insert(_:for:)` does the insertion.
After it has inserted a key, as the recursion goes up every node checks the number of keys in its child.  
if a node has too many keys, its parent calls the `split(child:atIndex:)` method on it.

根节点由树自身进行检查。如果根节点在插入后超量，那么树会执行 `splitRoot()` 方法。
The root node is checked by the tree itself.  
If the root has too many nodes after the insertion the tree calls the `splitRoot()` method.

## 移除 Removal

键只能从叶子节点移除。
Keys can only be removed from leaf nodes.

1. 首先搜索我们要移除的键 `k`。Perform a search for the key `k` we want to remove.
2. 找到之后If we have found it:
   - 如果位于叶子节点If we are on a leaf node:
   	 那么移除它。  
     We can remove the key.
   - 否则: 
   	  我们用 `k` 的前驱 `p` 覆盖它，之后在从叶子节点将 `p` 移除。
     We overwrite `k` with its inorder predecessor `p`, then we remove `p` from the leaf node.

在移除一个键之后，需要对节点进行足量检查。
After a key have been removed from a node we should check that the node has enough keys.
如果节点的键过少，那么就需要给它添加，或者从其子妹中合并一个。
If a node has fewer keys than the order of the tree, then we should move a key to it,  
or merge it with one of its siblings.

#### 将键移动到子节点 Moving a key to the child
如果可疑节点的临近兄弟节点拥有超量的键，那么就应该在树上执行这一操作，否则就应该与它的一个兄弟节点进行合并。
If the problematic node has a nearest sibling that has more keys than the order of the tree,  
we should perform this operation on the tree, else we should merge the node with one of its siblings.

假设要修复的子节点 `c1` 在其父节数组中的索引是 `i`。
Let's say the child we want to fix `c1` is at index `i` in its parent node's children array.

如果位于 `i-1` 的子节点 `c2` 拥有过量的键：
If the child `c2` at index `i-1` has more keys than the order of the tree:  

1. 首先，我们将位于 `i-1` 的键从父节点移动到子节点 `c1` 的 `0` 索引位。We move the key at index `i-1` from the parent node to the child `c1`'s keys array at index `0`.
2. 其次，我们将位于 `c2` 末尾的键移动到付节点的 `i-1` 位。We move the last key from `c2` to the parent's keys array at index `i-1`.
3. (如果`c1`为非叶子节点)我们将最后一个子节点从 `c2` 移动到 `c1` 到首位（index 0）。(If `c1` is non-leaf) We move the last child from `c2` to `c1`'s children array at index 0.

否则：
Else:  

1. 我们将索引为 `i` 到键从父节点移动 `c1` 子节点对应数组的末尾；We move the key at index `i` from the parent node to the end of child `c1`'s keys array.
2. 其次，我们将 `c2` 首位的键移动到父节点的对应数组的索引 `i` 位。We move the first key from `c2` to the parent's keys array at index `i`.
3. (如果 `c1` 是非叶子节点)我们将 `c2` 的第一个子节点移动到 `c` 对应数组的末尾。(If `c1` isn't a leaf) We move the first child from `c2` to the end of `c1`'s children array. 

![Moving Key](Images/MovingKey.png)

#### 合并两个节点Merging two nodes
假设我们想合并的节点 `c1` 其在其父节点的子节点数组中的索引为 `i`
Let's say we want to merge the child `c1` at index `i` in its parent's children array.

如果 `c1` 左侧存在一个兄弟节点 `c2`：
If `c1` has a left sibling `c2`:

1. 首先将父节点索引为 `i-1` 的键，移动到 `c2` 节点键值数组的末尾。We move the key from the parent at index `i-1` to the end of `c2`'s keys array.
2. 其次将 `c1` 的键和子节点（如果为非叶子节点）移动到 `c2` 键和节点数组的末尾。We move the keys and the children(if it's a non-leaf) from `c1` to the end of `c2`'s keys and children array.
3. 最后，从父节点中移除索引为 `i-1` 的子节点。We remove the child at index `i-1` from the parent node.

如果 `c1` 仅右侧存在一个兄弟节点 `c2`：
Else if `c1` only has a right sibling `c2`:

1. 首先将父节点索引为 `i` 的键，移动到 `c2` 节点键值数组的首位。We move the key from the parent at index `i` to the beginning of `c2`'s keys array.
2. 其次将 `c1` 的键和子节点（如果为非叶子节点）移动到 `c2` 键和节点数组的首位。We move the keys and the children(if it's a non-leaf) from `c1` to the beginning of `c2`'s keys and children array.
3. 最后，从父节点中移除索引为 `i` 的子节点。We remove the child at index `i` from the parent node.

合并之后，很可能导致父节点包含的键不足，最坏的情况是会逐层合并到根节点，此时会导致树的高度降低。
After merging it is possible that now the parent node contains too few keys,  
so in the worst case merging also can go up to the root, in which case the height of the tree decreases.

![Merging Nodes](Images/MergingNodes.png)

### 代码说明 The code
- `remove(_:)` 从树中移除指定的键。删除某个键之后，每个节点都会检查它的子节点所包含的键的数量。如果某个子节点的出现不足，那么它会调用 `fix(childWithTooFewKeys:atIndex:)` 方法。
- `remove(_:)` method removes the given key from the tree. After a key has been deleted,  
  every node checks the number of keys in its child. If a child has less nodes than the order of the tree,
  it calls the `fix(childWithTooFewKeys:atIndex:)` method.  

- `fix(childWithTooFewKeys:atIndex:)` 方法会决定如何修复子节点（向它移动键还是将它合并）,确定后在调用 `move(keyAtIndex:to:from:at:)` or `merge(child:atIndex:to:)` 
- `fix(childWithTooFewKeys:atIndex:)` method decides which way to fix the child (by moving a key to it,
  or by merging it), then calls `move(keyAtIndex:to:from:at:)` or 
  `merge(child:atIndex:to:)` method according to its choice.

## 参见 See also

[Wikipedia](https://en.wikipedia.org/wiki/B-tree)  
[GeeksforGeeks](http://www.geeksforgeeks.org/b-tree-set-1-introduction-2/)

*由 Viktor Szilárd Simkó 为 Swift 算法社区撰写*

*由 William Han 翻译*
