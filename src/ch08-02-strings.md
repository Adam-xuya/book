## 使用字符串存储 UTF-8 编码的文本

我们在第 4 章讨论过字符串，但现在我们将更深入地了解它们。新的 Rustaceans 经常在字符串上遇到困难，原因有三个：Rust 倾向于暴露可能的错误、字符串是比许多程序员认为的更复杂的数据结构，以及 UTF-8。这些因素结合起来，当你来自其他编程语言时，可能会显得困难。

我们在集合的上下文中讨论字符串，因为字符串被实现为字节的集合，加上一些方法，当这些字节被解释为文本时提供有用的功能。在本节中，我们将讨论 `String` 上每个集合类型都有的操作，例如创建、更新和读取。我们还将讨论 `String` 与其他集合的不同之处，即如何索引到 `String` 由于人和计算机解释 `String` 数据的方式不同而变得复杂。

<!-- Old headings. Do not remove or links may break. -->

<a id="what-is-a-string"></a>

### 定义字符串

我们首先定义术语 _字符串_ 的含义。Rust 在核心语言中只有一种字符串类型，即字符串切片 `str`，通常以其借用形式 `&str` 出现。在第 4 章中，我们讨论了字符串切片，它们是对存储在其他地方的 UTF-8 编码字符串数据的引用。例如，字符串字面量存储在程序的二进制文件中，因此是字符串切片。

`String` 类型由 Rust 的标准库提供，而不是编码到核心语言中，是一个可增长、可变、拥有的 UTF-8 编码字符串类型。当 Rustaceans 在 Rust 中提及"字符串"时，他们可能指的是 `String` 或字符串切片 `&str` 类型，而不仅仅是这些类型之一。尽管本节主要关于 `String`，但两种类型在 Rust 的标准库中都大量使用，并且 `String` 和字符串切片都是 UTF-8 编码的。

### 创建新字符串

许多与 `Vec<T>` 相同的操作也可用于 `String`，因为 `String` 实际上是作为字节向量的包装器实现的，具有一些额外的保证、限制和功能。与 `Vec<T>` 和 `String` 工作方式相同的函数示例是创建实例的 `new` 函数，如代码清单 8-11 所示。

<Listing number="8-11" caption="创建一个新的空 `String`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-11/src/main.rs:here}}
```

</Listing>

这行创建一个名为 `s` 的新空字符串，然后我们可以向其加载数据。通常，我们会有一些初始数据，我们希望用它来开始字符串。为此，我们使用 `to_string` 方法，该方法可用于任何实现 `Display` trait 的类型，就像字符串字面量一样。代码清单 8-12 显示了两个示例。

<Listing number="8-12" caption="使用 `to_string` 方法从字符串字面量创建 `String`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-12/src/main.rs:here}}
```

</Listing>

这段代码创建一个包含 `initial contents` 的字符串。

我们也可以使用函数 `String::from` 从字符串字面量创建 `String`。代码清单 8-13 中的代码等效于使用 `to_string` 的代码清单 8-12 中的代码。

<Listing number="8-13" caption="使用 `String::from` 函数从字符串字面量创建 `String`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-13/src/main.rs:here}}
```

</Listing>

因为字符串用于许多事情，我们可以为字符串使用许多不同的通用 API，为我们提供了很多选项。其中一些可能看起来是冗余的，但它们都有自己的位置！在这种情况下，`String::from` 和 `to_string` 做同样的事情，所以你选择哪一个是一个风格和可读性的问题。

记住字符串是 UTF-8 编码的，所以我们可以在其中包含任何正确编码的数据，如代码清单 8-14 所示。

<Listing number="8-14" caption="在字符串中存储不同语言的问候语">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:here}}
```

</Listing>

所有这些都是有效的 `String` 值。

### 更新字符串

`String` 可以增长大小，其内容可以更改，就像 `Vec<T>` 的内容一样，如果你向其中推送更多数据。此外，你可以方便地使用 `+` 操作符或 `format!` 宏来连接 `String` 值。

<!-- Old headings. Do not remove or links may break. -->

<a id="appending-to-a-string-with-push_str-and-push"></a>

#### 使用 `push_str` 或 `push` 追加

我们可以使用 `push_str` 方法追加字符串切片来增长 `String`，如代码清单 8-15 所示。

<Listing number="8-15" caption="使用 `push_str` 方法将字符串切片追加到 `String`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-15/src/main.rs:here}}
```

</Listing>

在这两行之后，`s` 将包含 `foobar`。`push_str` 方法接受字符串切片，因为我们不一定想要获取参数的所有权。例如，在代码清单 8-16 的代码中，我们希望能够在将其内容追加到 `s1` 后使用 `s2`。

<Listing number="8-16" caption="在将其内容追加到 `String` 后使用字符串切片">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-16/src/main.rs:here}}
```

</Listing>

如果 `push_str` 方法获取了 `s2` 的所有权，我们将无法在最后一行打印其值。但是，这段代码按我们预期的方式工作！

`push` 方法接受单个字符作为参数并将其添加到 `String`。代码清单 8-17 使用 `push` 方法将字母 _l_ 添加到 `String`。

<Listing number="8-17" caption="使用 `push` 向 `String` 值添加一个字符">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-17/src/main.rs:here}}
```

</Listing>

结果，`s` 将包含 `lol`。

<!-- Old headings. Do not remove or links may break. -->

<a id="concatenation-with-the--operator-or-the-format-macro"></a>

#### 使用 `+` 或 `format!` 连接

通常，你会想要组合两个现有的字符串。一种方法是使用 `+` 操作符，如代码清单 8-18 所示。

<Listing number="8-18" caption="使用 `+` 操作符将两个 `String` 值组合成一个新的 `String` 值">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-18/src/main.rs:here}}
```

</Listing>

字符串 `s3` 将包含 `Hello, world!`。`s1` 在加法后不再有效的原因，以及我们使用 `s2` 的引用的原因，与使用 `+` 操作符时调用的方法的签名有关。`+` 操作符使用 `add` 方法，其签名看起来像这样：

```rust,ignore
fn add(self, s: &str) -> String {
```

在标准库中，你会看到 `add` 使用泛型和关联类型定义。在这里，我们代入了具体类型，这是当我们使用 `String` 值调用此方法时发生的情况。我们将在第 10 章讨论泛型。这个签名为我们提供了理解 `+` 操作符的棘手部分所需的线索。

首先，`s2` 有一个 `&`，意味着我们将第二个字符串的引用添加到第一个字符串。这是因为 `add` 函数中的 `s` 参数：我们只能将字符串切片添加到 `String`；我们不能将两个 `String` 值加在一起。但是等等——`&s2` 的类型是 `&String`，而不是 `add` 的第二个参数中指定的 `&str`。那么，为什么代码清单 8-18 可以编译？

我们能够在 `add` 调用中使用 `&s2` 的原因是编译器可以将 `&String` 参数强制转换为 `&str`。当我们调用 `add` 方法时，Rust 使用解引用强制转换，这里将 `&s2` 转换为 `&s2[..]`。我们将在第 15 章更深入地讨论解引用强制转换。因为 `add` 不获取 `s` 参数的所有权，`s2` 在此操作后仍然是一个有效的 `String`。

其次，我们可以在签名中看到 `add` 获取 `self` 的所有权，因为 `self` _没有_ `&`。这意味着代码清单 8-18 中的 `s1` 将被移动到 `add` 调用中，之后将不再有效。所以，尽管 `let s3 = s1 + &s2;` 看起来像是复制两个字符串并创建一个新字符串，但此语句实际上获取 `s1` 的所有权，追加 `s2` 内容的副本，然后返回结果的所有权。换句话说，看起来它正在制作很多副本，但实际上没有；实现比复制更高效。

如果我们需要连接多个字符串，`+` 操作符的行为会变得笨拙：

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-01-concat-multiple-strings/src/main.rs:here}}
```

此时，`s` 将是 `tic-tac-toe`。由于所有的 `+` 和 `"` 字符，很难看到发生了什么。对于以更复杂的方式组合字符串，我们可以改用 `format!` 宏：

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-02-format/src/main.rs:here}}
```

这段代码也将 `s` 设置为 `tic-tac-toe`。`format!` 宏的工作原理类似于 `println!`，但它不是将输出打印到屏幕，而是返回一个包含内容的 `String`。使用 `format!` 的代码版本更容易阅读，并且 `format!` 宏生成的代码使用引用，因此此调用不会获取任何其参数的所有权。

### 索引字符串

在许多其他编程语言中，通过索引引用字符串中的单个字符是有效且常见的操作。但是，如果你尝试在 Rust 中使用索引语法访问 `String` 的部分，你会得到错误。考虑代码清单 8-19 中的无效代码。

<Listing number="8-19" caption="尝试对 `String` 使用索引语法">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-19/src/main.rs:here}}
```

</Listing>

这段代码将导致以下错误：

```console
{{#include ../listings/ch08-common-collections/listing-08-19/output.txt}}
```

错误说明了问题：Rust 字符串不支持索引。但为什么不支持？要回答这个问题，我们需要讨论 Rust 如何在内存中存储字符串。

#### 内部表示

`String` 是 `Vec<u8>` 的包装器。让我们看看代码清单 8-14 中一些正确编码的 UTF-8 示例字符串。首先，这个：

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:spanish}}
```

在这种情况下，`len` 将是 `4`，这意味着存储字符串 `"Hola"` 的向量是 4 字节长。这些字母中的每一个在 UTF-8 编码时占用 1 字节。但是，以下行可能会让你感到惊讶（注意此字符串以大写西里尔字母 _Ze_ 开头，而不是数字 3）：

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:russian}}
```

如果你被问到这个字符串有多长，你可能会说 12。事实上，Rust 的答案是 24：这是在 UTF-8 中编码"Здравствуйте"所需的字节数，因为该字符串中的每个 Unicode 标量值占用 2 字节的存储空间。因此，字符串字节的索引并不总是与有效的 Unicode 标量值相对应。为了演示，考虑这个无效的 Rust 代码：

```rust,ignore,does_not_compile
let hello = "Здравствуйте";
let answer = &hello[0];
```

你已经知道 `answer` 不会是 `З`，第一个字母。当在 UTF-8 中编码时，`З` 的第一个字节是 `208`，第二个是 `151`，所以看起来 `answer` 实际上应该是 `208`，但 `208` 本身不是一个有效字符。返回 `208` 可能不是用户想要的，如果他们要求此字符串的第一个字母；但是，这是 Rust 在字节索引 0 处拥有的唯一数据。用户通常不希望返回字节值，即使字符串只包含拉丁字母：如果 `&"hi"[0]` 是返回字节值的有效代码，它将返回 `104`，而不是 `h`。

那么，答案是为了避免返回意外值并导致可能不会立即发现的错误，Rust 根本不会编译此代码，并在开发过程的早期防止误解。

<!-- Old headings. Do not remove or links may break. -->

<a id="bytes-and-scalar-values-and-grapheme-clusters-oh-my"></a>

#### 字节、标量值和字素簇

关于 UTF-8 的另一点是，实际上有三种相关的方式从 Rust 的角度来看字符串：作为字节、标量值和字素簇（最接近我们称之为 _字母_ 的东西）。

如果我们查看用天城文脚本编写的印地语单词"नमस्ते"，它存储为如下所示的 `u8` 值向量：

```text
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```

这是 18 字节，是计算机最终存储此数据的方式。如果我们将它们视为 Unicode 标量值，这是 Rust 的 `char` 类型，那些字节看起来像这样：

```text
['न', 'म', 'स', '्', 'त', 'े']
```

这里有六个 `char` 值，但第四个和第六个不是字母：它们是本身没有意义的变音符号。最后，如果我们将它们视为字素簇，我们会得到一个人会称之为组成印地语单词的四个字母：

```text
["न", "म", "स्", "ते"]
```

Rust 提供了不同的方式来解释计算机存储的原始字符串数据，以便每个程序可以选择它需要的解释，无论数据是什么人类语言。

Rust 不允许我们索引到 `String` 以获取字符的最后一个原因是索引操作应该总是花费常数时间（O(1)）。但是使用 `String` 不可能保证该性能，因为 Rust 必须从头到尾遍历内容到索引以确定有多少有效字符。

### 字符串切片

索引字符串通常是一个坏主意，因为不清楚字符串索引操作的返回类型应该是什么：字节值、字符、字素簇还是字符串切片。如果你真的需要使用索引来创建字符串切片，Rust 要求你更具体。

而不是使用 `[]` 和单个数字进行索引，你可以使用 `[]` 和范围来创建包含特定字节的字符串切片：

```rust
let hello = "Здравствуйте";

let s = &hello[0..4];
```

这里，`s` 将是一个包含字符串前 4 字节的 `&str`。早些时候，我们提到这些字符中的每一个都是 2 字节，这意味着 `s` 将是 `Зд`。

如果我们尝试仅切片字符字节的一部分，例如 `&hello[0..1]`，Rust 会在运行时 panic，就像在向量中访问无效索引一样：

```console
{{#include ../listings/ch08-common-collections/output-only-01-not-char-boundary/output.txt}}
```

在使用范围创建字符串切片时应该小心，因为这样做可能会使你的程序崩溃。

<!-- Old headings. Do not remove or links may break. -->

<a id="methods-for-iterating-over-strings"></a>

### 迭代字符串

操作字符串片段的最佳方法是明确你想要字符还是字节。对于单个 Unicode 标量值，使用 `chars` 方法。在"Зд"上调用 `chars` 会分离并返回两个 `char` 类型的值，你可以遍历结果以访问每个元素：

```rust
for c in "Зд".chars() {
    println!("{c}");
}
```

这段代码将打印以下内容：

```text
З
д
```

或者，`bytes` 方法返回每个原始字节，这可能适合你的领域：

```rust
for b in "Зд".bytes() {
    println!("{b}");
}
```

这段代码将打印组成此字符串的 4 字节：

```text
208
151
208
180
```

但是一定要记住，有效的 Unicode 标量值可能由超过 1 字节组成。

从字符串中获取字素簇，就像天城文脚本一样，很复杂，所以标准库不提供此功能。如果你需要此功能，[crates.io](https://crates.io/)<!-- ignore -->上有可用的 crate。

<!-- Old headings. Do not remove or links may break. -->

<a id="strings-are-not-so-simple"></a>

### 处理字符串的复杂性

总结一下，字符串是复杂的。不同的编程语言在如何向程序员呈现这种复杂性方面做出了不同的选择。Rust 选择使正确处理 `String` 数据成为所有 Rust 程序的默认行为，这意味着程序员必须更多地考虑预先处理 UTF-8 数据。这种权衡比其他编程语言中明显暴露了更多字符串的复杂性，但它防止你不得不在开发生命周期的后期处理涉及非 ASCII 字符的错误。

好消息是标准库提供了大量基于 `String` 和 `&str` 类型构建的功能，以帮助正确处理这些复杂情况。请务必查看文档，了解有用的方法，如用于在字符串中搜索的 `contains` 和用于用另一个字符串替换字符串部分的 `replace`。

让我们切换到稍微不那么复杂的东西：哈希映射！
