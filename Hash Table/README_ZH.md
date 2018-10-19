# 哈希表（Hash Table）

哈希表允许你通过“key（键）”来访问对象。

哈希表通常的实现结构有 dictionary，map，associative array，它们的实现可以是一个树，也可以是数组，但最效率最高的还是哈希表。

这也说明了为什么 Swift 内建的 `Dictionary` 类型要求 key（键）必须遵循 `Hashable` 协议，因为它内部应用了哈希表，也就是我们接下来马上就学习的内容。

## 原理（How it works）

哈希表跟数组没什么两样。初始，这个数组是空的，当你通过 key（键）将一个 value（值）存放到数组中时，它会对 key（键）进行计算，从而得到数组内的一个索引（index）。下面是例子：

```swift
hashTable["firstName"] = "Steve"

The hashTable array:
+--------------+
| 0:           |
+--------------+
| 1:           |
+--------------+
| 2:           |
+--------------+
| 3: firstName |---> Steve
+--------------+
| 4:           |
+--------------+
```

在这个例子中， `"firstName"` key（键）对应到了数组的索引 3。

在添加另一对键值，它被映射到了数组内的另一个索引：

```swift
hashTable["hobbies"] = "Programming Swift"

The hashTable array:
+--------------+
| 0:           |
+--------------+
| 1: hobbies   |---> Programming Swift
+--------------+
| 2:           |
+--------------+
| 3: firstName |---> Steve
+--------------+
| 4:           |
+--------------+
```

哈希表的关键就在于如何映射到相应的数组索引。当你写出如下表达式时，哈希随之进行。

```swift
hashTable["firstName"] = "Steve"
```

哈希表接收到键 `"firstName"`，随即访问该键的 `hashValue` 属性，因此键必须是 `Hashable` 的。

`"firstName".hashValue` 会返回一个大整数：-4799450059917011053。`"hobbies".hashValue`返回的哈希值是 4799450060928805186。（这些数值看上去是无规律的）

这些数值都太大了，完全没法映射到数组中的索引，甚至还有负值！通常的做法是先对整数取绝对值，然后在针对数组的大小进行取模运算。

我们数组的大小是 5，所以 `"firstName"` 键映射到的索引是 `abs(-4799450059917011053) % 5 = 3` 。你可以自己计算下 `"hobbies"`键，它的索引应该是 1。

正式哈希才使 dictionary（字典）变的高效，查找一个元素时，首先通过哈谢得到数组中对应的索引，然后直接根据索引取得对应的元素。所有这些操作耗时都是恒定的，因此无论是添加、读取、还是删除对象，它的时间复杂度都是 **O(1)**。

> **注意:** 由于不能预先知晓哈希后的索引会对应到哪里，所以字典是不能保证哈希表中任何元素顺序的。

## 避免碰撞（Avoiding collisions）

有这样一个问题：由于我们是通过数组的大小对哈希值进行取模运算，那就有可能两个或着更多的键对应到一个数组索引。这种情况被称为碰撞。

解决方法之一就是创建个大数组，从而减少两个键同时映射到一个索引的情况。另一个办法就是选择一个数值作为数组的大小（该数值大于数组的容量）。然而，碰撞还是会出现，所以我们要找到一个终极解决办法。

由于我们的表很小，所以非常容易出现碰撞。例如，`"lastName"`键就也被映射到了索引 3，但是我们不想原来已经存好的值被覆盖掉。

通常解决碰撞的做法用一个链来存储，此时数组看起来是这样的：

```swift
buckets:
+-----+
|  0  |
+-----+     +----------------------------+
|  1  |---> | hobbies: Programming Swift |
+-----+     +----------------------------+
|  2  |
+-----+     +------------------+     +----------------+
|  3  |---> | firstName: Steve |---> | lastName: Jobs |
+-----+     +------------------+     +----------------+
|  4  |
+-----+
```

引入链后，数组不在直接存储键值对，而是把键值对列表作为自己的元素。通常我们称该数组的元素为 *bucket（桶）* ，把这个列表称为 *chains（链）*。（译者：有点像冬天做的腊肠，一节一节的刚做好，装在桶里）

如果我们通过下面的表达式去从哈希表中读区一个对象，

```swift
let x = hashTable["lastName"]
```
它首先会对`"lastName"` 进行哈希，从而得到索引 3。进而得到存储着目标值的链（腊肠），沿着链逐个进行字符串对比，查找`"lastName"`键，直到该链的最后一个元素，发现正好是我们要找的键，在将其对应的值返回。（顺着撸，找到心仪的那一节腊肠）

通常这样的链是由链表或者另一个数组来实现的。由于链上元素顺序无关紧要，所以你也可以把它理解为一个集合（Set）。

链要尽可能短，否则查找过程将变的耗时。理想状态，没有链是最好的，但实际上由于碰撞的存在，这种情况很难出现。不过你可以通过改进哈希算法，让哈希表有拥有更多的 buckets（桶）。

> **注意:** 链的一个替代方案是 "open addressing（开放地址？）"。它的思路是，如果一个数组索引已经存在，那么我们就把元素放到下一个未占用的索引，当然这种方式也有其自身的优缺点。
> 
## 上码

我们来看看 Swift 中哈希表的一个基本实现，之后我们会一步一步地构建它。

```swift
public struct HashTable<Key: Hashable, Value> {
private typealias Element = (key: Key, value: Value)
private typealias Bucket = [Element]
private var buckets: [Bucket]

private(set) public var count = 0

public var isEmpty: Bool { return count == 0 }

public init(capacity: Int) {
assert(capacity > 0)
buckets = Array<Bucket>(repeatElement([], count: capacity))
}
```

`HashTable`是一个通用容器，它有包含两个类型 `Key` (必须是 `Hashable（可哈希的）`) 和 `Value`。此外还定义了另外两种类型：用于在链中存储键值的`Element` , 和一个存储这些`Elements`的 `Bucket`数组。

主数组被命名为 `buckets`。其容量是固定的，所以其容量有初始化方法 `init(capacity)`确定。此外我们还通过 `count`来记录有多少对象被存到哈希表中。

创建一个新哈希表的范例：

```swift
var hashTable = HashTable<String, String>(capacity: 5)
```
此时这个哈希表还什么都不能做，接下来我们来添加一些方法，首先，添加一个用于计算键对应索引的函数：

```swift
private func index(forKey key: Key) -> Int {
return abs(key.hashValue) % buckets.count
}
```

该算法前面已经介绍过了，对键的哈希值取绝对值，再针对数组的大小进行取模运算，这里把它作为一个函数，是为了方便后续很多地方的使用。

哈希表或字典的四种常用方法：

- 增加一个元素
- 查询一个元素
- 更新一个已存在的元素
- 删除一个元素

对应的语法如下:

```swift
hashTable["firstName"] = "Steve"   // insert（增）
let x = hashTable["firstName"]     // lookup（查）
hashTable["firstName"] = "Tim"     // update（改）
hashTable["firstName"] = nil       // delete（删）
```

这四种操作我们都可以通过 `下标` 来实现：

```swift
public subscript(key: Key) -> Value? {
get {
return value(forKey: key)
}
set {
if let value = newValue {
updateValue(value, forKey: key)
} else {
removeValue(forKey: key)
}
}
}
```

它调用了3个帮手来完整实际的工作，我们来看看 `value(forKey:)` 是如何读从哈希表中读区一个对象的。

```swift
public func value(forKey key: Key) -> Value? {
let index = self.index(forKey: key)
for element in buckets[index] {
if element.key == key {
return element.value
}
}
return nil  // key not in hash table
}
```
首先通过`index(forKey:)` 方法将键转换为数组索引。这就给了我们一个 bucket（桶）的编号，但是如果存在碰撞，那么这个 bucket 就可能被不只一个键所占用，所以`value(forKey:)` 方法沿着该 bucket 中的链进行循环遍历查找，直到找到该键，并返回其对应的值，否则返回 `nil` 。

下面的代码展示如何通过 `updateValue(_:forKey:)`方法添加一个新元素或者更新一个已有元素，相对复杂一些：

```swift
public mutating func updateValue(_ value: Value, forKey key: Key) -> Value? {
let index = self.index(forKey: key)

// Do we already have this key in the bucket?
for (i, element) in buckets[index].enumerated() {
if element.key == key {
let oldValue = element.value
buckets[index][i].value = value
return oldValue
}
}

// This key isn't in the bucket yet; add it to the chain.
buckets[index].append((key: key, value: value))
count += 1
return nil
}
```

同上，还是先将键转换为对应的数组索引，以找到对应的 bucket（桶）。之后沿着 chain（链）查找，找到就更新它，如果没找到，就在链的末尾增加一个新的元素。

如你所见，保持链尽量的短是多么重要（让哈希表足够大）。否则就会在 `for`...`in` 循环中耗费大量时间，如此哈希表的时间复杂度就不再是 **O(1)** 而更加接近 **O(n)**。

删除类似也要对链进行遍历：

```swift
public mutating func removeValue(forKey key: Key) -> Value? {
let index = self.index(forKey: key)

// Find the element in the bucket's chain and remove it.
for (i, element) in buckets[index].enumerated() {
if element.key == key {
buckets[index].remove(at: i)
count -= 1
return element.value
}
}
return nil  // key not in hash table
}
```

这样一个哈希表的基础功能就都实现了。工作原理都非常相似：将键的哈希值转换为数组索引，根据索引找到对应的bucket，然后在沿着bucket存储的链循环遍历到目标元素，进而执行操作。

试着在 palyground 中把玩一下，应该跟 Swift 提供的 `Dictionary` 差不多。

## 哈希表扩容

这一版的`HashTable`的大小或容量是固定的。如果有更多的元素需要存到表中，那么选择一个大于元素个数的值作为大小显然更优。

哈希表的 *load factor（装载因子，装载率）* 是指，最大容量使用的百分比。如果有一个有 5 个 bucket的哈希表，已经存放了3个元素，那么它的装载因子就是`3/5 = 60%`。

如果哈希表比较小，那么链就会比较长，load factor（装载因子，装载率）就会大于 1，这样效率会比较差。

当load factor（装载因子，装载率）大于 75%是，就应该对哈希表进行扩容，这部分代码就留给读者作为练习了，但是请注意，当 bucket 数组变大时，随之也会改变键与索引之间的映射关系！这就要求你在扩容后必须重新添加一遍所有元素。

## 接下来？

这里提供的哈希表非常简单，如果通过 `SequenceType` 与 SWift标准库相结合，那么它将变的更加高效。

*由 Matthijs Hollemans 为 Swift Algorithm Club（Swift算法社区） 撰写*

*由 William Han  翻译并发表于 Github*



