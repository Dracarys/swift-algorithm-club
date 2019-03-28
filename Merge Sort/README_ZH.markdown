# Merge Sort

> 详细教程发布在[这里](https://www.raywenderlich.com/154256/swift-algorithm-club-swift-merge-sort)

目标：将一个数组从小到大进行排序（或者从大到小）

归并排序是由 John von Neumann 于 1945 年开创的，它是一种最优、最差以及平均的时间复杂度都是 **O(nlogⁿ)** 的高效排序算法。（译者：其实个人更推荐将其命名为“分治排序” 或者 “拆并排序”，因为这才比较符合中文的命名方式，突出该算法分而治之的思想。早起国内的计算机先驱受条件限制，翻译的不够中式，但是后期的，尤其是身处教育系统的工作者，绝对有责任对这些不准确的翻译进行纠正）

归并排序采用 **分而治之** 的策略，将一个大问题分解为多个小问题后在逐个击破。在我看来归并排序就是 **先拆** 和 **后合** 。

假设有一个包含 *n* 个数的数组需要排序，那么归并排序的操作步骤如下：

- 先将这些数放到一起。
- 然后拆成两堆儿，此时就有了**两堆无序**数字。
- 继续对每堆数字进行拆分，直至不能再分位置。最终得到 *n* 个仅包含一个数的堆。
- 最后开始一对一对地对分好的堆不断地进行 **合并** ，每一次合并都是有序的。由于每堆都是有序的，所以合并起来相对简单。

## 例子🌰

### 拆分

假设有这样一个无序数组 `[2, 1, 5, 4, 9]`。那么它就是可以看作是一堆数字。现在的目的就是将它拆散，拆到不能再拆为止。

首先，将数组拆成 2 个：`[2, 1]` 和 `[5, 4, 9]`。还能再拆吗？当然！

先看左边的数组 `[2, 1]` ，还可以拆成 `[2]` 和 `[1]`。这时还能再拆吗？不能了。那么看另外半边。

`[5, 4, 9]` 可以拆成 `[5]` 和 `[4, 9]` 。此时 `[5]` 已经不能再拆了，但是 `[4, 9]` 还可以，将其拆成 `[4]` 和 `[9]`。

至此拆分结束，最终每堆仅包含一个元素：`[2]` `[1]` `[5]` `[4]` `[9]`。

### 合并

数组已经拆分好了，现在该**边排序边合并**了。记住，（该算法）的思想是分而治之，因此每次合并，你都必须谨慎处理。（译者：分已经分了，是该治的时候了，小心解决好每个小问题，大问题自然迎刃而解。）

已拆分的序列是 `[2]` `[1]` `[5]` `[4]` `[9]`, 先将它们合并为 `[1, 2]`、`[4, 5]` 和 `[9]`。 由于 `[9]` 没有能与之合并的堆，所以这一轮合并先不考虑。 

紧接着将 `[1, 2]` 和 `[4, 5]` 合并为 `[1, 2, 4, 5]`，这一轮合并中，还是没有能与 `[9]` 合并的堆，所以依然不考虑。

最后，剩下 `[1, 2, 4, 5]` 和 `[9]` 两堆，此时可以合并 `[9]` 了， 最终合并后的数组为 `[1, 2, 4, 5, 9]`。

## 自顶向下的实现（Top-down implementation）

下面是一个 Swift 实现的归并排序:

```swift
func mergeSort(_ array: [Int]) -> [Int] {
  guard array.count > 1 else { return array }    // 1

  let middleIndex = array.count / 2              // 2

  let leftArray = mergeSort(Array(array[0..<middleIndex]))             // 3

  let rightArray = mergeSort(Array(array[middleIndex..<array.count]))  // 4

  return merge(leftPile: leftArray, rightPile: rightArray)             // 5
}
```

详解:

1. 如果数组为空或者仅包含一个元素，那么无序排序直接返回。

2. 求中间索引.

3. 通过前一步得到中间索引，对左半边数组进行递归拆分。

4. 同理，对右半边数组也进行递归拆分.

5. 最终，将所有拆分后的数字进行合并，并确保（合并）始终是有序的。

下面是合并的算法:

```swift
func merge(leftPile: [Int], rightPile: [Int]) -> [Int] {
  // 1
  var leftIndex = 0
  var rightIndex = 0

  // 2
  var orderedPile = [Int]()

  // 3
  while leftIndex < leftPile.count && rightIndex < rightPile.count {
    if leftPile[leftIndex] < rightPile[rightIndex] {
      orderedPile.append(leftPile[leftIndex])
      leftIndex += 1
    } else if leftPile[leftIndex] > rightPile[rightIndex] {
      orderedPile.append(rightPile[rightIndex])
      rightIndex += 1
    } else {
      orderedPile.append(leftPile[leftIndex])
      leftIndex += 1
      orderedPile.append(rightPile[rightIndex])
      rightIndex += 1
    }
  }

  // 4
  while leftIndex < leftPile.count {
    orderedPile.append(leftPile[leftIndex])
    leftIndex += 1
  }

  while rightIndex < rightPile.count {
    orderedPile.append(rightPile[rightIndex])
    rightIndex += 1
  }

  return orderedPile
}
```

该方法虽然看上去挺复杂，但其实相当简单直接:

1. 在合并两个个数组时，需要两个索引保持跟踪。You need two indexes to keep track of your progress for the two arrays while merging.

2. 这是合并后的数组。现在虽然是空的，但是之后的步骤中会将其它数组中的元素添加该数组中。This is the merged array. It is empty right now, but you will build it up in subsequent steps by appending elements from the other arrays.

3. 该 while 循环会比较左右两边的值，然后有序的将元素放到 `orderedPile` 中。This while-loop will compare the elements from the left and right sides and append them into the `orderedPile` while making sure that the result stays in order.

4. 一旦前面 While 循环结束，那么就意味着 `leftPile` 或 `rightPile`，它们中的一个已经将元素全部合并到 `orderedPile` 中。此时无需在进行比较，直接将另一个数组中剩余的元素直接添加进去即可。If control exits from the previous while-loop, it means that either the `leftPile` or the `rightPile` has its contents completely merged into the `orderedPile`. At this point, you no longer need to do comparisons. Just append the rest of the contents of the other array until there is no more to append.

下面通过一个例子来演示 `merge()` 的工作原理, 假定我们拥有一下两堆数字: `leftPile = [1, 7, 8]` 和 `rightPile = [3, 6, 9]`。注意，每一堆数字都是单独排好序堆——这是归并排序的基础。通过下面的方式将他们合并为一个更大的堆：

	leftPile       rightPile       orderedPile
	[ 1, 7, 8 ]    [ 3, 6, 9 ]     [ ]
      l              r

左侧索引 `l`, 指向左边一堆数字的第一个元素，`1`。 右侧索引, `r`, 指向 `3`。通过比较发现第一个要添加到 `orderedPile` 的元素是 `1`。 与此同时将索引 `l` 指向下一个元素。

	leftPile       rightPile       orderedPile
	[ 1, 7, 8 ]    [ 3, 6, 9 ]     [ 1 ]
      -->l           r

现在 `l` 指向 `7` 但是 `r` 还是指向 `3`. 将较小的元素 `2` 添加到 `orderedPile`:

	leftPile       rightPile       orderedPile
	[ 1, 7, 8 ]    [ 3, 6, 9 ]     [ 1, 3 ]
         l           -->r

重复这一过程。 每次从 `leftPile` 或 `rightPile` 中取一个最小的元素，并把它添加到 `orderedPile`:

	leftPile       rightPile       orderedPile
	[ 1, 7, 8 ]    [ 3, 6, 9 ]     [ 1, 3, 6 ]
         l              -->r

	leftPile       rightPile       orderedPile
	[ 1, 7, 8 ]    [ 3, 6, 9 ]     [ 1, 3, 6, 7 ]
         -->l              r

	leftPile       rightPile       orderedPile
	[ 1, 7, 8 ]    [ 3, 6, 9 ]     [ 1, 3, 6, 7, 8 ]
            -->l           r

现在，左边已经没有未被添加到元素了，那么我们直接把右边剩余的元素全部添加到 `orderedPile `，合并后的数组是 `[ 1, 3, 6, 7, 8, 9 ]`。

看到了吧，该算法非常简单：从左至右的遍历两个数组，每次都得到最小的元素。之所以如此，是因为我们给定的两个数组都是已经排好序的。

## 自底向上到实现（Bottom-up implementation）
前面实现的归并排序算法，由于采用先分成小数组，之后在逐步合并的方式，因此称之为“自顶向下”的排序。

The implementation of the merge-sort algorithm you have seen so far is called the "top-down" approach because it first splits the array into smaller piles and then merges them. When sorting an array (as opposed to, say, a linked list) you can actually skip the splitting step and immediately start merging the individual array elements. This is called the "bottom-up" approach.

下面是一个“自底向上”的 Swift 实现:

```swift
func mergeSortBottomUp<T>(_ a: [T], _ isOrderedBefore: (T, T) -> Bool) -> [T] {
  let n = a.count

  var z = [a, a]      // 1
  var d = 0

  var width = 1
  while width < n {   // 2

    var i = 0
    while i < n {     // 3

      var j = i
      var l = i
      var r = i + width

      let lmax = min(l + width, n)
      let rmax = min(r + width, n)

      while l < lmax && r < rmax {                // 4
        if isOrderedBefore(z[d][l], z[d][r]) {
          z[1 - d][j] = z[d][l]
          l += 1
        } else {
          z[1 - d][j] = z[d][r]
          r += 1
        }
        j += 1
      }
      while l < lmax {
        z[1 - d][j] = z[d][l]
        j += 1
        l += 1
      }
      while r < rmax {
        z[1 - d][j] = z[d][r]
        j += 1
        r += 1
      }

      i += width*2
    }

    width *= 2
    d = 1 - d      // 5
  }
  return z[d]
}
```

看上去比“自顶向下”的版本貌似更复杂，但是注意到了吗，其中包含了3个与 `merge()` 相同的 `while` 循环。 
It looks a lot more intimidating than the top-down version, but notice that the main body includes the same three `while` loops from `merge()`.

有几点需要注意⚠️:

归并排序需要一个临时数组，因为你不能在合并左右两个子数组的过程中同时修改它们对内容。因为每次合并都创建一个新数组很费时，所以我们使用两个两个临时数组，并且通过一个表示 0 或 1 的 `d` 的值来对它们进行交换。数组 `z[d]` 用于读， `z[1 - d]`用于写，这种方式被称为 *双缓冲*
1. The Merge-sort algorithm needs a temporary working array because you cannot merge the left and right piles and at the same time overwrite their contents. Because allocating a new array for each merge is wasteful, we are using two working arrays, and we will switch between them using the value of `d`, which is either 0 or 1. The array `z[d]` is used for reading, and `z[1 - d]` is used for writing. This is called *double-buffering*.

从概念上来说，自底向上与自顶下的运行模式是相同的。先合并所有单个元素，再合并所有两个一组的元素，之后是4个，依此类推。假设堆大小为 `width`，那么初始 `width` 是 `1`，每次循环结束，都将其扩大一倍。所以外层循环
2. Conceptually, the bottom-up version works the same way as the top-down version. First, it merges small piles of one element each, then it merges piles of two elements each, then piles of four elements each, and so on. The size of the pile is given by `width`. Initially, `width` is `1` but at the end of each loop iteration, we multiply it by two, so this outer loop determines the size of the piles being merged, and the subarrays to merge become larger in each step.

3. The inner loop steps through the piles and merges each pair of piles into a larger one. The result is written in the array given by `z[1 - d]`.

4. This is the same logic as in the top-down version. The main difference is that we're using double-buffering, so values are read from `z[d]` and written into `z[1 - d]`. It also uses an `isOrderedBefore` function to compare the elements rather than just `<`, so this merge-sort algorithm is generic, and you can use it to sort any kind of object you want.

5. At this point, the piles of size `width` from array `z[d]` have been merged into larger piles of size `width * 2` in array `z[1 - d]`. Here, we swap the active array, so that in the next step we'll read from the new piles we have just created.

This function is generic, so you can use it to sort any type you desire, as long as you provide a proper `isOrderedBefore` closure to compare the elements.

Example of how to use it:

```swift
let array = [2, 1, 5, 4, 9]
mergeSortBottomUp(array, <)   // [1, 2, 4, 5, 9]
```

## 效率

归并排序的运行效率取决于待排序数组的大小。数组越大，排序的工作量也就越大。

初始数组无论是否已经排好序，都会不会影响到归并排序的运行效率，因为始终要会对数组进行拆分，之后在合并的操作，与初始元素顺序无关。

正因为此，其时间复杂度在最好、最坏以及平均都始终是 **O(nlogⁿ)**。

归并排序的缺点是，（排序过程中）需要一个临时的与待排序数组容量相同的数组。因此与[快速排序](../Quicksort/)不同，它不是一个**原位**排序算法。

大多数归并排序的实现都是**稳定的**。也就是说数组中元素的相对位置即使经过多次排序，也始终是相同的。这对于简单数值或字符串排序来说没什么意义，但是对复杂对象进行排序时就很重要了。

## 参见

[Merge sort on Wikipedia](https://en.wikipedia.org/wiki/Merge_sort)



*由 Kelvin Lau 和 Matthijs Hollemans 联合发表于 Swift 算法社区*

*由 William Han 翻译*
