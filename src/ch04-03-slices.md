## 切片类型

_切片_让您引用[集合](ch08-00-common-collections.md)<!-- ignore -->中连续的元素序列。切片是一种引用，所以它没有所有权。

这里有一个小编程问题：编写一个函数，该函数接受一个由空格分隔的单词字符串，并返回它在该字符串中找到的第一个单词。如果函数在字符串中找不到空格，整个字符串必须是一个单词，因此应该返回整个字符串。

> 注意：为了介绍切片的目的，我们在本节中假设仅使用 ASCII；关于 UTF-8 处理的更详细讨论在第 8 章的["使用字符串存储 UTF-8 编码的文本"][strings]<!-- ignore -->部分中。

让我们不使用切片来编写此函数的签名，以了解切片将解决的问题：

```rust,ignore
fn first_word(s: &String) -> ?
```

`first_word` 函数的参数类型为 `&String`。我们不需要所有权，所以这很好。（在惯用的 Rust 中，函数不会获取其参数的所有权，除非它们需要，当我们继续前进时，这样做的原因会变得清楚。）但我们应该返回什么？我们真的没有方法来谈论字符串的*部分*。但是，我们可以返回单词结束的索引，由空格指示。让我们尝试一下，如清单 4-7 所示。

<Listing number="4-7" file-name="src/main.rs" caption="返回 `String` 参数的字节索引值的 `first_word` 函数">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:here}}
```

</Listing>

因为我们需要逐个元素地遍历 `String` 并检查值是否为空格，我们将使用 `as_bytes` 方法将 `String` 转换为字节数组。

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:as_bytes}}
```

接下来，我们使用 `iter` 方法在字节数组上创建一个迭代器：

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:iter}}
```

我们将在[第 13 章][ch13]<!-- ignore -->中更详细地讨论迭代器。现在，知道 `iter` 是一个返回集合中每个元素的方法，`enumerate` 包装 `iter` 的结果并将每个元素作为元组的一部分返回。从 `enumerate` 返回的元组的第一个元素是索引，第二个元素是对元素的引用。这比我们自己计算索引更方便一点。

因为 `enumerate` 方法返回元组，我们可以使用模式来解构该元组。我们将在[第 6 章][ch6]<!-- ignore -->中更详细地讨论模式。在 `for` 循环中，我们指定一个模式，在元组中 `i` 用于索引，`&item` 用于元组中的单个字节。因为我们从 `.iter().enumerate()` 获取对元素的引用，所以我们在模式中使用 `&`。

在 `for` 循环内部，我们通过使用字节字面量语法搜索表示空格的字节。如果我们找到空格，我们返回位置。否则，我们使用 `s.len()` 返回字符串的长度。

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:inside_for}}
```

我们现在有一种方法来找出字符串中第一个单词结束的索引，但有一个问题。我们单独返回 `usize`，但它只在 `&String` 的上下文中是有意义的数字。换句话说，因为它是一个与 `String` 分离的值，不能保证它将来仍然有效。考虑清单 4-8 中使用清单 4-7 中的 `first_word` 函数的程序。

<Listing number="4-8" file-name="src/main.rs" caption="存储调用 `first_word` 函数的结果，然后更改 `String` 内容">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-08/src/main.rs:here}}
```

</Listing>

此程序编译时没有任何错误，如果我们 after calling `s.clear()` 使用 `word`，它也会这样做。因为 `word` 根本没有连接到 `s` 的状态，`word` 仍然包含值 `5`。我们可以使用该值 `5` 和变量 `s` 来尝试提取第一个单词，但这将是一个错误，因为 `s` 的内容自我们将 `5` 保存在 `word` 中以来已经改变。

不得不担心 `word` 中的索引与 `s` 中的数据不同步是繁琐且容易出错的！如果我们编写 `second_word` 函数，管理这些索引会更加脆弱。它的签名必须看起来像这样：

```rust,ignore
fn second_word(s: &String) -> (usize, usize) {
```

现在我们正在跟踪开始_和_结束索引，我们还有更多从特定状态的数据计算的值，但根本没有绑定到该状态。我们有三个不相关的变量在浮动，需要保持同步。

幸运的是，Rust 有一个解决此问题的方案：字符串切片。

### 字符串切片

_字符串切片_是对 `String` 的连续元素序列的引用，它看起来像这样：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-17-slice/src/main.rs:here}}
```

而不是对整个 `String` 的引用，`hello` 是对 `String` 的一部分的引用，在额外的 `[0..5]` 位中指定。我们通过在方括号内指定范围来创建切片，通过指定 `[starting_index..ending_index]`，其中 _`starting_index`_ 是切片中的第一个位置，_`ending_index`_ 是切片中最后一个位置多一个。在内部，切片数据结构存储起始位置和切片的长度，对应于 _`ending_index`_ 减去 _`starting_index`_。所以，在 `let world = &s[6..11];` 的情况下，`world` 将是一个切片，包含指向 `s` 索引 6 处的字节的指针，长度为 `5`。

图 4-7 在图表中显示了这一点。

<img alt="三个表：一个表表示 s 的栈数据，它指向堆上字符串数据表中索引 0 处的字节。第三个表表示切片 world 的栈数据，长度为 5，指向堆数据表的字节 6。" src="img/trpl04-07.svg" class="center" style="width: 50%;" />

<span class="caption">图 4-7：引用 `String` 一部分的字符串切片</span>

使用 Rust 的 `..` 范围语法，如果您想从索引 0 开始，可以删除两个点之前的值。换句话说，这些是相等的：

```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

同样，如果您的切片包括 `String` 的最后一个字节，您可以删除尾随数字。这意味着这些是相等的：

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

您也可以删除两个值以获取整个字符串的切片。所以，这些是相等的：

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> 注意：字符串切片范围索引必须发生在有效的 UTF-8 字符边界处。如果您尝试在多字节字符的中间创建字符串切片，您的程序将因错误而退出。

考虑到所有这些信息，让我们重写 `first_word` 以返回切片。表示"字符串切片"的类型写为 `&str`：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-18-first-word-slice/src/main.rs:here}}
```

</Listing>

我们以与清单 4-7 中相同的方式获取单词结束的索引，通过查找空格的第一次出现。当我们找到空格时，我们使用字符串的开始和空格的索引作为起始和结束索引来返回字符串切片。

现在当我们调用 `first_word` 时，我们得到一个与底层数据相关的单个值。该值由对切片起点的引用和切片中的元素数量组成。

返回切片也适用于 `second_word` 函数：

```rust,ignore
fn second_word(s: &String) -> &str {
```

我们现在有一个更直接的 API，很难出错，因为编译器将确保对 `String` 的引用保持有效。还记得清单 4-8 中程序中的错误吗，当我们获得第一个单词结束的索引，但然后清除了字符串，所以我们的索引无效？该代码在逻辑上不正确，但没有显示任何立即错误。如果我们继续尝试将第一个单词索引与清空的字符串一起使用，问题会在稍后出现。切片使此错误不可能发生，并让我们更早地知道代码有问题。使用 `first_word` 的切片版本将抛出编译时错误：

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/src/main.rs:here}}
```

</Listing>

这是编译器错误：

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/output.txt}}
```

从借用规则回想，如果我们对某物有不可变引用，我们不能同时获取可变引用。因为 `clear` 需要截断 `String`，它需要获取可变引用。在调用 `clear` 之后的 `println!` 使用 `word` 中的引用，所以不可变引用必须在该点仍然有效。Rust 不允许 `clear` 中的可变引用和 `word` 中的不可变引用同时存在，编译失败。Rust 不仅使我们的 API 更易于使用，而且在编译时消除了整个类别的错误！

<!-- Old headings. Do not remove or links may break. -->

<a id="string-literals-are-slices"></a>

#### 字符串字面量作为切片

回想一下，我们谈论过字符串字面量存储在二进制文件中。现在我们知道切片了，我们可以正确理解字符串字面量：

```rust
let s = "Hello, world!";
```

这里 `s` 的类型是 `&str`：它是一个指向二进制文件中特定点的切片。这也是为什么字符串字面量是不可变的；`&str` 是不可变引用。

#### 字符串切片作为参数

知道您可以获取字面量和 `String` 值的切片，这使我们能够对 `first_word` 进行另一个改进，那就是它的签名：

```rust,ignore
fn first_word(s: &String) -> &str {
```

更有经验的 Rustacean 会编写清单 4-9 中显示的签名，因为它允许我们在 `&String` 值和 `&str` 值上使用相同的函数。

<Listing number="4-9" caption="通过使用字符串切片作为 `s` 参数的类型来改进 `first_word` 函数">

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:here}}
```

</Listing>

如果我们有字符串切片，我们可以直接传递它。如果我们有 `String`，我们可以传递 `String` 的切片或对 `String` 的引用。这种灵活性利用了解引用强制转换，我们将在第 15 章的["在函数和方法中使用解引用强制转换"][deref-coercions]<!--
ignore -->部分中介绍。

定义函数接受字符串切片而不是对 `String` 的引用，使我们的 API 更通用和有用，而不会丢失任何功能：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:usage}}
```

</Listing>

### 其他切片

字符串切片，正如您可能想象的那样，特定于字符串。但也有更通用的切片类型。考虑这个数组：

```rust
let a = [1, 2, 3, 4, 5];
```

就像我们可能想要引用字符串的一部分一样，我们可能想要引用数组的一部分。我们会这样做：

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

此切片的类型为 `&[i32]`。它的工作方式与字符串切片相同，通过存储对第一个元素的引用和长度。您将使用这种切片来处理所有其他类型的集合。我们将在第 8 章讨论向量时详细讨论这些集合。

## 总结

所有权、借用和切片的概念在编译时确保 Rust 程序中的内存安全。Rust 语言让您以与其他系统编程语言相同的方式控制内存使用。但拥有数据的拥有者在拥有者超出作用域时自动清理该数据，这意味着您不必编写和调试额外的代码来获得这种控制。

所有权影响 Rust 的许多其他部分的工作方式，所以我们将在本书的其余部分进一步讨论这些概念。让我们继续第 5 章，看看如何在 `struct` 中将数据片段组合在一起。

[ch13]: ch13-02-iterators.html
[ch6]: ch06-02-match.html#patterns-that-bind-to-values
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[deref-coercions]: ch15-02-deref.html#using-deref-coercions-in-functions-and-methods
