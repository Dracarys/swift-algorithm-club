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

1. If the array is empty or contains a single element, there is no way to split it into smaller pieces. You must just return the array.

2. Find the middle index.

3. Using the middle index from the previous step, recursively split the left side of the array.

4. Also, recursively split the right side of the array.

5. Finally, merge all the values together, making sure that it is always sorted.

Here's the merging algorithm:

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

This method may look scary, but it is quite straightforward:

1. You need two indexes to keep track of your progress for the two arrays while merging.

2. This is the merged array. It is empty right now, but you will build it up in subsequent steps by appending elements from the other arrays.

3. This while-loop will compare the elements from the left and right sides and append them into the `orderedPile` while making sure that the result stays in order.

4. If control exits from the previous while-loop, it means that either the `leftPile` or the `rightPile` has its contents completely merged into the `orderedPile`. At this point, you no longer need to do comparisons. Just append the rest of the contents of the other array until there is no more to append.

As an example of how `merge()` works, suppose that we have the following piles: `leftPile = [1, 7, 8]` and `rightPile = [3, 6, 9]`. Note that each of these piles is individually sorted already -- that is always true with merge sort. These are merged into one larger sorted pile in the following steps:

	leftPile       rightPile       orderedPile
	[ 1, 7, 8 ]    [ 3, 6, 9 ]     [ ]
      l              r

The left index, here represented as `l`, points at the first item from the left pile, `1`. The right index, `r`, points at `3`. Therefore, the first item we add to `orderedPile` is `1`. We also move the left index `l` to the next item.

	leftPile       rightPile       orderedPile
	[ 1, 7, 8 ]    [ 3, 6, 9 ]     [ 1 ]
      -->l           r

Now `l` points at `7` but `r` is still at `3`. We add the smallest item to the ordered pile, so that is `3`. The situation is now:

	leftPile       rightPile       orderedPile
	[ 1, 7, 8 ]    [ 3, 6, 9 ]     [ 1, 3 ]
         l           -->r

This process repeats. At each step, we pick the smallest item from either the `leftPile` or the `rightPile` and add the item into the `orderedPile`:

	leftPile       rightPile       orderedPile
	[ 1, 7, 8 ]    [ 3, 6, 9 ]     [ 1, 3, 6 ]
         l              -->r

	leftPile       rightPile       orderedPile
	[ 1, 7, 8 ]    [ 3, 6, 9 ]     [ 1, 3, 6, 7 ]
         -->l              r

	leftPile       rightPile       orderedPile
	[ 1, 7, 8 ]    [ 3, 6, 9 ]     [ 1, 3, 6, 7, 8 ]
            -->l           r

Now, there are no more items in the left pile. We simply add the remaining items from the right pile, and we are done. The merged pile is `[ 1, 3, 6, 7, 8, 9 ]`.

Notice that, this algorithm is very simple: it moves from left-to-right through the two piles and at every step picks the smallest item. This works because we guarantee that each of the piles is already sorted.

## Bottom-up implementation

The implementation of the merge-sort algorithm you have seen so far is called the "top-down" approach because it first splits the array into smaller piles and then merges them. When sorting an array (as opposed to, say, a linked list) you can actually skip the splitting step and immediately start merging the individual array elements. This is called the "bottom-up" approach.

Time to step up the game a little. :-) Here is a complete bottom-up implementation in Swift:

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

It looks a lot more intimidating than the top-down version, but notice that the main body includes the same three `while` loops from `merge()`.

Notable points:

1. The Merge-sort algorithm needs a temporary working array because you cannot merge the left and right piles and at the same time overwrite their contents. Because allocating a new array for each merge is wasteful, we are using two working arrays, and we will switch between them using the value of `d`, which is either 0 or 1. The array `z[d]` is used for reading, and `z[1 - d]` is used for writing. This is called *double-buffering*.

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

## Performance

The speed of the merge-sort algorithm is dependent on the size of the array it needs to sort. The larger the array, the more work it needs to do.

Whether or not the initial array is sorted already does not affect the speed of the merge-sort algorithm since you will be doing the same amount splits and comparisons regardless of the initial order of the elements.

Therefore, the time complexity for the best, worst, and average case will always be **O(n log n)**.

A disadvantage of the merge-sort algorithm is that it needs a temporary "working" array equal in size to the array being sorted. It is not an **in-place** sort, unlike for example [quicksort](../Quicksort/).

Most implementations of the merge-sort algorithm produce a **stable** sort. This means that array elements that have identical sort keys will stay in the same order relative to each other after sorting. This is not important for simple values such as numbers or strings, but it can be an issue when sorting more complex objects.

## See also

[Merge sort on Wikipedia](https://en.wikipedia.org/wiki/Merge_sort)

*Written by Kelvin Lau. Additions by Matthijs Hollemans.*
