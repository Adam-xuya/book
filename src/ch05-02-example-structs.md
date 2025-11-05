## 使用结构体的示例程序

要了解我们何时可能想要使用结构体，让我们编写一个计算矩形面积的程序。我们将从使用单个变量开始，然后重构程序，直到我们使用结构体。

让我们创建一个名为 _rectangles_ 的新 Cargo 二进制项目，该项目将接受以像素指定的矩形的宽度和高度，并计算矩形的面积。清单 5-8 显示了一个短程序，其中一种方法在我们项目的 _src/main.rs_ 中完全做到这一点。

<Listing number="5-8" file-name="src/main.rs" caption="通过单独的宽度和高度变量计算指定矩形的面积">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:all}}
```

</Listing>

现在，使用 `cargo run` 运行此程序：

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/output.txt}}
```

此代码通过使用每个维度调用 `area` 函数成功地计算出矩形的面积，但我们可以做更多来使代码清晰和可读。

此代码的问题在 `area` 的签名中很明显：

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:here}}
```

`area` 函数应该计算一个矩形的面积，但我们编写的函数有两个参数，并且在程序的任何地方都不清楚参数是相关的。将宽度和高度分组在一起会更易读和更易于管理。我们已经在第 3 章的["元组类型"][the-tuple-type]<!-- ignore -->部分讨论了一种可能的方法：使用元组。

### 使用元组重构

清单 5-9 显示了使用元组的我们程序的另一个版本。

<Listing number="5-9" file-name="src/main.rs" caption="使用元组指定矩形的宽度和高度">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-09/src/main.rs}}
```

</Listing>

在一种方式上，此程序更好。元组让我们添加一点结构，我们现在只传递一个参数。但在另一种方式上，此版本不太清楚：元组不命名其元素，所以我们必须索引到元组的各个部分，使我们的计算不太明显。

混淆宽度和高度对面积计算无关紧要，但如果我们想在屏幕上绘制矩形，那就很重要了！我们必须记住 `width` 是元组索引 `0`，`height` 是元组索引 `1`。如果其他人要使用我们的代码，这将更加困难，以便他们理解和记住。因为我们在代码中没有传达数据的含义，现在更容易引入错误。

<!-- Old headings. Do not remove or links may break. -->

<a id="refactoring-with-structs-adding-more-meaning"></a>

### 使用结构体重构

我们使用结构体通过标记数据来添加含义。我们可以将正在使用的元组转换为具有整体名称以及各部分名称的结构体，如清单 5-10 所示。

<Listing number="5-10" file-name="src/main.rs" caption="定义 `Rectangle` 结构体">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-10/src/main.rs}}
```

</Listing>

这里，我们定义了一个结构体并将其命名为 `Rectangle`。在花括号内，我们将字段定义为 `width` 和 `height`，两者都具有类型 `u32`。然后，在 `main` 中，我们创建了一个特定的 `Rectangle` 实例，宽度为 `30`，高度为 `50`。

我们的 `area` 函数现在使用一个参数定义，我们将其命名为 `rectangle`，其类型是结构体 `Rectangle` 实例的不可变借用。如第 4 章中提到的，我们想要借用结构体而不是获取其所有权。这样，`main` 保留其所有权并可以继续使用 `rect1`，这是我们在函数签名和使用该函数的地方使用 `&` 的原因。

`area` 函数访问 `Rectangle` 实例的 `width` 和 `height` 字段（请注意，访问借用结构体实例的字段不会移动字段值，这就是为什么您经常看到结构体的借用）。我们的 `area` 函数签名现在准确说明了我们的意思：使用 `Rectangle` 的 `width` 和 `height` 字段计算 `Rectangle` 的面积。这表明宽度和高度彼此相关，并为值提供描述性名称，而不是使用元组索引值 `0` 和 `1`。这对清晰度来说是一个胜利。

<!-- Old headings. Do not remove or links may break. -->

<a id="adding-useful-functionality-with-derived-traits"></a>

### 使用派生 Trait 添加功能

能够在调试程序时打印 `Rectangle` 的实例并查看其所有字段的值会很有用。清单 5-11 尝试使用 [`println!` 宏][println]<!-- ignore -->，就像我们在前面章节中使用的那样。但是，这不会起作用。

<Listing number="5-11" file-name="src/main.rs" caption="尝试打印 `Rectangle` 实例">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/src/main.rs}}
```

</Listing>

当我们编译此代码时，我们得到一条核心消息的错误：

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:3}}
```

`println!` 宏可以执行多种格式化，默认情况下，花括号告诉 `println!` 使用称为 `Display` 的格式化：用于直接最终用户消费的输出。我们到目前为止看到的原始类型默认实现 `Display`，因为只有一种方式您想要向用户显示 `1` 或任何其他原始类型。但对于结构体，`println!` 应该如何格式化输出不太清楚，因为有更多的显示可能性：您是否想要逗号？您是否想要打印花括号？应该显示所有字段吗？由于这种歧义，Rust 不会尝试猜测我们想要什么，结构体没有提供 `Display` 的实现以与 `println!` 和 `{}` 占位符一起使用。

如果我们继续阅读错误，我们会找到这个有用的注释：

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:9:10}}
```

让我们试试！`println!` 宏调用现在看起来像 `println!("rect1 is
{rect1:?}");`。将说明符 `:?` 放在花括号内告诉 `println!` 我们想要使用称为 `Debug` 的输出格式。`Debug` trait 使我们能够以对开发人员有用的方式打印结构体，以便我们可以在调试代码时看到其值。

使用此更改编译代码。哎呀！我们仍然得到错误：

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:3}}
```

但是再次，编译器给我们一个有用的注释：

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:9:10}}
```

Rust _确实_包括打印调试信息的功能，但我们必须显式选择加入以使该功能可用于我们的结构体。为此，我们在结构体定义之前添加外部属性 `#[derive(Debug)]`，如清单 5-12 所示。

<Listing number="5-12" file-name="src/main.rs" caption="添加属性以派生 `Debug` trait 并使用调试格式化打印 `Rectangle` 实例">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/src/main.rs}}
```

</Listing>

现在当我们运行程序时，我们不会得到任何错误，我们将看到以下输出：

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/output.txt}}
```

很好！这不是最漂亮的输出，但它显示了此实例的所有字段的值，这肯定会在调试期间有所帮助。当我们有更大的结构体时，拥有稍微更容易阅读的输出很有用；在这些情况下，我们可以在 `println!` 字符串中使用 `{:#?}` 而不是 `{:?}`。在此示例中，使用 `{:#?}` 样式将输出以下内容：

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-02-pretty-debug/output.txt}}
```

使用 `Debug` 格式打印值的另一种方法是使用 [`dbg!`
宏][dbg]<!-- ignore -->，它获取表达式的所有权（与 `println!` 相反，它接受引用），打印该 `dbg!` 宏调用在代码中发生的文件和行号以及该表达式的结果值，并返回值的所有权。

> 注意：调用 `dbg!` 宏打印到标准错误控制台流（`stderr`），与 `println!` 相反，它打印到标准输出控制台流（`stdout`）。我们将在第 12 章的["将错误重定向到标准错误"部分][err]<!-- ignore -->中更多地讨论 `stderr` 和 `stdout`。

这是一个示例，我们感兴趣的是赋值给 `width` 字段的值，以及 `rect1` 中整个结构体的值：

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/src/main.rs}}
```

我们可以将 `dbg!` 放在表达式 `30 * scale` 周围，因为 `dbg!` 返回表达式值的所有权，`width` 字段将获得与我们在那里没有 `dbg!` 调用相同的值。我们不希望 `dbg!` 获取 `rect1` 的所有权，所以我们在下一个调用中使用对 `rect1` 的引用。以下是此示例的输出：

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/output.txt}}
```

我们可以看到输出的第一部分来自 _src/main.rs_ 第 10 行，我们在那里调试表达式 `30 * scale`，其结果值是 `60`（为整数实现的 `Debug` 格式化是只打印它们的值）。_src/main.rs_ 第 14 行的 `dbg!` 调用输出 `&rect1` 的值，这是 `Rectangle` 结构体。此输出使用 `Rectangle` 类型的漂亮 `Debug` 格式化。`dbg!` 宏在您试图弄清楚代码在做什么时可能真的很有帮助！

除了 `Debug` trait，Rust 还为我们提供了许多 trait 与 `derive` 属性一起使用，这些属性可以为我们的自定义类型添加有用的行为。这些 trait 及其行为在[附录 C][app-c]<!--
ignore -->中列出。我们将在第 10 章中介绍如何使用自定义行为实现这些 trait，以及如何创建您自己的 trait。除了 `derive` 之外，还有许多其他属性；有关更多信息，请参见 [Rust 参考的"属性"部分][attributes]。

我们的 `area` 函数非常具体：它只计算矩形的面积。将此行为更紧密地绑定到我们的 `Rectangle` 结构体会很有帮助，因为它不会与任何其他类型一起工作。让我们看看如何通过将 `area` 函数转换为在我们的 `Rectangle` 类型上定义的 `area` 方法来继续重构此代码。

[the-tuple-type]: ch03-02-data-types.html#the-tuple-type
[app-c]: appendix-03-derivable-traits.md
[println]: https://doc.rust-lang.org/std/macro.println.html
[dbg]: https://doc.rust-lang.org/std/macro.dbg.html
[err]: ch12-06-writing-to-stderr-instead-of-stdout.html
[attributes]: https://doc.rust-lang.org/reference/attributes.html
