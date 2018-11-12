# 堆（Heap）
> 译者声明：由于本人在翻译时对堆和树的知识有欠缺（这也正是翻译的意义，帮助自己查漏补缺），导致部分翻译很不准确，尤其是专用术语，甚至是翻译错误，所以暂时不建议阅读该译文。
> 
> 相关教程在[这里](https://www.raywenderlich.com/160631/swift-algorithm-club-heap-and-priority-queue-data-structure)

堆实际上是一个内部元素以[二叉树（binary tree）](../Binary%20Tree/)形式存储的数组（译者：必须是完全二叉树），且无需父/子（节点）指针。堆是基于堆属性（heap property）存储的，堆属性决定了其子节点在树中的排序。（译者：堆属性或称为堆特征主要是指堆内元素的排序方式）

堆的常用场景：

- 构建 [优先级队列（priority queues）](../Priority%20Queue/)。
- 提供 [堆排序（heap sorts）](../Heap%20Sort/)。
- 快速查找集合中最小（或最大）元素。
- 装X。

## 堆属性（The heap property）

有两种堆：*大序堆* 和 *小序堆*，他们之间最大的区别就在于子节点在树中的排序不同。（译者：这里将 *max-heap* 和 *min-heap* 译为大序堆和小序堆，正是为了突出他们最大的区别——排序，也有译为大顶堆和小顶堆，最大堆和最小堆的）

在大序堆中，父节点的值要比其所有字节点的值都大。在小序堆中，每个父节点的值都要小于其子节点的值。这被称为“堆属性”，并且它适用于树中的每个节点。

例如：

![A max-heap](Images/Heap1.png)

这是一个大序堆，每个父节点都比其所有的子节点大。`(10)` 大于 `(7)` 和 `(2)`. `(7)` 大于 `(5)` 和 `(1)`。

受限于堆属性，大序堆总是将最大的对象作为树的根。反之，小序堆总是将最小的对象作为其树的根。正是因为堆有这样的属性，所以通常用它来实现[优先队列（priority queue）](../Priority%20Queue/) ，以便更快捷的获取那个“最重要”的对象。

> **注意:** 堆的根不是最大就是最小值，但是后续的其它元素顺序是无法保证的。例如，大序堆中索引为 0 的元素一定是最大的，但这并不意味着最后一个索引位元素就是最小的。只能保证最小值在其叶子节点中，但无法确定是哪一个。

## 堆与其它常见树有什么不同

堆不能取代二叉搜索树，他们之间既有相似也有不同。下面是它们之间的主要区别：

**节点顺序** 在[二叉搜索树（binary search tree——BST）](../Binary%20Search%20Tree/)中，左侧的子节点必须小于父节点，而右侧的子节点则必须大于父节点。而堆无此要求。大序堆中两个子节点都必须小于父节点，而小序堆则必须都大于父节点。

**内存占用** 传统树所耗内存比其实际存储数据要大一些。你需要开辟外的存储空间以存储节点对象和指向左右子节点的指针。堆则使用普通的数组作为存储结构，且不需要（额外的）指针。

**平衡性** 二叉搜索树必须是“平衡”的，这样大多数操作才具有 **O(log n)** 的时间杂度。因此你必须以完全随机的方式插入或删除数据，否则就要用[平衡树（AVL tree）](../AVL%20Tree/) 或 [红黑树（red-black tree）](../Red-Black%20Tree/)（以保持其平衡性）。而堆不需要整棵树都是有序的，只要堆属性完备即可，因此不存在平衡性的问题。堆 **O(log n)** 的时间杂度是有堆自身树觉结构决定的。

**查询** 二叉树的搜索很快，堆较慢。相对于在堆前端添加最大（或最小）值，以及允许快速插入和删除等特性，查询的优先级则不那么高。

## 数组中的树

用数组去实现一个树形结构，虽然听上去是有点奇怪，但是在时间和空间上都非常高效。

前面的例子如果用树来存储，形式如下：

	[ 10, 7, 2, 5, 1 ]


这就是全部了，完全不需要比数组更多的存储空间。

既然没有额外指针，怎么确定哪个是子节点，那个父节点呢？好问题！（实际上）树中各节点在数组中的索引顺序已经定义好了其间的父子关系。

假定 `i` 是某节点的索引，那么通过下面的公式就可以求得相应父节点和子节点的索引：

    parent(i) = floor((i - 1)/2)
    left(i)   = 2i + 1
    right(i)  = 2i + 2

注意， `right(i)` 仅仅是 `left(i) + 1`。因为左右节点总是相邻存储在一起。

将数组的相应索引带入上面的公式，就可以得到父节点和子节点对应的位置：

| 节点 | 数组索引 (`i`) | 父索引 | 左子节点 | 右子节点 |
|------|-------------|--------------|------------|-------------|
| 10 | 0 | -1 | 1 | 2 |
| 7 | 1 | 0 | 3 | 4 |
| 2 | 2 | 0 | 5 | 6 |
| 5 | 3 | 1 | 7 | 8 |
| 1 | 4 | 1 | 9 | 10 |

检查一下，确认数组索引与图中所示的树是不是一一对应的。

> **注意：** 根节点 `(10)` 没有父节点，因为 `-1` 不是一个有效的数组索引。同理， 节点 `(2)`, `(5)` 和 `(1)` 也不存在子节点，因为它们的数组索引越界了，由此可见，在使用前，务必要先确认索引的有效性。

回想一下大序堆，父节点的值必须使用大于（或等于）子节点的值。也就是说下面的（表达式）对数组中的所有索引 `i` 都必须为 true：

```swift
array[parent(i)] >= array[i]
```

确认一下，例子中给出的堆是否保持了该堆属性。

可见，该方程可以使我们不通过指针也能找到相应的父节点或子节点的索引。相比指针（的方式）复杂一些，但权衡利弊：虽增加了计算量，但也节约了内存空间。幸运地是，该计算方式够快，且时间复杂度为 **O(1)** 。

理解数组中索引与树中节点位置的映射关系非常重要。（例如）下面是一棵分为 4 层含有 15 个节点的树：

![Large heap](Images/LargeHeap.png)

图中的数字并不是节点的值，而是（节点）对应在数组中的索引。下面给出了数组索引与树不同层级之间的对应（关系）：

![The heap array](Images/Array.png)

要想公式有效，数组中的父节点必须位于子节点之前。可以上之前的图中印证（这一点）。

注意这种方式是有限制的。（例如）下图就必须用常规的二叉树，而不能用堆：

![Impossible with a heap](Images/RegularTree.png)

在当前最底层的节点被占满之前，是不可以生成新的一层的，所以堆的形状始终是下面这样：

![The shape of a heap](Images/HeapShape.png)

> **注意：** 你*可以*用堆的形式去遍历一个普通二叉树，只是这样不仅浪费空间，而且还需要把数组中空值标记出来。（译者：因为普通的二叉树的一层未必是满的）

突击测验！假设我们有下面这样一个数组：

	[ 10, 14, 25, 33, 81, 82, 99 ]

它是一个有效的堆吗？是的！一个从小到大的有序数组就是一个有效的小序堆。可以向下面这样把它画出来：

![A sorted array is a valid heap](Images/SortedArray.png)

由于父节点总是小于它对子节点，因此堆属性得以维系。(自己确认下，一个从大到小的的有序数组是不是一个有序的大序堆)

> **注意：** 不是所有的小序堆都是一个有序的数组！它是不可逆的。要想把堆变为有序数组，需要借助[堆排（heap sort）](../Heap%20Sort/)

## 更多的数学

如果你很好奇，那么这里还有一些公式可以更好的表现堆的属性（特征）。你不需要理解的很透彻，但它有时却真的可以派上用场。该小节可以跳过。
In case you are curious, here are a few more formulas that describe certain properties of a heap. You do not need to know these by heart, but they come in handy sometimes. Feel free to skip this section!

树的*高度*定义了从根部到最底部的叶子节点所需的跳数，或者更公式化一点说：高度就是节点间的最大边数。*h* 高度 *h* 的堆拥有 *h+1* 层。

下面的堆高度是 3， 所以他有 4 层：

![Large heap](Images/LargeHeap.png)

一个包涵 *n* 个节点的堆，其高度是 *h = floor(log2(n))*。之所以如此，是因为在添加新层之前，总是先把上一层级的节点添满。例子中有15个节点，所以它的高度是 `floor(log2(15)) = floor(3.91) = 3`。

如果最底层已经填满，那么这一层包涵 *2^h* 个节点。余下的树上层包涵 *2^h - 1* 个节点。带入例子中相应的数据：最低一层有 8 个节点，恰好是 `2^3 = 8`。前面的三层共包含 7 个节点，正好等于 `2^3 - 1 = 8 - 1 = 7`。

整个堆中节点总数是 *2^(h+1) - 1*。代入例子，`2^4 - 1 = 16 - 1 = 15`。

一个高度为 *h* 含有 *n* 个元素堆，最多可以拥有  *ceil(n/2^(h+1))* 个节点。（译者：这里没懂，有待更新）
There are at most *ceil(n/2^(h+1))* nodes of height *h* in an *n*-element heap.

叶子节点在数组中的索引范围总是 *floor(n/2)* 到 *n-1*。由此我们可以快速地通过数组来构建一个堆。如有疑问，可以通过例子来验证这一结论。;-)

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

- `removeAtIndex(index)`: 类似 `remove()`， 区别就在于，除了根元素，还允许你任意移除堆内其它元素。该（方法）同样会在新晋元素与其子元素顺序错误时或者新晋元素与其父元素顺序错误时调用到 `shiftDown()` 和 `shiftUp()` （方法）。
- `removeAtIndex(index)`: Just like `remove()` with the exception that it allows you to remove any item from the heap, not just the root. This calls both `shiftDown()`, in case the new element is out-of-order with its children, and `shiftUp()`, in case the element is out-of-order with its parents.

- `replace(index, value)`: 给一个 (小序堆)的节点赋相对更小的值或给一个 (大序堆) 的节点赋相对更大的值时， 由于该操作会破坏堆的排序特征, 所有会通过 `shiftUp()` 方法进行修正（译者：恢复堆的排序属性）. (也称为 "decrease key" 和 "increase key".)
- `replace(index, value)`: Assigns a smaller (min-heap) or larger (max-heap) value to a node. Because this invalidates the heap property, it uses `shiftUp()` to patch things up. (Also called "decrease key" and "increase key".)

以上所有操作的时间杂度都是 **O(log n)** 因为无论是升档 `shiftUp()` 还是降档 `shiftDown()` 都是昂贵的操作。此外还有一些操作更耗时：

All of the above take time **O(log n)** because shifting up or down is expensive. There are also a few operations that take more time:

- `search(value)`. 堆（结构）本身就不是为了高效查询而建立的, 但是因为 `replace()` 和 `removeAtIndex()` 操作需要查询目标节点的索引, 所以不得不如此。 时间杂度: **O(n)**。
- `search(value)`. Heaps are not built for efficient searches, but the `replace()` and `removeAtIndex()` operations require the array index of the node, so you need to find that index. Time: **O(n)**.

- `buildHeap(array)`: 通过反复调用 `insert()` 方法，将一个（无序）数组转化为堆。如果你做的够好，那么（该方法的）时间杂度应该是 **O(n)**。
- `buildHeap(array)`: Converts an (unsorted) array into a heap by repeatedly calling `insert()`. If you are smart about this, it can be done in **O(n)** time.

- [堆排（Heap sort）](../Heap%20Sort/). 鉴于堆是一个数组, 我们可以利用这点对其进行从低到高地排序。 时间杂度: **O(n lg n)**。
- [Heap sort](../Heap%20Sort/). Since the heap is an array, we can use its unique properties to sort the array from low to high. Time: **O(n lg n).**

堆还有一个  `peek()` 方法，可以返回最大（大序堆）或最小（小序堆）的元素，而无序从堆中移除。时间杂度：**O(1)**。
The heap also has a `peek()` function that returns the maximum (max-heap) or minimum (min-heap) element, without removing it from the heap. Time: **O(1)**.

> **注意:** 说了这多，但是常用的堆操作还是插入新元素 `insert()` 和移除最大或最小值的`remove()` 两个方法。它们的时间杂度都是 **O(log n)**。其它操作方法都是为了一些进阶应用，例如构建优先级队列时，可以对已添加的重要对象进行修改。

> **Note:** By far the most common things you will do with a heap are inserting new values with `insert()` and removing the maximum or minimum value with `remove()`. Both take **O(log n)** time. The other operations exist to support more advanced usage, such as building a priority queue where the "importance" of items can change after they have been added to the queue.

## 向堆中插入（Inserting into the heap）

下面我们通过一个插入的例子来看一下详细的过程。向堆中插入 `16` 这个值：
Let's go through an example of insertion to see in details how this works. We will insert the value `16` into this heap:

![The heap before insertion](Images/Heap1.png)

与堆对应的数组是 `[ 10, 7, 2, 5, 1 ]`.

首先将要插入的元素添加至数组末尾，此时数组变为：
The first step when inserting a new item is to append it to the end of the array. The array becomes:

	[ 10, 7, 2, 5, 1, 16 ]

对应的树如下：
This corresponds to the following tree:

![The heap before insertion](Images/Insert1.png)

`(16)` 被添加到了最下面一行的第一个有效位。
The `(16)` was added to the first available space on the last row.

不幸时的是，这样堆的属性就被破坏了，因为 `(2)` 排在了 `(16)` 上面，而正确的应该是是大数在小数上面。（这是大序堆）
Unfortunately, the heap property is no longer satisfied because `(2)` is above `(16)`, and we want higher numbers above lower numbers. (This is a max-heap.)

要恢复堆属性，我们需要将 `(16)` 和 `(2)` 互换。
To restore the heap property, we swap `(16)` and `(2)`.

![The heap before insertion](Images/Insert2.png)

还没完，因为 `(10)` 还是比 `(16)` 小。继续将新插入的元素与其父元素进行交换，直至父元素更大或者抵达数的顶端。该（操作）被称为 **shift-up** 或者 **sifting**，每次插入都会被执行。使得哪些比较大或比较小的数沿着树“上浮”。（译者：如果你习惯将树理解为倒置的树冠，这里顶和上浮，就需要理解为根和下沉）

We are not done yet because `(10)` is also smaller than `(16)`. We keep swapping our inserted value with its parent, until the parent is larger or we reach the top of the tree. This is called **shift-up** or **sifting** and is done after every insertion. It makes a number that is too large or too small "float up" the tree.

最终变成这样：
Finally, we get:

![The heap before insertion](Images/Insert3.png)

现在每个父节点都比其子节点要大了。（恢复了大序堆的堆属性）
And now every parent is greater than its children again.

The time required for shifting up is proportional to the height of the tree, so it takes **O(log n)** time. (The time it takes to append the node to the end of the array is only **O(1)**, so that does not slow it down.)

## 移除根（Removing the root）

从树中移除 `(10)`： 
Let's remove `(10)` from this tree:

![The heap before removal](Images/Heap1.png)

空出来的顶点会怎么样呢？
What happens to the empty spot at the top?

![The root is gone](Images/Remove1.png)

插入式，我们将新值添加到数组末尾。这次反着来：先获取到最后一个元素，将它放到树的顶端，然后在（逐步）恢复堆属性。
When inserting, we put the new value at the end of the array. Here, we do the opposite: we take the last object we have, stick it up on top of the tree, and restore the heap property.

![The last node goes to the root](Images/Remove2.png)

我们来看看怎么对 `(1)` 进行 **降档shift-down**。要会大序堆的堆属性，需要最大的值在顶端。有两个候选交换对象： `(7)` 和 `(2)`。这时选择三个节点中最大的。也就是 `(7)`（进行交换），交换`(1)` 和 `(7)` 后如下图

Let's look at how to **shift-down** `(1)`. To maintain the heap property for this max-heap, we want to the highest number of top. We have two candidates for swapping places with: `(7)` and `(2)`. We choose the highest number between these three nodes to be on top. That is `(7)`, so swapping `(1)` and `(7)` gives us the following tree.

![The last node goes to the root](Images/Remove3.png)

继续将挡，直到该节点没有子节点，或者它比其所有子节点都大。对我们这个堆来说，在进行一次交换即可恢复堆属性：
Keep shifting down until the node does not have any children or it is larger than both its children. For our heap, we only need one more swap to restore the heap property:

![The last node goes to the root](Images/Remove4.png)

升降操作的次数与树的高度是成比例的，其时间杂度是 **O(log n)**。
The time required for shifting all the way down is proportional to the height of the tree which takes **O(log n)** time.

> **Note:** `shiftUp()` and `shiftDown()` can only fix one out-of-place element at a time. If there are multiple elements in the wrong place, you need to call these functions once for each of those elements.

## 移除任意节点（Removing any node）

The vast majority of the time you will be removing the object at the root of the heap because that is what heaps are designed for.

However, it can be useful to remove an arbitrary element. This is a general version of `remove()` and may involve either `shiftDown()` or `shiftUp()`.

Let's take the example tree again and remove `(7)`:

![The heap before removal](Images/Heap1.png)

初始，数组是这样的：
As a reminder, the array is:

	[ 10, 7, 2, 5, 1 ]

如你所知，移除一个元素会潜在的破坏大序堆或小序堆属性。要修复这点，我们需要将最后一个元素与我们删除的元素做交换。
As you know, removing an element could potentially invalidate the max-heap or min-heap property. To fix this, we swap the node that we are removing with the last element:

	[ 10, 1, 2, 5, 7 ]

此时，最后一个元素正是我们需要返回的；调用 `removeLast()` 方法将其从堆中移除。此时 `(1)` 是不满足堆序的，因为它比它的子节点 `(5)` 小。通过 `shiftDown()` 来修正该问题。
The last element is the one that we will return; we will call `removeLast()` to remove it from the heap. The `(1)` is now out-of-order because it is smaller than its child, `(5)` but sits higher in the tree. We call `shiftDown()` to repair this.

然而，光降档可不够应对所有情况，还有可能需要升档。试想一下如果从下面的堆中移除 `(5)`。
However, shifting down is not the only situation we need to handle. It may also happen that the new element must be shifted up. Consider what happens if you remove `(5)` from the following heap:

![We need to shift up](Images/Remove5.png)

`(5)` 跟 `(8)` 交换。此时由于 `(8)` 比其父节点要大，所以需要进行 `shiftUp()`。
Now `(5)` gets swapped with `(8)`. Because `(8)` is larger than its parent, we need to call `shiftUp()`.

## 通过数组创建堆（Creating a heap from an array）

数组转堆很方便，只要将数组中的元素按堆属性进行排序即可。
It can be convenient to convert an array into a heap. This just shuffles the array elements around until the heap property is satisfied.

代码是如下：
In code it would look like this:

```swift
  private mutating func buildHeap(fromArray array: [T]) {
    for value in array {
      insert(value)
    }
  }
```

只是简单地对数组中的每个元素调用了 `insert()` 方法。简单是简单，但不够效率。由于要对 **n** 个元素逐个执行 **log n** 插入操作，所以它的时间杂度是 **O(n log n)**。
We simply call `insert()` for each of the values in the array. Simple enough but not very efficient. This takes **O(n log n)** time in total because there are **n** elements and each insertion takes **log n** time.

如果之前的数学光环还在，
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

## 堆搜索（Searching the heap）

堆就不是为了搜索而建立的，但是如果你想通过 `removeAtIndex()` 随机移除一个元素或者通过 replace()` 随机的修改一个元素，那么就需要先获取该元素的索引。查询就成了必然，虽然比较慢。
Heaps are not made for fast searches, but if you want to remove an arbitrary element using `removeAtIndex()` or change the value of an element with `replace()`, then you need to obtain the index of that element. Searching is one way to do this, but it is slow.

在[二叉搜索树（binary search tree）](../Binary%20Search%20Tree/)中，由于各节点的的顺序固定，所以快速检索是可以保证的。然而堆节点的顺序是不同的，所以二叉搜索算法（此时）行不通，我们必须遍历树中的每个节点。
In a [二叉搜索树（binary search tree）](../Binary%20Search%20Tree/), depending on the order of the nodes, a fast search can be guaranteed. Since a heap orders its nodes differently, a binary search will not work, and you need to check every node in the tree.

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
