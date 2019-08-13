# 快速排序

目标：从低到高（活着高到低）对数组进行排序。

快速排序（Quicksort）是历史上最著名的算法之一。它由 Tony Hoare 于 1959年发明，那是一个对递归概念尚且比较模糊的时代。

下面是一个 Swift 实现的简单版，较容易理解：

```swift
func quicksort<T: Comparable>(_ a: [T]) -> [T] {
  // 递归终止条件
  guard a.count > 1 else { return a }
  
  // 
  let pivot = a[a.count/2]
  let less = a.filter { $0 < pivot }
  let equal = a.filter { $0 == pivot }
  let greater = a.filter { $0 > pivot }

  return quicksort(less) + equal + quicksort(greater)
}
```

将如下测试代码放到 playground 中运行：
```swift
let list = [ 10, 0, 3, 9, 2, 14, 8, 27, 1, 5, 8, -1, 26 ]
quicksort(list)
```

当给定一个数组，函数 `quicksort()` 首先通过变量 “pivot（中间数？）”将它切分成三部分。这里，我们通过取数组的中间元素来作为“pivot”（稍后会介绍其他取得 pivot 的方法）。

所有小于 pivot 的元素都放到 “less” 数组中。所有等于 pivot 的元素都放到 `equal` 数组中。接下来你可能已经猜到了，所有大于 pivot 的元素都放入 `greater` 数组中。这就是为什么范型 `T` 必须遵循 `Comparabel` 协议，只有这样，才能对元素进行 `<`, `==`, 以及 `>` 等比较操作。

一旦得到这三个数组，函数 `quicksort()` 就会对 `less` 数组以及 `greater` 数组进行递归排序，最终将排好序的自述组与 `equal` 数组合并，从而得到最终结果。

## 例子

下面通过例子来进行逐步讲解。初始数组如下：

	[ 10, 0, 3, 9, 2, 14, 8, 27, 1, 5, 8, -1, 26 ]

首先，我们选出 pivot 元素 `8`。这是因为它正好位于数组的中间位置。接下来，我们将数组分成 less, equal, 和 greater 三个部分：

	less:    [ 0, 3, 2, 1, 5, -1 ]
	equal:   [ 8, 8 ]
	greater: [ 10, 9, 14, 27, 26 ]

看来我们选择了一个不错的 pivot，刚好将原数组一切为二，使得 `less` 、 `greater` 包涵的元素数量相同。

注意此时 `less` 和 `greater` 都还是无序的。所以我们需要再次调用  `quicksort()`，以便对其进行排序。与之前操作相同，选出 pivot，将子数组继续切分成三个更小的数组。

这里我们仅以 `less` 数组为例进行说明：

	[ 0, 3, 2, 1, 5, -1 ]

此次我们选择的 pivot 元素是位于中间的 `1`（当然选择 `2`也可以，无关紧要）。再次通过 pivot 创建三个子数组：

	less:    [ 0, -1 ]
	equal:   [ 1 ]
	greater: [ 3, 2, 5 ]

还没完，继续对 `less` and `greater` 数组递归调用 `quicksort()`。继续对 `less` 进行说明：

	[ 0, -1 ]

选择 `-1`作为 pivot。此时拆分后对子数组如下：

	less:    [ ]
	equal:   [ -1 ]
	greater: [ 0 ]

因为已经没有比 `-1` 更小的数，所以`less` 数组为空；其它两个数组各包含一个数值。这意味着递归至此结束，我们可以继续对先前的 `greater` 进行排序了。

`greater` 数组如下：

	[ 3, 2, 5 ]

与先前相同：挑选位于中间的元素 `2` 作为 pivot，并以此拆分为子数组：

	less:    [ ]
	equal:   [ 2 ]
	greater: [ 3, 5 ]

注意，如果选择 `3` 作为 pivot，那么直接就可以结束了。但是现在不得不继续对 `greater` 进行递归操作，以保证它是排好序的。这也是为什么选择一个好的 pivot 至关重要。一旦你选择很多“糟糕”的 pivot，那么快速排序将会变得很慢。 More on that below.

继续对 `greater` 拆分后发现：

	less:    [ 3 ]
	equal:   [ 5 ]
	greater: [ ]

已经无法继续再分，递归至此已经结束。

这一处理过程会不断重复，直至所有的子数组都排好序。如下图：

![Example](Images/Example.png)

注意到图中高亮的元素了吗，正是排好序的数组：

	[ -1, 0, 1, 2, 3, 5, 8, 8, 9, 10, 14, 26, 27 ]

此图也表明了，`8` 是一个非常好的初始 pivot，因为它也正好位于排好序后数组的中间位置。

希望这能让你对快速排序的原理有一个大致的了解。但不幸的是，因为我们对同一个数组调用了三次 `filter()`，所以这个快速排序并不怎么快。还有更好的方式对数组进行拆分。

## 拆分（Partitioning）
围绕着 pivot 对数组进行分割的操作被称为 *拆分（Partitioning）*，有几种不同方案：

假设有如下数组：

	[ 10, 0, 3, 9, 2, 14, 8, 27, 1, 5, 8, -1, 26 ]

我们选择中间元素 `8` 作为 pivot，将数组进行拆分：


	[ 0, 3, 2, 1, 5, -1, 8, 8, 10, 9, 14, 27, 26 ]
	  -----------------        -----------------
	     所有元素 < 8               所有元素 > 8


The key thing to realize is that after partitioning the pivot element is in its final sorted place already. 
剩下的数值尚未排好序，只是简单的通过 pivot 进行了拆分（partition）。快速排序算法会反复对数组进行拆分，直到所有数值都位于正确的最终位置上。The rest of the numbers are not sorted yet, they are simply partitioned around the pivot value. Quicksort partitions the array many times over, until all the values are in their final places.

在拆分的过程中，各个数值之间的相对顺序是无法保证的。所以在以 `8` 作为 pivot 进行拆分后，数组可能看起来是这样的：
There is no guarantee that partitioning keeps the elements in the same relative order, so after partitioning around pivot `8` you could also end up with something like this:

	[ 3, 0, 5, 2, -1, 1, 8, 8, 14, 26, 10, 27, 9 ]

唯一可以保证的是，所有小于 pivot 的数值全部位于左侧，而位于右侧的所有数值都大于它。
The only guarantee is that to the left of the pivot are all the smaller elements and to the right are all the larger elements. Because partitioning can change the original order of equal elements, 快速排序不是一个稳定的排序算法（这一点与[归并排序（merge sort）不同](../Merge%20Sort/)。大多数情况，这一点影响不大。quicksort does not produce a "stable" sort (unlike [merge sort](../Merge%20Sort/), for example). Most of the time that's not a big deal.

## Lomuto 分治策略
在第一个例子中，我们通过调用三次 Swift 的 `filter()` 函数来进行拆分，（这直接导致该算法）不够高效。现在我们来实现一个相对更聪明的*原位*拆分算法，即直接修改原数组。
In the first example of quicksort I showed you, partitioning was done by calling Swift's `filter()` function three times. That is not very efficient. So let's look at a smarter partitioning algorithm that works *in place*, i.e. by modifying the original array.

下面是一个 Lomuto 拆分策略的 Swift 实现：
Here's an implementation of Lomuto's partitioning scheme in Swift:

```swift
// 这也是《算法导论》中提到的第一个快排方案
func partitionLomuto<T: Comparable>(_ a: inout [T], low: Int, high: Int) -> Int {
  let pivot = a[high]

  var i = low
  for j in low..<high {
    if a[j] <= pivot {
      (a[i], a[j]) = (a[j], a[i])
      i += 1
    }
  }

  (a[i], a[high]) = (a[high], a[i])
  return i
}
```

在 playground 中的测试代码如下：

```swift
var list = [ 10, 0, 3, 9, 2, 14, 26, 27, 1, 5, 8, -1, 8 ]
let p = partitionLomuto(&list, low: 0, high: list.count - 1)
list  // 显示结果
```

注意 `list` 必须是可变的 `var`，因为 `partitionLomuto()` 需要对数组（以 `inout` 参数传入）的内容进行直接修改。这样就比每次都创建一个新数组要高效多了。

为了避免每次都从头重新拆分，`low` 和 `high` 两个参数是必不可少的，只有限定范围才能逐步缩小。
The `low` and `high` parameters are necessary because when this is used inside quicksort, you don't always want to (re)partition the entire array, only a limited range that becomes smaller and smaller.

先前我们使用数组的中间元素作为基准元素，而 Lomuto 算法则总是使用*末尾*元素 `a[high]` 作为基准元素，认清这点不同非常关键。鉴于之前我们一直使用 `8` 作为基准元素，所以这次我们将 `8` 和 `26` 的位置进行交换，让 `8` 位于数组末尾，依然被选为基准元素。
Previously we used the middle array element as the pivot but it's important to realize that the Lomuto algorithm always uses the *last* element, `a[high]`, for the pivot. Because we've been pivoting around `8` all this time, I swapped the positions of `8` and `26` in the example so that `8` is at the end of the array and is used as the pivot value here too.

拆分之后的数组：
After partitioning, the array looks like this:

	[ 0, 3, 2, 1, 5, 8, -1, 8, 9, 10, 14, 26, 27 ]
	                        *

变量 `p` 包含的是 `partitionLomuto()` 后的返回值 `7`。它是基准元素在新数组中的索引（以星号标记）。 

左半段从 0 到 `p-1` 是 `[ 0, 3, 2, 1, 5, 8, -1 ]`。右半段从 `p+1` 到末尾是 `[ 9, 10, 14, 26, 27 ]` （右侧恰巧已经排好序了）。

你可能已经注意到了一个比较有趣的情况...那就是 `8` 在数组中有重复出现。
You may notice something interesting... The value `8` occurs more than once in the array. One of those `8`s did not end up neatly in the middle but somewhere in the left partition. 对 Lomuto 算法来说，这是一个小小对不利，重复元素越多对快速排序来说越不利。That's a small downside of the Lomuto algorithm as it makes quicksort slower if there are a lot of duplicate elements.

那么 Lomuto 算法是如何运行到呢？关键就位于 `for` 循环中。它将数组分成四个区域：

1. `a[low...i]` 包含所有 <= 基准元素的元素
2. `a[i+1...j-1]` 包含所有 > 基准元素对元素
3. `a[j...high-1]` are values we haven't looked at yet
4. `a[high]` 基准元素

So how does the Lomuto algorithm actually work? The magic happens in the `for` loop. This loop divides the array into four regions:

1. `a[low...i]` contains all values <= pivot
2. `a[i+1...j-1]` contains all values > pivot
3. `a[j...high-1]` are values we haven't looked at yet
4. `a[high]` is the pivot value

拆分后的数组如下：
In ASCII art the array is divided up like this:

	[ values <= pivot | values > pivot | not looked at yet | pivot ]
	  low           i   i+1        j-1   j          high-1   high

循环会从 `low` 到 `high-1` 便利每个元素，如果该元素的值小于等于基准元素，就通过交换将它移到第一个区域。
The loop looks at each element from `low` to `high-1` in turn. If the value of the current element is less than or equal to the pivot, it is moved into the first region using a swap.
> **注意：** 在 Swift 中，`(x, y) = (y, x)` 是对 `x` and `y` 进行交换对一个便利操作，与使用 `swap(&x, &y)` 是完全等价的。

> **Note:** In Swift, the notation `(x, y) = (y, x)` is a convenient way to perform a swap between the values of `x` and `y`. You can also write `swap(&x, &y)`.

循环结束后，基准元素仍然是数组中的最后一个元素，所以我们将它与大于基准元素部分的首元素进行交换。此时基准元素正好为与小于等于与大于它的两部分的中间，恰好将数组一分为二。

After the loop is over, the pivot is still the last element in the array. So we swap it with the first element that is greater than the pivot. Now the pivot sits between the <= and > regions and the array is properly partitioned.

继续该例子。此时数组是这样的：
Let's step through the example. The array we're starting with is:

	[| 10, 0, 3, 9, 2, 14, 26, 27, 1, 5, 8, -1 | 8 ]
	   low                                       high
	   i
	   j

初始时，从索引 0 到 11 为“未查看”区域，基准元素位于索引 12。由于查询尚未开始，所以 "values <= pivot" 和 "values > pivot" 的区域均为空。
Initially, the "not looked at" region stretches from index 0 to 11. The pivot is at index 12. The "values <= pivot" and "values > pivot" regions are empty, because we haven't looked at any values yet.

看第一个元素，`10`。是否小于基准元素？否，跳至下一个元素。  

	[| 10 | 0, 3, 9, 2, 14, 26, 27, 1, 5, 8, -1 | 8 ]
	   low                                        high
	   i
	       j

现在“未查看”区域变成了 1 到 11，"values > pivot" 的区域也包含了一个数字 `10`，而 "values <= pivot" 到区域依然为空。

在查看第二个数值，`0`。是否比基准元素小呢？是，那么将 `10` 和 `0` 进行交换，并且将 `i` 向前移动一位。

	[ 0 | 10 | 3, 9, 2, 14, 26, 27, 1, 5, 8, -1 | 8 ]
	  low                                         high
	      i
	           j

此时“未查看”区域变成了 2 到 11，"values > pivot"的区域还是只包含 `10`，而"values <= pivot"的区域则多了个 `0`。

继续查看第三个数值，`3`。比基准元素小，所以与 `10` 交换后得到：

	[ 0, 3 | 10 | 9, 2, 14, 26, 27, 1, 5, 8, -1 | 8 ]
	  low                                         high
	         i
	             j

"values <= pivot"的区域变成了 `[ 0, 3 ]`。继续查看...。`9` 比基准元素大，所以跳过：

	[ 0, 3 | 10, 9 | 2, 14, 26, 27, 1, 5, 8, -1 | 8 ]
	  low                                         high
	         i
	                 j

这是"values > pivot"的区域变成了 `[ 10, 9 ]`。如果按此操作继续，那么最终结果会是：

	[ 0, 3, 2, 1, 5, 8, -1 | 27, 9, 10, 14, 26 | 8 ]
	  low                                        high
	                         i                   j

还有最后一件事要做，那就是通过交换 `a[i]` 和 `a[high]`，将基准元素放回原位
The final thing to do is to put the pivot into place by swapping `a[i]` with `a[high]`:

	[ 0, 3, 2, 1, 5, 8, -1 | 8 | 9, 10, 14, 26, 27 ]
	  low                                       high
	                         i                  j

此外还返回了 `i`，基准元素的索引。
> **注意：** 如果你对该算法的工作原理还是不完全明白，那么我建议你到 playground 中亲自尝试一番，在循环过程中，仔细观察一下这四个区域是如何创建的。

我们以此分治策略来构建一个快速排序。代码如下：

```swift
func quicksortLomuto<T: Comparable>(_ a: inout [T], low: Int, high: Int) {
  if low < high {
    let p = partitionLomuto(&a, low: low, high: high)
    quicksortLomuto(&a, low: low, high: p - 1)
    quicksortLomuto(&a, low: p + 1, high: high)
  }
}
```

现在是不是一下简单很多。我们首先调用 `partitionLomuto()` 函数，以数组的最后一个元素为基准元素，对数组进行重排。然后在对左右两部分递归调用`quicksortLomuto()`，进行排序。

测试一下:

```swift
var list = [ 10, 0, 3, 9, 2, 14, 26, 27, 1, 5, 8, -1, 8 ]
quicksortLomuto(&list, low: 0, high: list.count - 1)
```

Lomuto 策略虽不是唯一的分治策略，但它却是最容易理解的。与交换操作更少的 Hoare 分治策略相比，在效率上略逊一筹。

## Hoare's 分治策略
该分治方案由快速算法的发明者 Hoare 创建。

代码如下：

```swift
func partitionHoare<T: Comparable>(_ a: inout [T], low: Int, high: Int) -> Int {
  let pivot = a[low]
  var i = low - 1
  var j = high + 1

  while true {
    repeat { j -= 1 } while a[j] > pivot
    repeat { i += 1 } while a[i] < pivot

    if i < j {
      a.swapAt(i, j)
    } else {
      return j
    }
  }
}
```

playground 测试代码如下:

```swift
var list = [ 8, 0, 3, 9, 2, 14, 10, 27, 1, 5, 8, -1, 26 ]
let p = partitionHoare(&list, low: 0, high: list.count - 1)
list  // show the results
```

注意在 Hoare 方案中，总是以数组的*第一个*元素，`a[low]` 作为基准元素。同样，我们还是以 `8` 作为基准元素。

结果如下:

	[ -1, 0, 3, 8, 2, 5, 1, 27, 10, 14, 9, 8, 26 ]

注意这次基准元素并没有位于正中。与 Lomuto 方案不同，返回值，即基准元素在新数组中的索引，并不是必须的。
Note that this time the pivot isn't in the middle at all. Unlike with Lomuto's scheme, the return value is not necessarily the index of the pivot element in the new array.

数组将被分成  `[low...p]` 以及 `[p+1...high]` 两部分。例如，返回值 `p` 是 6，那么这两部分将分别是` [ -1, 0, 3, 8, 2, 5, 1 ]` 和 `[ 27, 10, 14, 9, 8, 26 ]`。
Instead, the array is partitioned into the regions `[low...p]` and `[p+1...high]`. Here, the return value `p` is 6, so the two partitions are `[ -1, 0, 3, 8, 2, 5, 1 ]` and `[ 27, 10, 14, 9, 8, 26 ]`.

基准元素会位于两个部分中的一个，但算法并没有明确指明具体是哪一个。如果基准元素出现不止一次。那么在具体例子中，它技能位于左半边，也可能位于右半边。
The pivot is placed somewhere inside one of the two partitions, but the algorithm doesn't tell you which one or where. If the pivot value occurs more than once, then some instances may appear in the left partition and others may appear in the right partition.

基于这点不同，Hoare 快速排序算法的实现也有些许差异：

```swift
func quicksortHoare<T: Comparable>(_ a: inout [T], low: Int, high: Int) {
  if low < high {
    let p = partitionHoare(&a, low: low, high: high)
    quicksortHoare(&a, low: low, high: p)
    quicksortHoare(&a, low: p + 1, high: high)
  }
}
```

至于 Hoare 分治策略的详细步骤，我就把它当作练习，留给各位读者了。 :-)

## 选一个好的基准元素（Picking a good pivot）
虽然 Lomuto 分治策略总是以末位元素作为基准元素，而 Hoare 策略则则总用首元素，但没有任何证据表明这么做有什么优势。
Lomuto's partitioning scheme always chooses the last array element for the pivot. Hoare's scheme uses the first element. But there is no guarantee that these pivots are any good.

下面向你展示一下如果选了一个糟糕的基准元素会如何。以如下数组为例：

	[ 7, 6, 5, 4, 3, 2, 1 ]

采用 Lomuto 分治策略。基准元素为末位元素，`1`。经拆分，得到如下数组：

	   less than pivot: [ ]
	    equal to pivot: [ 1 ]
	greater than pivot: [ 7, 6, 5, 4, 3, 2 ]

递归对“大于”部分进行拆分：

	   less than pivot: [ ]
	    equal to pivot: [ 2 ]
	greater than pivot: [ 7, 6, 5, 4, 3 ]

继续:

	   less than pivot: [ ]
	    equal to pivot: [ 3 ]
	greater than pivot: [ 7, 6, 5, 4 ]

以此重复下去...

糟糕，直接将快速排序降级成插入排序了。为了让快速排序更效率，需要将它尽可能的平分。
That's no good, because this pretty much reduces quicksort to the much slower insertion sort. For quicksort to be efficient, it needs to split the array into roughly two halves.

在这个例子中，最优的基准元素应该是 `4`，这样可以得到：
The optimal pivot for this example would have been `4`, so we'd get:

	   less than pivot: [ 3, 2, 1 ]
	    equal to pivot: [ 4 ]
	greater than pivot: [ 7, 6, 5 ]

那么是不是选择中间元素就一定比首末元素更好呢？试着看下这个例子：

	[ 7, 6, 5, 1, 4, 3, 2 ]

此时中间元素是 `1`，如果以它为基准元素，那么会导致跟之前例子中一样糟糕的结果。
Now the middle element is `1` and that gives the same lousy results as in the previous example.

理想的基准元素应该是待分数组的*中位数*，也就是位于有序数组正中的那个元素。然而，排序之前，你又不可能知道。所以这有点像先有鸡还是先有蛋的问题。尽管如此，我们还有一些技巧可以选择一个相对较好的基准元素。
Ideally, the pivot is the *median* element of the array that you're partitioning, i.e. the element that sits in the middle of the sorted array. Of course, you won't know what the median is until after you've sorted the array, so this is a bit of a chicken-and-egg problem. However, there are some tricks to choose good, if not ideal, pivots.

其中一个技巧是，取“三元素的中位数”，即子数组首、中、末三个元素的中位数。理论上而言，它通常是最接近真实中位数的。

另一个常见的解决方案是随机选择基准元素。虽然有时选中的基准元素不怎么理想，但平均而言，这是一个非常不错的选择。

下面展示一个通过随机基准元素的快速排序算法实现：

```swift
func quicksortRandom<T: Comparable>(_ a: inout [T], low: Int, high: Int) {
  if low < high {
    let pivotIndex = random(min: low, max: high)         // 1

    (a[pivotIndex], a[high]) = (a[high], a[pivotIndex])  // 2

    let p = partitionLomuto(&a, low: low, high: high)
    quicksortRandom(&a, low: low, high: p - 1)
    quicksortRandom(&a, low: p + 1, high: high)
  }
}
```

与之前相比有两点重要区别：

1. `random(min:max:)` 函数会返回一个位于 `min...max` 区间内的整数，作为基准元素的索引。
2. 由于 Lomuto 分治策略总是以 `a[high]` 作为基准元素，所以在调用 `partitionLomuto()` 之前，我们先对`a[pivotIndex]` 和 `a[high]` 进行交换，以便将基准元素放置末位。

虽然在排序算法中使用随机数看上去有点怪，但为让快速排序在所有情况下都相对高效，这一操作就是必不可少的。一个糟糕的基准元素可以让快速排序的时间复杂度飙至 **O(n^2)**，而如果你借助随机数生成器，通常都能选择一个较好的基准元素，那么它的时间复杂度就会变为 **O(n log n)**，这样的排序算法就相当不错了。
It may seem strange to use random numbers in something like a sorting function, but it is necessary to make quicksort behave efficiently under all circumstances. With bad pivots, the performance of quicksort can be quite horrible, **O(n^2)**. But if you choose good pivots on average, for example by using a random number generator, the expected running time becomes **O(n log n)**, which is as good as sorting algorithms get.

## 荷兰国立标记分治法（Dutch national flag partitioning）

> 译者：这是作者搞怪起的小标题，实际是作者对基准元素重复情况下的一种优化，改进。

But there are more improvements to make! In the first example of quicksort I showed you, we ended up with an array that was partitioned like this:

	[ values < pivot | values equal to pivot | values > pivot ]

But as you've seen with the Lomuto partitioning scheme, if the pivot occurs more than once the duplicates end up in the left half. And with Hoare's scheme the pivot can be all over the place. The solution to this is "Dutch national flag" partitioning, named after the fact that the [Dutch flag](https://en.wikipedia.org/wiki/Flag_of_the_Netherlands) has three bands just like we want to have three partitions.

The code for this scheme is:

```swift
func partitionDutchFlag<T: Comparable>(_ a: inout [T], low: Int, high: Int, pivotIndex: Int) -> (Int, Int) {
  let pivot = a[pivotIndex]

  var smaller = low
  var equal = low
  var larger = high

  while equal <= larger {
    if a[equal] < pivot {
      swap(&a, smaller, equal)
      smaller += 1
      equal += 1
    } else if a[equal] == pivot {
      equal += 1
    } else {
      swap(&a, equal, larger)
      larger -= 1
    }
  }
  return (smaller, larger)
}
```

This works very similarly to the Lomuto scheme, except that the loop partitions the array into four (possibly empty) regions:

- `[low ... smaller-1]` contains all values < pivot
- `[smaller ... equal-1]` contains all values == pivot
- `[equal ... larger]` contains all values > pivot
- `[larger ... high]` are values we haven't looked at yet

Note that this doesn't assume the pivot is in `a[high]`. Instead, you have to pass in the index of the element you wish to use as a pivot.

An example of how to use it:

```swift
var list = [ 10, 0, 3, 9, 2, 14, 8, 27, 1, 5, 8, -1, 26 ]
partitionDutchFlag(&list, low: 0, high: list.count - 1, pivotIndex: 10)
list  // show the results
```

Just for fun, we're giving it the index of the other `8` this time. The result is:

	[ -1, 0, 3, 2, 5, 1, 8, 8, 27, 14, 9, 26, 10 ]

Notice how the two `8`s are in the middle now. The return value from `partitionDutchFlag()` is a tuple, `(6, 7)`. This is the range that contains the pivot value(s).

Here is how you would use it in quicksort:

```swift
func quicksortDutchFlag<T: Comparable>(_ a: inout [T], low: Int, high: Int) {
  if low < high {
    let pivotIndex = random(min: low, max: high)
    let (p, q) = partitionDutchFlag(&a, low: low, high: high, pivotIndex: pivotIndex)
    quicksortDutchFlag(&a, low: low, high: p - 1)
    quicksortDutchFlag(&a, low: q + 1, high: high)
  }
}
```

Using Dutch flag partitioning makes for a more efficient quicksort if the array contains many duplicate elements. (And I'm not just saying that because I'm Dutch!)

> **Note:** The above implementation of `partitionDutchFlag()` uses a custom `swap()` routine for swapping the contents of two array elements. Unlike Swift's own `swap()`, this doesn't give an error when the two indices refer to the same array element. See [Quicksort.swift](Quicksort.swift) for the code.

## 参见

[Quicksort on Wikipedia](https://en.wikipedia.org/wiki/Quicksort)

*Written for Swift Algorithm Club by Matthijs Hollemans*
