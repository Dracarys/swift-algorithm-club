# 堆（Heap）
> 译者声明：由于本人在翻译时对堆和树的知识有欠缺（这也正是翻译的意义，帮助自己查漏补缺），导致部分翻译很不准确，尤其是专用术语，甚至是翻译错误，所以暂时不建议阅读该译文。
> 
> 相关教程在[这里](https://www.raywenderlich.com/160631/swift-algorithm-club-heap-and-priority-queue-data-structure)
> This topic has been tutorialized [here](https://www.raywenderlich.com/160631/swift-algorithm-club-heap-and-priority-queue-data-structure)

堆实际上是一个存储了[二叉树（binary tree）](../Binary%20Tree/)的数组，且无需父/子（节点）指针。堆是基于堆属性存储的，堆属性决定了其子节点在树中的排序。（译者：看到这我也是糊涂的，但是别慌，继续即可）

A heap is a [binary tree](../Binary%20Tree/) inside an array, so it does not use parent/child pointers. A heap is sorted based on the "heap property" that determines the order of the nodes in the tree.

堆的常用场景：

- 建立 [优先级队列（priority queues）](../Priority%20Queue/)。
- 提供 [堆排序（heap sorts）](../Heap%20Sort/)。
- 快速查找集合中最小（或最大）元素。
- 让你不会编程的朋友感到震惊。
 
Common uses for heap:

- To build [priority queues](../Priority%20Queue/).
- To support [heap sorts](../Heap%20Sort/).
- To compute the minimum (or maximum) element of a collection quickly.
- To impress your non-programmer friends.

## 堆属性（The heap property）

存在两种堆：*大序堆* 和 *小序堆*，他们之间最大的区别就在于字节点在树中的排序不同。（译者：这里将 *max-heap* 和 *min-heap* 译为大序堆和小序堆，正是为了突出他们最大的区别——排序，也有译为大顶堆和小顶堆，最大堆和最小堆的）
There are two kinds of heaps: a *max-heap* and a *min-heap* which are different by the order in which they store the tree nodes.

在大序堆中，父节点的值要比其所有字节点的值都大。在小序堆中，每个父节点的值都要小于其字节点的值。这被称为“堆属性”，并且它适用于树中的每个节点。
In a max-heap, parent nodes have a greater value than each of their children. In a min-heap, every parent node has a smaller value than its child nodes. This is called the "heap property", and it is true for every single node in the tree.

例如：
An example:

![A max-heap](Images/Heap1.png)

这是一个大序堆，每个父节点都比其所有的子节点大。`(10)` 大于 `(7)` 和 `(2)`. `(7)` 大于 `(5)` 和 `(1)`.

This is a max-heap because every parent node is greater than its children. `(10)` is greater than `(7)` and `(2)`. `(7)` is greater than `(5)` and `(1)`.

受限于堆属性，大序堆总是将最大的对象作为树的根。反之，小序堆总是将最小的对象作为其树的根。正式因为堆有这样的属性，所以通常用它来实现[优先级队列（priority queue）](../Priority%20Queue/) ，以便更快捷的获取那个“最重要”的对象。

As a result of this heap property, a max-heap always stores its largest item at the root of the tree. For a min-heap, the root is always the smallest item in the tree. The heap property is useful because heaps are often used as a [priority queue](../Priority%20Queue/) to access the "most important" element quickly.

> **注意:** 堆的根不是最大就是最小值，但是后续的其它元素顺序是无法保证的。例如，大序堆 0 索引元素一定是最大的，但这并不意味着最后一个索引位元素就是最小的。只能保证最小值在其子叶中，但无法确定是哪一个。

> **Note:** The root of the heap has the maximum or minimum element, but the sort order of other elements are not predictable. For example, the maximum element is always at index 0 in a max-heap, but the minimum element isn’t necessarily the last one. -- the only guarantee you have is that it is one of the leaf nodes, but not which one.

## 堆与其它常见树有什么不同（How does a heap compare to regular trees）?

堆不能取代二叉搜索树，他们之间既有相似也有不同。下面是它们之间的主要区别：

A heap is not a replacement for a binary search tree, and there are similarities and differnces between them. Here are some main differences:

**节点顺序** 在[二叉搜索树（binary search tree——BST）](../Binary%20Search%20Tree/)中，左侧的子节点必须小于父节点，而右侧的子节点则必须大于其它节点。而堆则不是这样。大序堆中所有子节点都必须小于父节点，而小序堆又都必须大于父节点。

**Order of the nodes.** In a [binary search tree (BST)(BST)](../Binary%20Search%20Tree/), the left child must be smaller than its parent, and the right child must be greater. This is not true for a heap. In a max-heap both children must be smaller than the parent, while in a min-heap they both must be greater.

**内存占用** 传统树所耗内存比其实际存储数据要大一些。你需要开辟外的存储空间以存储节点对象和指向左右子节点的指针。堆则使用传统的数组作为存储结构，且不需要指针。

**Memory.** Traditional trees take up more memory than just the data they store. You need to allocate additional storage for the node objects and pointers to the left/right child nodes. A heap only uses a plain array for storage and uses no pointers.

**平衡性** 二叉搜索树必须是“平衡”的，所以大多数操作都具有 **O(log n)** 执行效率。它允许你随机的插入和删除数据，或者像操作[平衡树（AVL tree）](../AVL%20Tree/) 或 [红黑树（red-black tree）](../Red-Black%20Tree/)那样。（译者：这里翻译不顺，暂时保留原文）

**Balancing.** A binary search tree must be "balanced" so that most operations have **O(log n)** performance. You can either insert and delete your data in a random order or use something like an [AVL tree](../AVL%20Tree/) or [red-black tree](../Red-Black%20Tree/), but with heaps we don't actually need the entire tree to be sorted. We just want the heap property to be fulfilled, so balancing isn't an issue. Because of the way the heap is structured, heaps can guarantee **O(log n)** performance.

**查询特性** 二叉树对搜索很快，堆较慢。相对于堆前端的最大值（最小值），以及允许快速插入和删除等特性，查询的优先级则不那么高。
**Searching.** Whereas searching is fast in a binary tree, it is slow in a heap. Searching isn't a top priority in a heap since the purpose of a heap is to put the largest (or smallest) node at the front and to allow relatively fast inserts and deletes.

## 数组中的树（The tree inside an array）

用数组去实现一个树形结构，听上去是有点奇怪的，但是在耗时和所占空间上还是相当高效的。

An array may seem like an odd way to implement a tree-like structure, but it is efficient in both time and space.

This is how we are going to store the tree from the above example:

	[ 10, 7, 2, 5, 1 ]

这就是全部了，不需要比数组更多的存储空间。
That's all there is to it! We don't need any more storage than just this simple array.

既然没有额外指针，怎么确定哪个是子节点，那个父节点呢？好问题！（实际上）树中各节点在数组中的索引顺序已经定义好了其间的父子关系。

So how do we know which nodes are the parents and which are the children if we are not allowed to use any pointers? Good question! There is a well-defined relationship between the array index of a tree node and the array indices of its parent and children.

假定 `i` 是节点的索引，那么通过下面的共识就可以得出相应父节点和子节点的索引：
If `i` is the index of a node, then the following formulas give the array indices of its parent and child nodes:

    parent(i) = floor((i - 1)/2)
    left(i)   = 2i + 1
    right(i)  = 2i + 2

注意， `right(i)` 就是简单的 `left(i) + 1`。因为左右节点总是相邻存储在一起。
Note that `right(i)` is simply `left(i) + 1`. The left and right nodes are always stored right next to each other.

将数组的相应索引带入上面的公式，就可以得到父节点和子节点对应的位置：

| 节点 | 数组索引 (`i`) | 父索引 | 左子节点 | 右子节点 |
|------|-------------|--------------|------------|-------------|
| 10 | 0 | -1 | 1 | 2 |
| 7 | 1 | 0 | 3 | 4 |
| 2 | 2 | 0 | 5 | 6 |
| 5 | 3 | 1 | 7 | 8 |
| 1 | 4 | 1 | 9 | 10 |

Let's use these formulas on the example. Fill in the array index and we should get the positions of the parent and child nodes in the array:

| Node | Array index (`i`) | Parent index | Left child | Right child |
|------|-------------|--------------|------------|-------------|
| 10 | 0 | -1 | 1 | 2 |
| 7 | 1 | 0 | 3 | 4 |
| 2 | 2 | 0 | 5 | 6 |
| 5 | 3 | 1 | 7 | 8 |
| 1 | 4 | 1 | 9 | 10 |


检查一下，确认数组索引与图中所示的树是一一对应的。
Verify for yourself that these array indices indeed correspond to the picture of the tree.

> **Note:** 根节点 `(10)` 没有父节点，因为 `-1` 不是一个有效的数组索引。同理， 节点 `(2)`, `(5)` 和 `(1)` 也不存在子节点，因为它们的数组索引越界了，由此可见，在使用前，始终要先确认索引的是否有效。

> **Note:** The root node `(10)` does not have a parent because `-1` is not a valid array index. Likewise, nodes `(2)`, `(5)`, and `(1)` do not have children because those indices are greater than the array size, so we always have to make sure the indices we calculate are actually valid before we use them.

在大序堆中再运行一次，父节点的值必须使用大于（或等于）子节点的值。也就是说下面的（表达式）对数组中的所有索引 `i` 都必须为 true：

Recall that in a max-heap, the parent's value is always greater than (or equal to) the values of its children. This means the following must be true for all array indices `i`:

```swift
array[parent(i)] >= array[i]
```

确认一下，例子中给出的堆是否保持了堆属性
Verify that this heap property holds for the array from the example heap.

如你所见，该方程可以使我们不通过指针也能找到相应的父节点或子节点。它比指针（对方式）相对复杂一些，权衡利弊：增加了计算量，节约了内存空间。不过幸运的是，计算够快，而且时间复杂度仅是 **O(1)** 。
As you can see, these equations allow us to find the parent or child index for any node without the need for pointers. It is complicated than just dereferencing a pointer, but that is the tradeoff: we save memory space but pay with extra computations. Fortunately, the computations are fast and only take **O(1)** time.

理解数组中索引与树中节点位置的映射关系非常重要。（例如，）这里有一个包涵4层15个节点的树：

It is important to understand this relationship between array index and position in the tree. Here is a larger heap which has 15 nodes divided over four levels:

![Large heap](Images/LargeHeap.png)

图中的数字并不是节点的值，而是（节点）对应在数组中的索引。下面给出了数组索引与树不同层级之间的对应（关系）；
The numbers in this picture are not the values of the nodes but the array indices that store the nodes! Here is the array indices correspond to the different levels of the tree:

![The heap array](Images/Array.png)

要想公式正确，数组中的父节点必须位于子节点之前。可以上之前的图中印证（这一点）。
For the formulas to work, parent nodes must appear before child nodes in the array. You can see that in the above picture.

注意该方案的限定条件。下图就可以不选择堆而是用不同二叉树（代替）：
Note that this scheme has limitations. You can do the following with a regular binary tree but not with a heap:

![Impossible with a heap](Images/RegularTree.png)

在当前最低层级的节点占满之前，是不可以生成新层级的，所以堆的形状始终是下面这样：

You can not start a new level unless the current lowest level is completely full, so heaps always have this kind of shape:

![The shape of a heap](Images/HeapShape.png)

> **Note:** 你*可以*用堆的形式去遍历一个二叉树，只是这样不仅浪费空间，而且还需要把数组中空值标记出来。（译者：因为普通的二叉树的一层未必是满的）

> **Note:** You *could* emulate a regular binary tree with a heap, but it would be a waste of space, and you would need to mark array indices as being empty.

突击测验！假设我们有下面这样一个数组：
Pop quiz! Let's say we have the array:

	[ 10, 14, 25, 33, 81, 82, 99 ]

它是一个有效的堆吗？答案是对！一个从小到大的有序数组就是一个有效对小序堆。可以向下面这样把它画出来：

Is this a valid heap? The answer is yes! A sorted array from low-to-high is a valid min-heap. We can draw this heap as follows:

![A sorted array is a valid heap](Images/SortedArray.png)

由于父节点总是小于它对子节点，因此堆属性得以维系。(自己确认下，一个从大到小的的有序数组是不是一个有序的大序堆)
The heap property holds for each node because a parent is always smaller than its children. (Verify for yourself that an array sorted from high-to-low is always a valid max-heap.)

> **注意:** 不是所有的小序堆都是一个有序的数组！它是不逆的。要想把堆变为有序数组，需要用到[堆排（heap sort）](../Heap%20Sort/)
> **Note:** But not every min-heap is necessarily a sorted array! It only works one way. To turn a heap back into a sorted array, you need to use [heap sort](../Heap%20Sort/).

## 更多的数学（More math）!

如果你很好奇，那么这里还有一些公式可以更好的表现堆的属性（特征）。你不需要理解的很透彻，但它有时却真的可以派上用场。该小节可以跳过。
In case you are curious, here are a few more formulas that describe certain properties of a heap. You do not need to know these by heart, but they come in handy sometimes. Feel free to skip this section!

树的*高度*定义了，从根部到最底部的节点一共几步，或者更公式化一点说：高度就是节点间的最大行高数。*h* 高度的堆拥有 *h+1* 层。
The *height* of a tree is defined as the number of steps it takes to go from the root node to the lowest leaf node, or more formally: the height is the maximum number of edges between the nodes. A heap of height *h* has *h + 1* levels.

下面的堆高度是 3， 所以他有 4 层：
This heap has height 3, so it has 4 levels:

![Large heap](Images/LargeHeap.png)

一个包涵 *n* 个节点的堆，其高度是 *h = floor(log2(n))*。之所以如此，是因为在添加新层之前，总是先把上一层级的节点添满。例子中有15个节点，所以它的高度是 `floor(log2(15)) = floor(3.91) = 3`。

A heap with *n* nodes has height *h = floor(log2(n))*. This is because we always fill up the lowest level completely before we add a new level. The example has 15 nodes, so the height is `floor(log2(15)) = floor(3.91) = 3`.

如果最底层已经填满，那么这一层包涵 *2^h* 个节点。余下的树上层包涵 *2^h - 1* 个节点。带入例子中相应的数据：最低一层有 8 个节点，恰好是 `2^3 = 8`。前面的三层共包含 7 个节点，正好等于 `2^3 - 1 = 8 - 1 = 7`。
If the lowest level is completely full, then that level contains *2^h* nodes. The rest of the tree above it contains *2^h - 1* nodes. Fill in the numbers from the example: the lowest level has 8 nodes, which indeed is `2^3 = 8`. The first three levels contain a total of 7 nodes, i.e. `2^3 - 1 = 8 - 1 = 7`.

整个堆中节点总数是 *2^(h+1) - 1*。代入例子，`2^4 - 1 = 16 - 1 = 15`。
The total number of nodes *n* in the entire heap is therefore *2^(h+1) - 1*. In the example, `2^4 - 1 = 16 - 1 = 15`.

一个高度为 *h* 含有 *n* 个元素堆，最多可以拥有  *ceil(n/2^(h+1))* 个节点。（译者：这里没懂，有待更新）
There are at most *ceil(n/2^(h+1))* nodes of height *h* in an *n*-element heap.

叶子节点在数组中的索引范围总是 *floor(n/2)* 到 *n-1*。由此我们可以快速地通过数组来构建一个堆。如有疑问，可以通过例子来验证这一结论。;-)
The leaf nodes are always located at array indices *floor(n/2)* to *n-1*. We will make use of this fact to quickly build up the heap from an array. Verify this for the example if you don't believe it. ;-)

几个简单的数学知识照亮了你的一天。（译者：这句翻译的不够好。）
Just a few math facts to brighten your day.

## 可以用堆来做什么呢（What can you do with a heap）？

在插入和移除元素后，有两件基础的事必须要做，以保证堆是一个大序堆或者小序堆。

There are two primitive operations necessary to make sure the heap is a valid max-heap or min-heap after you insert or remove an element:

- `shiftUp()`: 如果某元素大于 (大序堆) 或小于 (小序堆) 它的父（节点）, 它就需要与父节点进行交换. 使它在整个树中上移。
- `shiftUp()`: If the element is greater (max-heap) or smaller (min-heap) than its parent, it needs to be swapped with the parent. This makes it move up the tree.

- `shiftDown()`. 如果某元素小于 (max-heap) 或大于 (min-heap) 它的父节点, 它就需要在整个树中下移。 这一操作也被称为 "堆化（heapify）".
- `shiftDown()`. If the element is smaller (max-heap) or greater (min-heap) than its children, it needs to move down the tree. This operation is also called "heapify".

上移或下移的递归时间复杂度都是 **O(log n)**。
Shifting up or down is a recursive procedure that takes **O(log n)** time.

下面是一些构建与基础操作之上的操作：
Here are other operations that are built on primitive operations:

- `insert(value)`: 在堆末端添加一个新元素，之后通过 `shiftUp()` 对堆进行修复.
- `insert(value)`: Adds the new element to the end of the heap and then uses `shiftUp()` to fix the heap.

- `remove()`: 移除并返回最大值(大序堆) or 最小值 (小序堆). 要填上由于移除元素而留下的空位，To fill up the hole left by removing the element, the very last element is moved to the root position and then `shiftDown()` fixes up the heap. (This is sometimes called "extract min" or "extract max".)
- `remove()`: Removes and returns the maximum value (max-heap) or the minimum value (min-heap). To fill up the hole left by removing the element, the very last element is moved to the root position and then `shiftDown()` fixes up the heap. (This is sometimes called "extract min" or "extract max".)

- `removeAtIndex(index)`: Just like `remove()` with the exception that it allows you to remove any item from the heap, not just the root. This calls both `shiftDown()`, in case the new element is out-of-order with its children, and `shiftUp()`, in case the element is out-of-order with its parents.
- `removeAtIndex(index)`: Just like `remove()` with the exception that it allows you to remove any item from the heap, not just the root. This calls both `shiftDown()`, in case the new element is out-of-order with its children, and `shiftUp()`, in case the element is out-of-order with its parents.

- `replace(index, value)`: Assigns a smaller (min-heap) or larger (max-heap) value to a node. Because this invalidates the heap property, it uses `shiftUp()` to patch things up. (Also called "decrease key" and "increase key".)
- `replace(index, value)`: Assigns a smaller (min-heap) or larger (max-heap) value to a node. Because this invalidates the heap property, it uses `shiftUp()` to patch things up. (Also called "decrease key" and "increase key".)

All of the above take time **O(log n)** because shifting up or down is expensive. There are also a few operations that take more time:

- `search(value)`. Heaps are not built for efficient searches, but the `replace()` and `removeAtIndex()` operations require the array index of the node, so you need to find that index. Time: **O(n)**.
- `search(value)`. Heaps are not built for efficient searches, but the `replace()` and `removeAtIndex()` operations require the array index of the node, so you need to find that index. Time: **O(n)**.

- `buildHeap(array)`: Converts an (unsorted) array into a heap by repeatedly calling `insert()`. If you are smart about this, it can be done in **O(n)** time.
- `buildHeap(array)`: Converts an (unsorted) array into a heap by repeatedly calling `insert()`. If you are smart about this, it can be done in **O(n)** time.

- [堆排（Heap sort）](../Heap%20Sort/). Since the heap is an array, we can use its unique properties to sort the array from low to high. Time: **O(n lg n).**
- [Heap sort](../Heap%20Sort/). Since the heap is an array, we can use its unique properties to sort the array from low to high. Time: **O(n lg n).**

The heap also has a `peek()` function that returns the maximum (max-heap) or minimum (min-heap) element, without removing it from the heap. Time: **O(1)**.

> **Note:** By far the most common things you will do with a heap are inserting new values with `insert()` and removing the maximum or minimum value with `remove()`. Both take **O(log n)** time. The other operations exist to support more advanced usage, such as building a priority queue where the "importance" of items can change after they have been added to the queue.

## Inserting into the heap

Let's go through an example of insertion to see in details how this works. We will insert the value `16` into this heap:

![The heap before insertion](Images/Heap1.png)

The array for this heap is `[ 10, 7, 2, 5, 1 ]`.

The first step when inserting a new item is to append it to the end of the array. The array becomes:

	[ 10, 7, 2, 5, 1, 16 ]

This corresponds to the following tree:

![The heap before insertion](Images/Insert1.png)

The `(16)` was added to the first available space on the last row.

Unfortunately, the heap property is no longer satisfied because `(2)` is above `(16)`, and we want higher numbers above lower numbers. (This is a max-heap.)

To restore the heap property, we swap `(16)` and `(2)`.

![The heap before insertion](Images/Insert2.png)

We are not done yet because `(10)` is also smaller than `(16)`. We keep swapping our inserted value with its parent, until the parent is larger or we reach the top of the tree. This is called **shift-up** or **sifting** and is done after every insertion. It makes a number that is too large or too small "float up" the tree.

Finally, we get:

![The heap before insertion](Images/Insert3.png)

And now every parent is greater than its children again.

The time required for shifting up is proportional to the height of the tree, so it takes **O(log n)** time. (The time it takes to append the node to the end of the array is only **O(1)**, so that does not slow it down.)

## Removing the root

Let's remove `(10)` from this tree:

![The heap before removal](Images/Heap1.png)

What happens to the empty spot at the top?

![The root is gone](Images/Remove1.png)

When inserting, we put the new value at the end of the array. Here, we do the opposite: we take the last object we have, stick it up on top of the tree, and restore the heap property.

![The last node goes to the root](Images/Remove2.png)

Let's look at how to **shift-down** `(1)`. To maintain the heap property for this max-heap, we want to the highest number of top. We have two candidates for swapping places with: `(7)` and `(2)`. We choose the highest number between these three nodes to be on top. That is `(7)`, so swapping `(1)` and `(7)` gives us the following tree.

![The last node goes to the root](Images/Remove3.png)

Keep shifting down until the node does not have any children or it is larger than both its children. For our heap, we only need one more swap to restore the heap property:

![The last node goes to the root](Images/Remove4.png)

The time required for shifting all the way down is proportional to the height of the tree which takes **O(log n)** time.

> **Note:** `shiftUp()` and `shiftDown()` can only fix one out-of-place element at a time. If there are multiple elements in the wrong place, you need to call these functions once for each of those elements.

## Removing any node

The vast majority of the time you will be removing the object at the root of the heap because that is what heaps are designed for.

However, it can be useful to remove an arbitrary element. This is a general version of `remove()` and may involve either `shiftDown()` or `shiftUp()`.

Let's take the example tree again and remove `(7)`:

![The heap before removal](Images/Heap1.png)

As a reminder, the array is:

	[ 10, 7, 2, 5, 1 ]

As you know, removing an element could potentially invalidate the max-heap or min-heap property. To fix this, we swap the node that we are removing with the last element:

	[ 10, 1, 2, 5, 7 ]

The last element is the one that we will return; we will call `removeLast()` to remove it from the heap. The `(1)` is now out-of-order because it is smaller than its child, `(5)` but sits higher in the tree. We call `shiftDown()` to repair this.

However, shifting down is not the only situation we need to handle. It may also happen that the new element must be shifted up. Consider what happens if you remove `(5)` from the following heap:

![We need to shift up](Images/Remove5.png)

Now `(5)` gets swapped with `(8)`. Because `(8)` is larger than its parent, we need to call `shiftUp()`.

## Creating a heap from an array

It can be convenient to convert an array into a heap. This just shuffles the array elements around until the heap property is satisfied.

In code it would look like this:

```swift
  private mutating func buildHeap(fromArray array: [T]) {
    for value in array {
      insert(value)
    }
  }
```

We simply call `insert()` for each of the values in the array. Simple enough but not very efficient. This takes **O(n log n)** time in total because there are **n** elements and each insertion takes **log n** time.

If you didn't gloss over the math section, you'd have seen that for any heap the elements at array indices *n/2* to *n-1* are the leaves of the tree. We can simply skip those leaves. We only have to process the other nodes, since they are parents with one or more children and therefore may be in the wrong order.

In code:

```swift
  private mutating func buildHeap(fromArray array: [T]) {
    elements = array
    for i in stride(from: (nodes.count/2-1), through: 0, by: -1) {
      shiftDown(index: i, heapSize: elements.count)
    }
  }
```

Here, `elements` is the heap's own array. We walk backwards through this array, starting at the first non-leaf node, and call `shiftDown()`. This simple loop puts these nodes, as well as the leaves that we skipped, in the correct order. This is known as Floyd's algorithm and only takes **O(n)** time. Win!

## Searching the heap

Heaps are not made for fast searches, but if you want to remove an arbitrary element using `removeAtIndex()` or change the value of an element with `replace()`, then you need to obtain the index of that element. Searching is one way to do this, but it is slow.

In a [binary search tree](../Binary%20Search%20Tree/), depending on the order of the nodes, a fast search can be guaranteed. Since a heap orders its nodes differently, a binary search will not work, and you need to check every node in the tree.

Let's take our example heap again:

![The heap](Images/Heap1.png)

If we want to search for the index of node `(1)`, we could just step through the array `[ 10, 7, 2, 5, 1 ]` with a linear search.

Even though the heap property was not conceived with searching in mind, we can still take advantage of it. We know that in a max-heap a parent node is always larger than its children, so we can ignore those children (and their children, and so on...) if the parent is already smaller than the value we are looking for.

Let's say we want to see if the heap contains the value `8` (it doesn't). We start at the root `(10)`. This is not what we are looking for, so we recursively look at its left and right child. The left child is `(7)`. That is also not what we want, but since this is a max-heap, we know there is no point in looking at the children of `(7)`. They will always be smaller than `7` and are therefore never equal to `8`; likewise, for the right child, `(2)`.

Despite this small optimization, searching is still an **O(n)** operation.

> **Note:** There is a way to turn lookups into a **O(1)** operation by keeping an additional dictionary that maps node values to indices. This may be worth doing if you often need to call `replace()` to change the "priority" of objects in a [priority queue](../Priority%20Queue/) that's built on a heap.

## The code

See [Heap.swift](Heap.swift) for the implementation of these concepts in Swift. Most of the code is quite straightforward. The only tricky bits are in `shiftUp()` and `shiftDown()`.

You have seen that there are two types of heaps: a max-heap and a min-heap. The only difference between them is in how they order their nodes: largest value first or smallest value first.

Rather than create two different versions, `MaxHeap` and `MinHeap`, there is just one `Heap` object and it takes an `isOrderedBefore` closure. This closure contains the logic that determines the order of two values. You have probably seen this before because it is also how Swift's `sort()` works.

To make a max-heap of integers, you write:

```swift
var maxHeap = Heap<Int>(sort: >)
```

And to create a min-heap you write:

```swift
var minHeap = Heap<Int>(sort: <)
```

I just wanted to point this out, because where most heap implementations use the `<` and `>` operators to compare values, this one uses the `isOrderedBefore()` closure.

## See also

[Heap on Wikipedia](https://en.wikipedia.org/wiki/Heap_%28data_structure%29)

*Written for the Swift Algorithm Club by [Kevin Randrup](http://www.github.com/kevinrandrup) and Matthijs Hollemans*
