# 环形缓存器（Ring Buffer）

也被称为循环缓存器
Also known as a circular buffer.

[基于数组实现的队列](../Queue/)存在一个问题，队尾添加新对象很快，**O(1)**，但是队首移除对象则较慢，**O(n)**。之所以如此，是因为数组会将剩余的对象整体向前移动。
The problem with a [queue based on an array](../Queue/) is that adding new items to the back of the queue is fast, **O(1)**, but removing items from the front of the queue is slow, **O(n)**. Removing is slow because it requires the remaining array elements to be shifted in memory.

改进的方案之一就是用环形缓存器或循环缓存器去实现队列。概念上就是一个首尾相连的数组，这样就不用删除任何对象了，所有操作都是 **O(1)**。
A more efficient way to implement a queue is to use a ring buffer or circular buffer. This is an array that conceptually wraps around back to the beginning, so you never have to remove any items. All operations are **O(1)**.

其原理如下。假定我们有一个大小固定的数组，其可存储 5 个对象：
Here is how it works in principle. We have an array of a fixed size, say 5 items:

 	[    ,    ,    ,    ,     ]
	 r
	 w

起初，数组为空，并且读(`r`)和写(`r`)指针都位于起始位置（译者：注意看上图）。
Initially, the array is empty and the read (`r`) and write (`w`) pointers are at the beginning.

向数组中添加一些数据。如写入或者说"入队（enqueue）"一个数字 `123`：
Let's add some data to this array. We'll write, or "enqueue", the number `123`:

	[ 123,    ,    ,    ,     ]
	  r
	  ---> w

每次添加数据，写指针都会向前移动一步。我们再添加几个：
Each time we add data, the write pointer moves one step forward. Let's add a few more elements:

	[ 123, 456, 789, 666,     ]
	  r    
	       -------------> w

数组中仅剩一个空位，此时在继续添加数据前应用要读取一些数据。由于写指针相对读指针更靠前，所以是可以读的。读指针随着有效数据的读取而向前移动。
There is now one open spot left in the array, but rather than enqueuing another item the app decides to read some data. That's possible because the write pointer is ahead of the read pointer, meaning data is available for reading. The read pointer advances as we read the available data:

	[ 123, 456, 789, 666,     ]
	  ---> r              w

在向前读取 2 个对象：
Let's read two more items:

	[ 123, 456, 789, 666,     ]
	       --------> r    w

此时应用又突然要写入（入队）两个的对象， `333` 和 `555`：
Now the app decides it's time to write again and enqueues two more data items, `333` and `555`:

	[ 123, 456, 789, 666, 333 ]
	                 r    ---> w

糟了，写指针已经抵达数组末尾，没地方存储 `555`。怎么办？这时候就该环形存储器大展身手了：写指针返回数组对开端继续写入数据：
Whoops, the write pointer has reached the end of the array, so there is no more room for object `555`. What now? Well, this is why it's a circular buffer: we wrap the write pointer back to the beginning and write the remaining data:

	[ 555, 456, 789, 666, 333 ]
	  ---> w         r        

之后在读取剩下的 3 个对象，`666`, `333`, 和 `555`。
We can now read the remaining three items, `666`, `333`, and `555`.

	[ 555, 456, 789, 666, 333 ]
	       w         --------> r        

同理，读指针抵缓存末尾后也会转回来：
Of course, as the read pointer reaches the end of the buffer it also wraps around:

	[ 555, 456, 789, 666, 333 ]
	       w            
	  ---> r

由于此时读指针追上了写指针，所以缓冲又空了。
And now the buffer is empty again because the read pointer has caught up with the write pointer.

下面是（环形缓存器）的 Swift 简单实现：
Here is a very basic implementation in Swift:

```swift
public struct RingBuffer<T> {
  fileprivate var array: [T?]
  fileprivate var readIndex = 0
  fileprivate var writeIndex = 0

  public init(count: Int) {
    array = [T?](repeating: nil, count: count)
  }

  public mutating func write(_ element: T) -> Bool {
    if !isFull {
      array[writeIndex % array.count] = element
      writeIndex += 1
      return true
    } else {
      return false
    }
  }

  public mutating func read() -> T? {
    if !isEmpty {
      let element = array[readIndex % array.count]
      readIndex += 1
      return element
    } else {
      return nil
    }
  }

  fileprivate var availableSpaceForReading: Int {
    return writeIndex - readIndex
  }

  public var isEmpty: Bool {
    return availableSpaceForReading == 0
  }

  fileprivate var availableSpaceForWriting: Int {
    return array.count - availableSpaceForReading
  }

  public var isFull: Bool {
    return availableSpaceForWriting == 0
  }
}
```

 `RingBuffer` 用一个数组作为其存储对象。同时还有 `readIndex` 和 `writeIndex` 两个索引作为数组中的“（读写）指针”。`write()` 可以将一个对象添加数组中 `writeIndex` 索引的位置，而 `read()`方法则可以返回位于 `readIndex` 索引位置的对象。
The `RingBuffer` object has an array for the actual storage of the data, and `readIndex` and `writeIndex` variables for the "pointers" into the array. The `write()` function puts the new element into the array at the `writeIndex`, and the `read()` function returns the element at the `readIndex`.

但是等等，不是说可以循环的嘛？实现该功能有几种方案，这里我选了一个相对争议较小的。该实现中，`writeIndex` 和 `readIndex` 会始终增加，不会变回（初始值）。取而代之的是通过以下方法来找到数组中真正的索引位置：
But hold up, you say, how does this wrapping around work? There are several ways to accomplish this and I chose a slightly controversial one. In this implementation, the `writeIndex` and `readIndex` always increment and never actually wrap around. Instead, we do the following to find the actual index into the array:

```swift
array[writeIndex % array.count]
```

以及:

```swift
array[readIndex % array.count]
```

也就是说，通过对读/写索引除以数组大小的结果进行取模或者说取余（来得到真正的索引）。
In other words, we take the modulo (or the remainder) of the read index and write index divided by the size of the underlying array.

该方案的争议就在于 `writeIndex` 和 `readIndex` 会始终增长，进而导致超出整型的范围而引起应用崩溃。然而，只要简单估算一下就可以打消这种顾虑。
The reason this is a bit controversial is that `writeIndex` and `readIndex` always increment, so in theory these values could become too large to fit into an integer and the app will crash. However, a quick back-of-the-napkin calculation should take away those fears.

`writeIndex` 和 `readIndex` 都是 64 位整型。假定他们每秒增长1000次，那么由此可以推算一年才需要 `ceil(log_2(365 * 24 * 60 * 60 * 1000)) = 35` 比特。还剩 28 个比特（译者：注意这里是有符号整型），也就是说还需要大约 2^28 年才会发生。已经足够久了。:-)
Both `writeIndex` and `readIndex` are 64-bit integers. If we assume that they are incremented 1000 times per second, which is a lot, then doing this continuously for one year requires `ceil(log_2(365 * 24 * 60 * 60 * 1000)) = 35` bits. That leaves 28 bits to spare, so that should give you about 2^28 years before running into problems. That's a long time. :-)

要尝试换行缓存器，将一下代码拷贝到 palyground 中，然后像上面的例子那样操作：
To play with this ring buffer, copy the code to a playground and do the following to mimic the example above:

```swift
var buffer = RingBuffer<Int>(count: 5)

buffer.write(123)
buffer.write(456)
buffer.write(789)
buffer.write(666)

buffer.read()   // 123
buffer.read()   // 456
buffer.read()   // 789

buffer.write(333)
buffer.write(555)

buffer.read()   // 666
buffer.read()   // 333
buffer.read()   // 555
buffer.read()   // nil
```

如你所见，通过环形缓存器可以实现一个更优的队列，但它也有自己的缺点：难以重设队列的大小。但如果你恰好需要一个固定大小的队列，那么它就是你最高的选择。
You've seen that a ring buffer can make a more optimal queue but it also has a disadvantage: the wrapping makes it tricky to resize the queue. But if a fixed-size queue is suitable for your purposes, then you're golden.

当数据写入速率与数据读取速率不同时，环形缓存器也是个非常不错的选择。这种情况通常发生在文件或者网络 I/O（过程中）。此外，缓存缓冲器在（解决）高优先级线程与低优先级线程，以及其它或者系统线程间通讯时，也是一个相当好的方案。
A ring buffer is also very useful for when a producer of data writes into the array at a different rate than the consumer of the data reads it. This happens often with file or network I/O. Ring buffers are also the preferred way of communicating between high priority threads (such as an audio rendering callback) and other, slower, parts of the system.

这里给出的实现不是线程安全的。它仅用来展示环形缓存器是如何工作的。要实现线程安全，一个相对直接的方案就是：通过`OSAtomicIncrement64()` 来改变读写指针，以保证同一时间只有读或写。
The implementation given here is not thread-safe. It only serves as an example of how a ring buffer works. That said, it should be fairly straightforward to make it thread-safe for a single reader and single writer by using `OSAtomicIncrement64()` to change the read and write pointers.

有一个创建真正高效环形缓冲器的技巧，就是通过操作系统的虚拟内存在不同的内存页上映射出一个相同的缓冲。有点疯狂，但如果你的高效执行环境真的需要一个环形缓存器，那么值得一试。
A cool trick to make a really fast ring buffer is to use the operating system's virtual memory system to map the same buffer onto different memory pages. Crazy stuff but worth looking into if you need to use a ring buffer in a high performance environment.

*Written for Swift Algorithm Club by Matthijs Hollemans*

*由 Matthijs Hollemans 首发于 Swift 算法社区*

*由 William Han 翻译*