## 在哈希映射中存储键和关联值

我们常见集合中的最后一个是哈希映射。类型 `HashMap<K, V>` 使用 _哈希函数_ 存储类型 `K` 的键到类型 `V` 的值的映射，该函数决定了它如何将这些键和值放入内存。许多编程语言支持这种数据结构，但它们经常使用不同的名称，例如 _hash_、_map_、_object_、_hash table_、_dictionary_ 或 _associative array_，仅举几例。

当你想要通过使用键（可以是任何类型）而不是使用索引（就像你可以使用向量一样）来查找数据时，哈希映射很有用。例如，在游戏中，你可以在哈希映射中跟踪每个团队的分数，其中每个键是团队名称，值是每个团队的分数。给定团队名称，你可以检索其分数。

我们将在本节中介绍哈希映射的基本 API，但标准库在 `HashMap<K, V>` 上定义的函数中隐藏了更多好东西。一如既往，请查看标准库文档以获取更多信息。

### 创建新哈希映射

创建空哈希映射的一种方法是使用 `new` 并使用 `insert` 添加元素。在代码清单 8-20 中，我们跟踪两个团队的分数，它们的名称是 _Blue_ 和 _Yellow_。Blue 团队以 10 分开始，Yellow 团队以 50 分开始。

<Listing number="8-20" caption="创建新哈希映射并插入一些键和值">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-20/src/main.rs:here}}
```

</Listing>

注意，我们需要首先从标准库的集合部分 `use` `HashMap`。在我们的三个常见集合中，这是最不常用的，所以它不包含在预导入模块中自动引入作用域的功能中。哈希映射从标准库得到的支持也较少；例如，没有内置宏来构造它们。

就像向量一样，哈希映射将其数据存储在堆上。这个 `HashMap` 的键类型是 `String`，值类型是 `i32`。就像向量一样，哈希映射是同质的：所有键必须具有相同的类型，所有值必须具有相同的类型。

### 访问哈希映射中的值

我们可以通过向 `get` 方法提供其键来从哈希映射中获取值，如代码清单 8-21 所示。

<Listing number="8-21" caption="访问存储在哈希映射中的 Blue 团队的分数">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-21/src/main.rs:here}}
```

</Listing>

这里，`score` 将具有与 Blue 团队关联的值，结果将是 `10`。`get` 方法返回 `Option<&V>`；如果哈希映射中没有该键的值，`get` 将返回 `None`。此程序通过调用 `copied` 获取 `Option<i32>` 而不是 `Option<&i32>` 来处理 `Option`，然后使用 `unwrap_or` 如果 `scores` 没有该键的条目，则将 `score` 设置为零。

我们可以使用 `for` 循环以与向量类似的方式迭代哈希映射中的每个键值对：

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-03-iterate-over-hashmap/src/main.rs:here}}
```

这段代码将以任意顺序打印每对：

```text
Yellow: 50
Blue: 10
```

<!-- Old headings. Do not remove or links may break. -->

<a id="hash-maps-and-ownership"></a>

### 管理哈希映射中的所有权

对于实现 `Copy` trait 的类型，如 `i32`，值被复制到哈希映射中。对于拥有的值，如 `String`，值将被移动，哈希映射将成为这些值的所有者，如代码清单 8-22 所示。

<Listing number="8-22" caption="显示键和值在插入后由哈希映射拥有">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-22/src/main.rs:here}}
```

</Listing>

在通过调用 `insert` 将它们移动到哈希映射后，我们无法使用变量 `field_name` 和 `field_value`。

如果我们将值的引用插入到哈希映射中，值不会被移动到哈希映射中。引用指向的值必须至少在哈希映射有效的时间内有效。我们将在第 10 章的["使用生命周期验证引用"][validating-references-with-lifetimes]<!-- ignore -->中更详细地讨论这些问题。

### 更新哈希映射

尽管键值对的数量是可增长的，但每个唯一键一次只能有一个与其关联的值（但反之则不然：例如，Blue 团队和 Yellow 团队都可以在 `scores` 哈希映射中存储值 `10`）。

当你想要更改哈希映射中的数据时，你必须决定如何处理键已分配值的情况。你可以用新值替换旧值，完全忽略旧值。你可以保留旧值并忽略新值，仅在键 _没有_ 值时才添加新值。或者你可以组合旧值和新值。让我们看看如何做到这些！

#### 覆盖值

如果我们将键和值插入到哈希映射中，然后用不同的值插入相同的键，与该键关联的值将被替换。尽管代码清单 8-23 中的代码调用 `insert` 两次，哈希映射将只包含一个键值对，因为我们在两次调用中都为 Blue 团队的键插入值。

<Listing number="8-23" caption="替换与特定键一起存储的值">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-23/src/main.rs:here}}
```

</Listing>

这段代码将打印 `{"Blue": 25}`。原始值 `10` 已被覆盖。

<!-- Old headings. Do not remove or links may break. -->

<a id="only-inserting-a-value-if-the-key-has-no-value"></a>

#### 仅在键不存在时添加键和值

检查特定键是否已在哈希映射中存在值，然后采取以下操作是很常见的：如果键确实存在于哈希映射中，现有值应保持原样；如果键不存在，则插入它和它的值。

哈希映射有一个用于此目的的特殊 API，称为 `entry`，它接受你想要检查的键作为参数。`entry` 方法的返回值是一个名为 `Entry` 的枚举，它表示可能存在也可能不存在的值。假设我们想检查 Yellow 团队的键是否有与其关联的值。如果没有，我们想插入值 `50`，Blue 团队也是如此。使用 `entry` API，代码看起来像代码清单 8-24。

<Listing number="8-24" caption="使用 `entry` 方法仅在键还没有值时插入">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-24/src/main.rs:here}}
```

</Listing>

`Entry` 上的 `or_insert` 方法定义为如果该键存在，则返回对相应 `Entry` 键值的可变引用，如果不存在，它将参数作为该键的新值插入并返回对新值的可变引用。这种技术比我们自己编写逻辑要干净得多，此外，它与借用检查器配合得更好。

运行代码清单 8-24 中的代码将打印 `{"Yellow": 50, "Blue": 10}`。第一次调用 `entry` 将为 Yellow 团队插入键，值为 `50`，因为 Yellow 团队还没有值。第二次调用 `entry` 不会更改哈希映射，因为 Blue 团队已经有值 `10`。

#### 基于旧值更新值

哈希映射的另一个常见用例是查找键的值，然后基于旧值更新它。例如，代码清单 8-25 显示了计算每个单词在文本中出现次数的代码。我们使用一个以单词为键的哈希映射，并递增该值以跟踪我们看到该单词的次数。如果这是我们第一次看到单词，我们将首先插入值 `0`。

<Listing number="8-25" caption="使用存储单词和计数的哈希映射计算单词出现次数">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-25/src/main.rs:here}}
```

</Listing>

这段代码将打印 `{"world": 2, "hello": 1, "wonderful": 1}`。你可能会看到相同的键值对以不同的顺序打印：回想["访问哈希映射中的值"][access]<!-- ignore -->，迭代哈希映射以任意顺序发生。

`split_whitespace` 方法返回 `text` 中值的子切片的迭代器，由空白分隔。`or_insert` 方法返回对指定键值的可变引用（`&mut V`）。这里，我们将该可变引用存储在 `count` 变量中，因此为了分配给该值，我们必须首先使用星号（`*`）解引用 `count`。可变引用在 `for` 循环结束时超出作用域，所以所有这些更改都是安全的，并且允许借用规则。

### 哈希函数

默认情况下，`HashMap` 使用称为 _SipHash_ 的哈希函数，可以提供对涉及哈希表的拒绝服务（DoS）攻击的抵抗力[^siphash]<!-- ignore -->。这不是最快的可用哈希算法，但为了更好的安全性而降低性能的权衡是值得的。如果你分析代码并发现默认哈希函数对你的目的来说太慢，你可以通过指定不同的 hasher 来切换到另一个函数。_hasher_ 是实现 `BuildHasher` trait 的类型。我们将在[第 10 章][traits]<!-- ignore -->中讨论 trait 以及如何实现它们。你不一定需要从头实现自己的 hasher；[crates.io](https://crates.io/)<!-- ignore -->上有其他 Rust 用户共享的库，这些库提供了实现许多常见哈希算法的 hasher。

[^siphash]: [https://en.wikipedia.org/wiki/SipHash](https://en.wikipedia.org/wiki/SipHash)

## 总结

向量、字符串和哈希映射将在你需要存储、访问和修改数据的程序中提供大量必要的功能。以下是一些你现在应该能够解决的练习：

1. 给定一个整数列表，使用向量并返回列表的中位数（排序后，中间位置的值）和众数（出现最频繁的值；哈希映射在这里会很有帮助）。
1. 将字符串转换为 Pig Latin。每个单词的第一个辅音移动到单词的末尾并添加 _ay_，所以 _first_ 变成 _irst-fay_。以元音开头的单词在末尾添加 _hay_（_apple_ 变成 _apple-hay_）。记住 UTF-8 编码的细节！
1. 使用哈希映射和向量，创建一个文本界面，允许用户将员工姓名添加到公司中的部门；例如，"Add Sally to Engineering" 或 "Add Amir to Sales"。然后，让用户按字母顺序检索部门中的所有人员或公司中按部门的所有人员列表。

标准库 API 文档描述了向量、字符串和哈希映射具有的方法，这些方法对这些练习很有帮助！

我们正在进入更复杂的程序，其中操作可能会失败，所以这是讨论错误处理的最佳时机。我们接下来会这样做！

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[access]: #accessing-values-in-a-hash-map
[traits]: ch10-02-traits.html
