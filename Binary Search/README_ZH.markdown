# 二分查找（Binary Search）

目标：在数组中快速查询某个元素。

假定有一个包涵大量数值的数组，现在需要判断是否包含某个特定值，以及这个值的索引。

大多数情况下，Swift 自带的 `Collection.index(of:)` 函数就够用了：

```swift
let numbers = [11, 59, 3, 2, 53, 17, 31, 7, 19, 67, 47, 13, 37, 61, 29, 43, 5, 41, 23]

numbers.index(of: 43)  // returns 15
```

内建的 `Collection.index(of:)` 函数执行 [线性查找（linear search）](../Linear%20Search/)。代码如下：

```swift
func linearSearch<T: Equatable>(_ a: [T], _ key: T) -> Int? {
    for i in 0 ..< a.count {
        if a[i] == key {
            return i
        }
    }
    return nil
}
```

应用方法如下：

```swift
linearSearch(numbers, 43)  // returns 15
```

这有什么问题吗？`linearSearch()` 从头开始循环遍历整个数组，直到找到那个数。最糟糕的情况就是，那个数不存在，整个遍历做了无用功。

平均来说，线性查找算法至少要查找数组中一半的数值。如果你的数组够大，那么这将变的异常缓慢。

## 分而治之（Divide and conquer）

典型的增效方法就是采用 *二分查找*。该方法持续将数组对半分开，直到找到目标数值。

对于大小为 `n` 的数组，查找的时间杂度不在是线性查找时的 **O(n)**，而仅仅是 **O(log n)**。举例来说，一个拥有 1,000,000 元素的数组，只要 20 步即可找到你要的元素，因为 `log_2(1,000,000) = 19.9`。对于十亿级的也仅需要 30 步。（又来了，上次你接触十亿级数组时什么时候？）（译者：就没遇到，百万都没遇到过）

哇，听上去棒极了是不是，但是二分查找也有它的缺点：待查的数组必须时有序的。实践过程中，这通常不是问题。

下面是二分查找的原理：

- 先将数组对半分成两段，然后判断你要查找的*搜索关键词*是在左半段，还是在右半段。
- 怎么判断在那半段呢？这就是为什么需要先将数组排序，因为这样你就可以进行 `<` 或 `>` 比较了。
- 如果在左半边，那么重复这一过程：继续将左半段对分成更小的段，然后再判断在那一段。
- 如此重复，直至找到。如果数组无法在分，那就只能遗憾地表示找不到，即不存在。

现在应该明白为什么叫“二分”查找了吧：每一步都在将数组拆分成两段。这种*分而治之*的过程可以快速地缩小范围锁定目标位置。

## 代码

下面是用 SWift 实现的递归二分查找:

```swift
func binarySearch<T: Comparable>(_ a: [T], key: T, range: Range<Int>) -> Int? {
    if range.lowerBound >= range.upperBound {
        // If we get here, then the search key is not present in the array.
        return nil

    } else {
        // Calculate where to split the array.
        let midIndex = range.lowerBound + (range.upperBound - range.lowerBound) / 2

        // Is the search key in the left half?
        if a[midIndex] > key {
            return binarySearch(a, key: key, range: range.lowerBound ..< midIndex)

        // Is the search key in the right half?
        } else if a[midIndex] < key {
            return binarySearch(a, key: key, range: midIndex + 1 ..< range.upperBound)

        // If we get here, then we've found the search key!
        } else {
            return midIndex
        }
    }
}
```

要尝试，清将代码拷贝到 playground 中，在执行如下操作:

```swift
let numbers = [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67]

binarySearch(numbers, key: 43, range: 0 ..< numbers.count)  // gives 13
```

注意数组 `numbers` 是有序的。否则二分查找算法将是无效的！

虽然二分查找是将数组以对分的方式进行查询，但实际上（这个过程中）并不会创建两个新的数组。相反只是通过 Swift 的 	`Range` 对象进行记录。初始，range 涵盖整个数组，`0 ..< numbers.count`，随着数组的对分，range 也随之变的越来越小。

> **注意：** 有一点要小心。`range.upperBound` 始终指向最后一个元素之后。在例子中，因为数组包含19个数，所以range 是 `0..<19`，`range.lowerBound = 0`，并且 `range.upperBound = 19`。但是数组中最后一个元素的真实索引是18，而不是19，是从 0 开始的。应用 range时，务必时刻牢记：`upperBound` 总是比末元素的索引大一。

## 逐步讲解

It might be useful to look at how the algorithm works in detail.

The array from the above example consists of 19 numbers and looks like this when sorted:

	[ 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67 ]

We're trying to determine if the number `43` is in this array.

To split the array in half, we need to know the index of the object in the middle. That's determined by this line:

```swift
let midIndex = range.lowerBound + (range.upperBound - range.lowerBound) / 2
```

Initially, the range has `lowerBound = 0` and `upperBound = 19`. Filling in these values, we find that `midIndex` is `0 + (19 - 0)/2 = 19/2 = 9`. It's actually `9.5` but because we're using integers, the answer is rounded down.

In the next figure, the `*` shows the middle item. As you can see, the number of items on each side is the same, so we're split right down the middle.

	[ 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67 ]
                                      *

Now binary search will determine which half to use. The relevant section from the code is:

```swift
if a[midIndex] > key {
    // use left half
} else if a[midIndex] < key {
    // use right half
} else {
    return midIndex
}
```

In this case, `a[midIndex] = 29`. That's less than the search key, so we can safely conclude that the search key will never be in the left half of the array. After all, the left half only contains numbers smaller than `29`. Hence, the search key must be in the right half somewhere (or not in the array at all).

Now we can simply repeat the binary search, but on the array interval from `midIndex + 1` to `range.upperBound`:

	[ x, x, x, x, x, x, x, x, x, x | 31, 37, 41, 43, 47, 53, 59, 61, 67 ]

Since we no longer need to concern ourselves with the left half of the array, I've marked that with `x`'s. From now on we'll only look at the right half, which starts at array index 10.

We calculate the index of the new middle element: `midIndex = 10 + (19 - 10)/2 = 14`, and split the array down the middle again.

	[ x, x, x, x, x, x, x, x, x, x | 31, 37, 41, 43, 47, 53, 59, 61, 67 ]
	                                                 *

As you can see, `a[14]` is indeed the middle element of the array's right half.

Is the search key greater or smaller than `a[14]`? It's smaller because `43 < 47`. This time we're taking the left half and ignore the larger numbers on the right:

	[ x, x, x, x, x, x, x, x, x, x | 31, 37, 41, 43 | x, x, x, x, x ]

The new `midIndex` is here:

	[ x, x, x, x, x, x, x, x, x, x | 31, 37, 41, 43 | x, x, x, x, x ]
	                                     *

The search key is greater than `37`, so continue with the right side:

	[ x, x, x, x, x, x, x, x, x, x | x, x | 41, 43 | x, x, x, x, x ]
	                                        *

Again, the search key is greater, so split once more and take the right side:

	[ x, x, x, x, x, x, x, x, x, x | x, x | x | 43 | x, x, x, x, x ]
	                                            *

And now we're done. The search key equals the array element we're looking at, so we've finally found what we were searching for: number `43` is at array index `13`. w00t!

It may have seemed like a lot of work, but in reality it only took four steps to find the search key in the array, which sounds about right because `log_2(19) = 4.23`. With a linear search, it would have taken 14 steps.

What would happen if we were to search for `42` instead of `43`? In that case, we can't split up the array any further. The `range.upperBound` becomes smaller than `range.lowerBound`. That tells the algorithm the search key is not in the array and it returns `nil`.

> **Note:** Many implementations of binary search calculate `midIndex = (lowerBound + upperBound) / 2`. This contains a subtle bug that only appears with very large arrays, because `lowerBound + upperBound` may overflow the maximum number an integer can hold. This situation is unlikely to happen on a 64-bit CPU, but it definitely can on 32-bit machines.

## 迭代 VS 递归

Binary search is recursive in nature because you apply the same logic over and over again to smaller and smaller subarrays. However, that does not mean you must implement `binarySearch()` as a recursive function. It's often more efficient to convert a recursive algorithm into an iterative version, using a simple loop instead of lots of recursive function calls.

Here is an iterative implementation of binary search in Swift:

```swift
func binarySearch<T: Comparable>(_ a: [T], key: T) -> Int? {
    var lowerBound = 0
    var upperBound = a.count
    while lowerBound < upperBound {
        let midIndex = lowerBound + (upperBound - lowerBound) / 2
        if a[midIndex] == key {
            return midIndex
        } else if a[midIndex] < key {
            lowerBound = midIndex + 1
        } else {
            upperBound = midIndex
        }
    }
    return nil
}
```

As you can see, the code is very similar to the recursive version. The main difference is in the use of the `while` loop.

Use it like this:

```swift
let numbers = [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67]

binarySearch(numbers, key: 43)  // gives 13
```

## 结束语

数组必须先排序这会是问题吗？看情况。别忘了排序也是要耗时的 —— 排序加上二分查找可能比简单的线性查找还是慢。只有那些一次排序，多次查询的场景才能让二分查找绽放出光辉。

参见[维基百科（英）](https://en.wikipedia.org/wiki/Binary_search_algorithm).

*Written for Swift Algorithm Club by Matthijs Hollemans*
