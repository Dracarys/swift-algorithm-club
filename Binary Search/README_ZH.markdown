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

下面将详解讲解该算法的运行原理。

之前例子中的数组内含19个已经排好序的数字，如下：

	[ 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67 ]

假设我们要查找数字 `43`。

要将数组进行对分，就需要先确定中间位置的索引。可以通过方式取得：

```swift
let midIndex = range.lowerBound + (range.upperBound - range.lowerBound) / 2
```

初始，range 的`lowerBound = 0`，`upperBound = 19`。代入上面的方程求`midIndex` 得 `0 + (19 - 0)/2 = 19/2 = 9`。实际应为 `9.5`，但由于索引必须为整数，所以向下取整数。

如下， `*` 标记的真实中间位置的元素，可以看到，标记两边的元素个数是相同的，此时正好将数组从中间一分为二。

	[ 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67 ]
                                      *

接下来，需要判断在那半边查找。相关判断逻辑如下：

```swift
if a[midIndex] > key {
    // use left half
} else if a[midIndex] < key {
    // use right half
} else {
    return midIndex
}
```

由于 `a[midIndex] = 29`，小于目标值，因此可以确定目标值肯定不会在左半边，因为左半边都是小于 `29` 的数，相应的，目标值就可能在右半边（或者不存在）。

接下来，再对 `midIndex + 1` 到 `range.upperBound` 这部分数组进行二分查找：

	[ x, x, x, x, x, x, x, x, x, x | 31, 37, 41, 43, 47, 53, 59, 61, 67 ]

由于不用在考虑左半边的数组内容，所以这里将这部分元素统一用 `x` 标记。现在开始我们将只关注右半边即可，它在树中的起始索引是 10 。

重新求中间元素索引：`midIndex = 10 + (19 - 10)/2 = 14`，再次从中间分开。

	[ x, x, x, x, x, x, x, x, x, x | 31, 37, 41, 43, 47, 53, 59, 61, 67 ]
	                                                 *
此时，`a[14]` 正好位于数组中半边的中间位置。

目标值是否小于 `a[14]` 呢？ `43 < 47`。这次选择左半边，将右边大于的部分忽略掉:

	[ x, x, x, x, x, x, x, x, x, x | 31, 37, 41, 43 | x, x, x, x, x ]

新的 `midIndex` 在这:

	[ x, x, x, x, x, x, x, x, x, x | 31, 37, 41, 43 | x, x, x, x, x ]
	                                     *

目标值大于 `37`, 所以看右半边:

	[ x, x, x, x, x, x, x, x, x, x | x, x | 41, 43 | x, x, x, x, x ]
	                                        *
还是目标值较大，那么继续对右半边进行对分：

	[ x, x, x, x, x, x, x, x, x, x | x, x | x | 43 | x, x, x, x, x ]
	                                            *

好了，目标值正好与我们找到的数组元素相等，终于找到：数字 `43` 在数组中的索引是 `13`。耶！

看上去好像工作量不小，但实际上只用光了 4 步就找到了，恰好是 `log_2(19) = 4.23`。如果是线性查找，就会变成 14 步之多。

如果目标值是 `42` 而不是 `43` 呢？那么最后将无法对数组进行在分。`range.upperBound` 会变的小于 `range.lowerBound`。这说明目标值不在该数组中，并返回 `nil`。

> **注意：** 众多二分查找实现中涉及到的求 `midIndex = (lowerBound + upperBound) / 2` 方程，在处理非常大的数组时有个问题，那就是 `lowerBound + upperBound` 可能超出整型所能容纳的范围，导致溢出。该问题几乎不会在 64 位 CPU 上发生，但是 32 位是绝对有可能。

## 迭代 VS 递归

二分查找是一个递归（逻辑），因为过程中总是在不断重复地缩小数组的范围。然而，这并不意味着就必须以递归的方式去实现 `binarySearch()` 函数。通常更高效的做法是将递归算法用迭代的方式去实现，用简单的循环替换大量递归调用。

下面是通过 Swift 以迭代的方式实现的二分查找：

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

可以看到，代码比递归的方式少了很多。最大的区别就在 `while` 循环中。

用法如下：

```swift
let numbers = [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67]

binarySearch(numbers, key: 43)  // gives 13
```

## 结束语

数组必须先排序这会是问题吗？看情况。别忘了排序也是要耗时的 —— 排序加上二分查找可能比简单的线性查找还是慢。只有那些一次排序，多次查询的场景才能让二分查找绽放出光辉。

参见[维基百科（英）](https://en.wikipedia.org/wiki/Binary_search_algorithm).

*Written for Swift Algorithm Club by Matthijs Hollemans*
