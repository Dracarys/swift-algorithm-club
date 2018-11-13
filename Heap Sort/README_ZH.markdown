# 堆排（Heap Sort）

利用堆对一个数组进行从低到高的排序。

[heap](../Heap/)是一个存在数组内的部分有序的二叉树。堆排算法正是借助堆的结构特征进行快速排序的。

要进行从低到高的堆排，必须先将一个无序数组转化为一个大序堆，让最大元素位于数组首位。

假定我们有如下一个待排序的数组：

	[ 5, 13, 2, 25, 7, 17, 20, 8, 4 ]

首先将其转换为如下的大序堆：

![The max-heap](Images/MaxHeap.png)

此时堆内部数组如下:

	[ 25, 13, 20, 8, 7, 17, 2, 5, 4 ]

虽然这也叫排序！但是真正的排序过程才刚刚开始：将首元素(索引为 *0*)与末位索引为 *n-1* 的元素进行交换，如下：

	[ 4, 13, 20, 8, 7, 17, 2, 5, 25 ]
	  *                          *

新的根节点 `4` 会小于它的子节点，所以此时我们对前面的 *n-2* 个元素进行*降档*或“堆化”处理。堆修复完毕之后，新的根将是数组内第二大的元素。

	[20, 13, 17, 8, 7, 4, 2, 5 | 25]

重点：修复堆时，必须忽略位于 *n-1* 的末元素。因为它就是数组内的最大值，且已位于最终正确的位置上。`|` 竖线用来标识那些已经排好序的部分。之后我们将保留该部分不动。

再次交换首尾元素（这次末元素的索引是 *n-2*）：

	[5, 13, 17, 8, 7, 4, 2, 20 | 25]
	 *                      *

再次将其修复为大序堆：

	[17, 13, 5, 8, 7, 4, 2 | 20, 25]

可以看到最大的元素再次被排到了后面。如此重复这一过程，直至抵达根节点，此时整个数组就排好了。

> **注意：** 这个过程与[选择排序（selection sort）](../Selection%20Sort/)非常相似，后者是在余下的数组中不断地查找最小值。而取最小或最大值正式堆所擅长的。

堆排的时间杂度在最好、最坏已经平均都是 **O(n lg n)**，
Performance of heap sort is **O(n lg n)** in best, worst, and average case. Because we modify the array directly, heap sort can be performed in-place. But it is not a stable sort: the relative order of identical elements is not preserved.

下面是一个堆排的 Swift 实现：

```swift
extension Heap {
  public mutating func sort() -> [T] {
    for i in stride(from: (elements.count - 1), through: 1, by: -1) {
      swap(&elements[0], &elements[i])
      shiftDown(0, heapSize: i)
    }
    return elements
  }
}
```

在已有的[堆（heap）](../Heap/) 实现中添加了一个扩展方法 `sort()`。使用方法如下：

```swift
var h1 = Heap(array: [5, 13, 2, 25, 7, 17, 20, 8, 4], sort: >)
let a1 = h1.sort()
```

Because we need a max-heap to sort from low-to-high, you need to give `Heap` the reverse of the sort function. To sort `<`, the `Heap` object must be created with `>` as the sort function. In other words, sorting from low-to-high creates a max-heap and turns it into a min-heap.

We can write a handy helper function for that:

```swift
public func heapsort<T>(_ a: [T], _ sort: @escaping (T, T) -> Bool) -> [T] {
  let reverseOrder = { i1, i2 in sort(i2, i1) }
  var h = Heap(array: a, sort: reverseOrder)
  return h.sort()
}
```

*Written for Swift Algorithm Club by Matthijs Hollemans*
