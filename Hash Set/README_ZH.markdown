# 哈希集合（Hash Set）

集合（set）是一个类似数组的元素集，但有两点不同：集合内元素是无序的，且仅出现一次。

如果下面展示的是数组，那么它们是不同的，但如果是集合，那么它们就是相同的：

```swift
[1 ,2, 3]
[2, 1, 3]
[3, 2, 1]
[1, 2, 2, 3, 1]
```

这是因为每个元素仅能出现一次，无论你写入多少次，都仅计入一个。

> **注意：** 当集合内元素顺序无关紧要时，我个人更倾向于使用集合而不是数组。通常而言，大多数编程都与元素顺序无关。

集合的一些典型操作：

- 插入一个元素
- 移除一个元素
- 检查是否已经包含某个元素
- 与另一个集合取并集（Union）
- 与另一个集合取交集（intersection）
- 计算与另一个集合的差集（difference，相对补集合）

并集，交集，和差集是将两个集合组合成一个集合的几种不同形式：

![Union, intersection, difference](Images/CombineSets.png)

从 Swift 1.2 开始，标准库中内建了一个 `Set` 类型，但这里我将想你展示如何自己构建一个。虽然产品中不会用到，但是它能让你明白集合是怎么实现的。

可以通过一个简单的数组来实现集合，但是那样不够高效。这里我们使用字典。Swift 内建的字典是一个哈希表，那么我们实现的这个集合就是哈希集合了。

## 代码

Swift 实现的初始 `HashSet`：

```swift
public struct HashSet<T: Hashable> {
    fileprivate var dictionary = Dictionary<T, Bool>()

    public init() {

    }

    public mutating func insert(_ element: T) {
        dictionary[element] = true
    }

    public mutating func remove(_ element: T) {
        dictionary[element] = nil
    }

    public func contains(_ element: T) -> Bool {
        return dictionary[element] != nil
    }

    public func allElements() -> [T] {
        return Array(dictionary.keys)
    }

    public var count: Int {
        return dictionary.count
    }

    public var isEmpty: Bool {
        return dictionary.isEmpty
    }
}
```

代码非常简单，这是因为累活都被 Swift 内建的字典干了。之所以选择字典，是因为字典的键也必须是唯一的，跟集合的要求一样。此外，字典的大部分操作都具有 **O(1)** 时间复杂度，如此实现的集合会非常高效。

由于我们使用了字典，那么范型 `T` 就必须遵循 `Hashable` 协议。这样我们就可以想集合中插入任何可哈希的对象。（Swift 内建的 `Set` 也是如此）

通常，在字典中将键和值关联起来，但是因为集合只需关注键，所以这里我们使用 `Bool` 作为值的类型。即便如此，也只会将它设置为 `true`，而不是 `false`。（当然选择其它类也可以，但是布尔类型占用的空间是最小的）

将代码拷贝到 playground，在添加一些测试代码：

```swift
var set = HashSet<String>()

set.insert("one")
set.insert("two")
set.insert("three")
set.allElements()      // ["one, "three", "two"]

set.insert("two")
set.allElements()      // still ["one, "three", "two"]

set.contains("one")    // true
set.remove("one")
set.contains("one")    // false
```

`allElements()` 函数会将集合中所有内容转换为一个数组。注意数组中的元素顺序可能与你添加的顺序不同。正如之前提及的，集合内的元素是无序的。（跟字典一样）

## 合并集合（Combining sets）

集合的一大用处就是如何合并它们。下面是求并集的代码:

```swift
extension HashSet {
    public func union(_ otherSet: HashSet<T>) -> HashSet<T> {
        var combined = HashSet<T>()
        for obj in self.dictionary.keys {
            combined.insert(obj)
        }
        for obj in otherSet.dictionary.keys {
            combined.insert(obj)
        }
        return combined
    }
}
```

求两个集合的 *并集（union）* 会创建一个新的集合，它包含 A，B 两个集合中的所有元素。当然重复元素仅计入一次。

例如:

```swift
var setA = HashSet<Int>()
setA.insert(1)
setA.insert(2)
setA.insert(3)
setA.insert(4)

var setB = HashSet<Int>()
setB.insert(3)
setB.insert(4)
setB.insert(5)
setB.insert(6)

let union = setA.union(setB)
union.allElements()           // [5, 6, 2, 3, 1, 4]
```

如你所见，现在两个集合的并集包含所有元素。`3` 和 `4` 也仅出现一次，即使它们原来各自被包含在两个集合内。

*并集（intersection）*是指两个集合共有的元素。代码如下：

```swift
extension HashSet {
    public func intersect(_ otherSet: HashSet<T>) -> HashSet<T> {
        var common = HashSet<T>()
        for obj in dictionary.keys {
            if otherSet.contains(obj) {
                common.insert(obj)
            }
        }
        return common
    }
}
```

测试一下:

```swift
let intersection = setA.intersect(setB)
intersection.allElements()
```

打印结果为 `[3, 4]`，这是一位呢它们集出现在 A 集合，又出现在 B 集合。

最后，求*差集（difference）*会移除两个集合共有的元素。代码如下：

```swift
extension HashSet {
    public func difference(_ otherSet: HashSet<T>) -> HashSet<T> {
        var diff = HashSet<T>()
        for obj in dictionary.keys {
            if !otherSet.contains(obj) {
                diff.insert(obj)
            }
        }
        return diff
    }
}
```

正好与交集 `intersect()` 相反，试一下：

```swift
let difference1 = setA.difference(setB)
difference1.allElements()                // [2, 1]

let difference2 = setB.difference(setA)
difference2.allElements()                // [5, 6]
```

## 接下来

如果你看过 Swift 内建 `Set` 的[文档](http://swiftdoc.org/v2.1/type/Set/)，就会注意到它包含由更多的函数。一个最显著的扩展就是让 `HashSet` 遵循 `SequenceType`, 如此就可以通过 `for`...`in` 对集合进行遍历

另一个可以尝试的操作是，用一个真正的[hash table](../Hash%20Table)代替 `Dictionary`，这样只需存储键即可，无需在关联其他，也就不需要 `Bool` 类型的值。

如果你将长需要查询一个元素是否属于某个集合，以及求交集等操作，那么[union-find](../Union-Find/)这个数据结构可能更合适。它使用一个树来代替字典，让查询和求并集操作更高效。

> **注意：** ~~我更倾向于让 `HashSet` 遵从 `ArrayLiteralConvertible`，如此就可以这样写了：`let setA: HashSet<Int> = [1, 2, 3, 4]`，但是目前这样会导致编译器崩溃~~。

> **注意：** 译者：原来的 `ArrayLiteralConvertible ` 已经被替换为 `ExpressibleByArrayLiteral`，现在已经可以正常使用了。

*由 Matthijs Hollemans 为 Swift 算法合集撰写*

*由 William Han 翻译*
