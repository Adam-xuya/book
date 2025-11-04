## 数据类型

Rust 中的每个值都有特定的_数据类型_，它告诉 Rust 正在指定什么类型的数据，以便它知道如何处理该数据。我们将查看两种数据类型的子集：标量和复合。

请记住，Rust 是一种_静态类型_语言，这意味着它必须在编译时知道所有变量的类型。编译器通常可以根据值以及我们如何使用它来推断我们要使用的类型。在可能有许多类型的情况下，例如在第 2 章的["将猜测与秘密数字进行比较"][comparing-the-guess-to-the-secret-number]<!-- ignore -->部分中，我们使用 `parse` 将 `String` 转换为数字类型，我们必须添加类型注释，如下所示：

```rust
let guess: u32 = "42".parse().expect("Not a number!");
```

如果我们不添加前面代码中显示的 `: u32` 类型注释，Rust 将显示以下错误，这意味着编译器需要我们提供更多信息以了解我们要使用哪种类型：

```console
{{#include ../listings/ch03-common-programming-concepts/output-only-01-no-type-annotations/output.txt}}
```

您将看到其他数据类型的不同类型注释。

### 标量类型

_标量_类型表示单个值。Rust 有四种主要的标量类型：整数、浮点数、布尔值和字符。您可能在其他编程语言中认识这些。让我们深入了解它们在 Rust 中的工作方式。

#### 整数类型

_整数_是没有小数部分的数字。我们在第 2 章中使用了一种整数类型，即 `u32` 类型。此类型声明表示与其关联的值应该是无符号整数（有符号整数类型以 `i` 而不是 `u` 开头），占用 32 位空间。表 3-1 显示了 Rust 中内置的整数类型。我们可以使用这些变体中的任何一个来声明整数值的类型。

<span class="caption">表 3-1：Rust 中的整数类型</span>

| 长度  | 有符号  | 无符号 |
| ------- | ------- | -------- |
| 8 位   | `i8`    | `u8`     |
| 16 位  | `i16`   | `u16`    |
| 32 位  | `i32`   | `u32`    |
| 64 位  | `i64`   | `u64`    |
| 128 位 | `i128`  | `u128`   |
| 架构相关 | `isize` | `usize`  |

每个变体可以是有符号或无符号的，并且有明确的大小。_有符号_和_无符号_指的是数字是否可能为负数——换句话说，数字是否需要带符号（有符号），或者是否永远为正数，因此可以在没有符号的情况下表示（无符号）。这就像在纸上写数字：当符号很重要时，数字会显示加号或减号；但是，当可以安全地假设数字为正数时，它不显示符号。有符号数字使用[二进制补码][twos-complement]<!-- ignore
-->表示法存储。

每个有符号变体可以存储从 −(2<sup>n − 1</sup>) 到 2<sup>n −
1</sup> − 1（包括）的数字，其中 _n_ 是该变体使用的位数。所以，`i8` 可以存储从 −(2<sup>7</sup>) 到 2<sup>7</sup> − 1 的数字，等于
−128 到 127。无符号变体可以存储从 0 到 2<sup>n</sup> − 1 的数字，
所以 `u8` 可以存储从 0 到 2<sup>8</sup> − 1 的数字，等于 0 到 255。

此外，`isize` 和 `usize` 类型取决于程序运行所在的计算机架构：如果您在 64 位架构上，则为 64 位；如果您在 32 位架构上，则为 32 位。

您可以使用表 3-2 中显示的任何形式编写整数字面量。请注意，可以是多种数字类型的数字字面量允许类型后缀，例如 `57u8`，以指定类型。数字字面量也可以使用 `_` 作为视觉分隔符，使数字更易于阅读，例如 `1_000`，它将具有与您指定 `1000` 相同的值。

<span class="caption">表 3-2：Rust 中的整数字面量</span>

| 数字字面量  | 示例       |
| ---------------- | ------------- |
| 十进制          | `98_222`      |
| 十六进制              | `0xff`        |
| 八进制            | `0o77`        |
| 二进制           | `0b1111_0000` |
| 字节（仅 `u8`） | `b'A'`        |

那么您如何知道要使用哪种整数类型？如果您不确定，Rust 的默认值通常是很好的起点：整数类型默认为 `i32`。
您使用 `isize` 或 `usize` 的主要情况是在索引某种集合时。

> ##### 整数溢出
>
> 假设您有一个 `u8` 类型的变量，可以保存 0 到 255 之间的值。如果您尝试将变量更改为该范围之外的值，例如 256，将发生_整数溢出_，这可能导致两种行为之一。当您在调试模式下编译时，Rust 包含整数溢出检查，如果发生此行为，这些检查会在运行时导致程序_panic_。Rust 使用术语_panicking_来表示程序因错误退出；我们将在第 9 章的["使用 `panic!` 处理不可恢复的错误"][unrecoverable-errors-with-panic]<!-- ignore -->部分更详细地讨论 panic。
>
> 当您使用 `--release` 标志在发布模式下编译时，Rust _不_包含导致 panic 的整数溢出检查。相反，如果发生溢出，Rust 执行_二进制补码包装_。简而言之，大于类型可以保存的最大值的值会"回绕"到类型可以保存的最小值。对于 `u8`，值 256 变为 0，值 257 变为 1，依此类推。程序不会 panic，但变量的值可能不是您期望的值。依赖整数溢出的包装行为被认为是错误。
>
> 要显式处理溢出的可能性，您可以使用标准库为原始数字类型提供的这些方法族：
>
> - 使用 `wrapping_*` 方法在所有模式下包装，例如 `wrapping_add`。
> - 如果有溢出，使用 `checked_*` 方法返回 `None` 值。
> - 使用 `overflowing_*` 方法返回值和表示是否有溢出的布尔值。
> - 使用 `saturating_*` 方法在值的最小值或最大值处饱和。

#### 浮点类型

Rust 还有两种_浮点数_的原始类型，即带小数点的数字。Rust 的浮点类型是 `f32` 和 `f64`，大小分别为 32 位和 64 位。默认类型是 `f64`，因为在现代 CPU 上，它的速度与 `f32` 大致相同，但精度更高。所有浮点类型都是有符号的。

这是一个显示浮点数在实际中的示例：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-06-floating-point/src/main.rs}}
```

浮点数根据 IEEE-754 标准表示。

#### 数值运算

Rust 支持所有数字类型的基本数学运算：加法、减法、乘法、除法和余数。整数除法向零截断到最近的整数。以下代码显示了如何在 `let` 语句中使用每个数值运算：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-07-numeric-operations/src/main.rs}}
```

这些语句中的每个表达式都使用数学运算符并求值为单个值，然后绑定到变量。[附录
B][appendix_b]<!-- ignore --> 包含 Rust 提供的所有运算符的列表。

#### 布尔类型

与大多数其他编程语言一样，Rust 中的布尔类型有两个可能的值：`true` 和 `false`。布尔值的大小为一个字节。Rust 中的布尔类型使用 `bool` 指定。例如：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-08-boolean/src/main.rs}}
```

使用布尔值的主要方式是通过条件语句，例如 `if` 表达式。我们将在["控制流"][control-flow]<!-- ignore -->部分介绍 `if` 表达式在 Rust 中的工作原理。

#### 字符类型

Rust 的 `char` 类型是该语言最原始的字母类型。以下是一些声明 `char` 值的示例：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-09-char/src/main.rs}}
```

请注意，我们使用单引号指定 `char` 字面量，这与使用双引号的字符串字面量相反。Rust 的 `char` 类型大小为 4 字节，表示 Unicode 标量值，这意味着它可以表示的内容远不止 ASCII。重音字母；中文、日文和韩文字符；表情符号；零宽度空格都是 Rust 中有效的 `char` 值。Unicode 标量值范围从 `U+0000` 到 `U+D7FF` 和 `U+E000` 到 `U+10FFFF`（包括）。但是，"字符"在 Unicode 中并不是真正的概念，因此您对"字符"是什么的人类直觉可能与 Rust 中的 `char` 不匹配。我们将在第 8 章的["使用字符串存储 UTF-8 编码的文本"][strings]<!-- ignore -->中详细讨论此主题。

### 复合类型

_复合类型_可以将多个值组合成一种类型。Rust 有两种原始复合类型：元组和数组。

#### 元组类型

_元组_是将多个具有各种类型的值组合成一个复合类型的一般方法。元组具有固定长度：一旦声明，它们不能增长或缩小。

我们通过在括号内写入逗号分隔的值列表来创建元组。元组中的每个位置都有一个类型，元组中不同值的类型不必相同。我们在本示例中添加了可选的类型注释：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-10-tuples/src/main.rs}}
```

变量 `tup` 绑定到整个元组，因为元组被视为单个复合元素。要从元组中获取单个值，我们可以使用模式匹配来解构元组值，如下所示：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-11-destructuring-tuples/src/main.rs}}
```

此程序首先创建一个元组并将其绑定到变量 `tup`。然后，它使用带有 `let` 的模式将 `tup` 转换为三个单独的变量 `x`、`y` 和 `z`。这称为_解构_，因为它将单个元组分解为三个部分。最后，程序打印 `y` 的值，即 `6.4`。

我们还可以通过使用点 (`.`) 后跟我们要访问的值的索引来直接访问元组元素。例如：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-12-tuple-indexing/src/main.rs}}
```

此程序创建元组 `x`，然后使用各自的索引访问元组的每个元素。与大多数编程语言一样，元组中的第一个索引是 0。

没有任何值的元组有一个特殊名称，_单元_。此值及其对应的类型都写为 `()`，表示空值或空返回类型。如果表达式不返回任何其他值，它们会隐式返回单元值。

#### 数组类型

拥有多个值集合的另一种方法是使用_数组_。与元组不同，数组的每个元素必须具有相同的类型。与某些其他语言中的数组不同，Rust 中的数组具有固定长度。

我们在方括号内写入逗号分隔的值列表来写入数组中的值：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-13-arrays/src/main.rs}}
```

当您希望数据分配在栈上（与我们到目前为止看到的其他类型相同）而不是堆上时（我们将在[第 4 章][stack-and-heap]<!-- ignore -->中更详细地讨论栈和堆），或者当您想确保始终有固定数量的元素时，数组很有用。但是，数组不如向量类型灵活。向量是标准库提供的类似集合类型，_允许_增长或缩小大小，因为其内容位于堆上。如果您不确定是使用数组还是向量，您应该使用向量。[第
8 章][vectors]<!-- ignore -->更详细地讨论了向量。

但是，当您知道元素数量不需要更改时，数组更有用。例如，如果您在程序中使用月份名称，您可能会使用数组而不是向量，因为您知道它将始终包含 12 个元素：

```rust
let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];
```

您使用方括号、每个元素的类型、分号，然后是数组中元素的数量来编写数组的类型，如下所示：

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```

这里，`i32` 是每个元素的类型。分号后，数字 `5` 表示数组包含五个元素。

您还可以通过指定初始值，后跟分号，然后在方括号中指定数组的长度来初始化数组，使其每个元素包含相同的值，如下所示：

```rust
let a = [3; 5];
```

名为 `a` 的数组将包含 `5` 个元素，这些元素最初都将设置为值 `3`。这与编写 `let a = [3, 3, 3, 3, 3];` 相同，但方式更简洁。

<!-- Old headings. Do not remove or links may break. -->
<a id="accessing-array-elements"></a>

#### 数组元素访问

数组是已知固定大小的单个内存块，可以分配在栈上。您可以使用索引访问数组的元素，如下所示：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-14-array-indexing/src/main.rs}}
```

在此示例中，名为 `first` 的变量将获得值 `1`，因为这是数组中索引 `[0]` 的值。名为 `second` 的变量将从数组中索引 `[1]` 获得值 `2`。

#### 无效的数组元素访问

让我们看看如果您尝试访问数组末尾之后的数组元素会发生什么。假设您运行此代码，类似于第 2 章中的猜数字游戏，从用户获取数组索引：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore,panics
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access/src/main.rs}}
```

此代码编译成功。如果您使用 `cargo run` 运行此代码并输入 `0`、`1`、`2`、`3` 或 `4`，程序将打印出数组中该索引处的对应值。如果您改为输入数组末尾之后的数字，例如 `10`，您将看到如下输出：

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access
cargo run
10
-->

```console
thread 'main' panicked at src/main.rs:19:19:
index out of bounds: the len is 5 but the index is 10
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

程序在使用无效值进行索引操作时导致运行时错误。程序退出并显示错误消息，并且没有执行最终的 `println!` 语句。当您尝试使用索引访问元素时，Rust 将检查您指定的索引是否小于数组长度。如果索引大于或等于长度，Rust 将 panic。此检查必须在运行时进行，特别是在这种情况下，因为编译器无法知道用户稍后运行代码时将输入什么值。

这是 Rust 内存安全原则在实际中的示例。在许多低级语言中，不进行这种检查，当您提供错误的索引时，可以访问无效内存。Rust 通过立即退出而不是允许内存访问并继续来保护您免受此类错误。第 9 章更详细地讨论了 Rust 的错误处理，以及如何编写既不会 panic 也不会允许无效内存访问的可读、安全代码。

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[twos-complement]: https://en.wikipedia.org/wiki/Two%27s_complement
[control-flow]: ch03-05-control-flow.html#control-flow
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[stack-and-heap]: ch04-01-what-is-ownership.html#the-stack-and-the-heap
[vectors]: ch08-01-vectors.html
[unrecoverable-errors-with-panic]: ch09-01-unrecoverable-errors-with-panic.html
[appendix_b]: appendix-02-operators.md
