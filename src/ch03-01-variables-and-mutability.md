## 变量和可变性

如["使用变量存储值"][storing-values-with-variables]<!-- ignore -->部分中提到的，默认情况下，变量是不可变的。这是 Rust 鼓励您以利用 Rust 提供的安全性和简单并发性的方式编写代码的众多推动之一。但是，您仍然可以选择使变量可变。让我们探讨 Rust 如何以及为什么鼓励您偏爱不可变性，以及为什么有时您可能想要选择退出。

当变量不可变时，一旦值绑定到名称，您就无法更改该值。为了说明这一点，在您的 _projects_ 目录中使用 `cargo new variables` 生成一个名为 _variables_ 的新项目。

然后，在您的新 _variables_ 目录中，打开 _src/main.rs_ 并将其代码替换为以下代码，该代码目前还无法编译：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/src/main.rs}}
```

保存并使用 `cargo run` 运行程序。您应该收到关于不可变性错误的错误消息，如本输出所示：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/output.txt}}
```

此示例展示了编译器如何帮助您查找程序中的错误。编译器错误可能令人沮丧，但实际上它们只是意味着您的程序尚未安全地执行您想要的操作；它们_并不_意味着您不是一个好程序员！经验丰富的 Rustaceans 仍然会遇到编译器错误。

您收到错误消息 `` cannot assign twice to immutable variable `x` `` 是因为您尝试为不可变的 `x` 变量分配第二个值。

当我们尝试更改指定为不可变的值时，在编译时获得错误很重要，因为这种情况可能导致错误。如果我们代码的一部分基于值永远不会改变的假设进行操作，而代码的另一部分更改了该值，则代码的第一部分可能无法按设计执行。这种错误的原因可能难以事后追踪，特别是当第二段代码_有时_才更改值时。Rust 编译器保证当您声明值不会改变时，它真的不会改变，因此您不必自己跟踪它。因此，您的代码更容易推理。

但是可变性可能非常有用，并且可以使代码编写更方便。尽管变量默认是不可变的，但您可以通过在变量名前面添加 `mut` 来使它们可变，就像您在[第
2 章][storing-values-with-variables]<!-- ignore -->中所做的那样。添加 `mut` 还通过指示代码的其他部分将更改此变量的值来向代码的未来读者传达意图。

例如，让我们将 _src/main.rs_ 更改为以下内容：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/src/main.rs}}
```

当我们现在运行程序时，我们得到这个：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/output.txt}}
```

当使用 `mut` 时，我们允许将绑定到 `x` 的值从 `5` 更改为 `6`。最终，决定是否使用可变性取决于您，并取决于您认为在特定情况下最清楚的方式。

<!-- Old headings. Do not remove or links may break. -->
<a id="constants"></a>

### 声明常量

与不可变变量一样，_常量_是绑定到名称且不允许更改的值，但常量和变量之间有一些区别。

首先，您不允许对常量使用 `mut`。常量不仅仅是默认不可变——它们总是不可变的。您使用 `const` 关键字而不是 `let` 关键字声明常量，并且_必须_注释值的类型。我们将在下一节["数据类型"][data-types]<!-- ignore -->中介绍类型和类型注释，所以现在不用担心细节。只需知道您必须始终注释类型。

常量可以在任何作用域中声明，包括全局作用域，这使得它们对于代码的许多部分需要了解的值很有用。

最后一个区别是常量只能设置为常量表达式，而不能设置为只能在运行时计算的值的结果。

这是一个常量声明的示例：

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

常量的名称是 `THREE_HOURS_IN_SECONDS`，其值设置为 60（一分钟的秒数）乘以 60（一小时的分钟数）乘以 3（我们在此程序中想要计算的小时数）的结果。Rust 对常量的命名约定是使用全大写，单词之间用下划线分隔。编译器能够在编译时评估一组有限的操作，这让我们选择以更易于理解和验证的方式写出此值，而不是将此常量设置为值 10,800。有关在声明常量时可以使用哪些操作的更多信息，请参见 [Rust 参考的常量评估部分][const-eval]。

常量在程序运行的整个时间内都是有效的，在它们被声明的作用域内。此属性使常量对于应用程序域中程序的多个部分可能需要了解的值很有用，例如游戏中任何玩家允许获得的最大点数，或光速。

将整个程序中使用的硬编码值命名为常量有助于向代码的未来维护者传达该值的含义。如果将来需要更新硬编码值，它还有助于在代码中只有一个地方需要更改。

### 遮蔽

如您在[第 2 章][comparing-the-guess-to-the-secret-number]<!-- ignore -->的猜数字游戏教程中看到的，您可以使用与先前变量相同的名称声明新变量。Rustaceans 说第一个变量被第二个变量_遮蔽_，这意味着当您使用变量名时，编译器将看到第二个变量。实际上，第二个变量遮蔽了第一个变量，将变量名的任何使用都归于自己，直到它自己被遮蔽或作用域结束。我们可以通过使用相同的变量名并重复使用 `let` 关键字来遮蔽变量，如下所示：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/src/main.rs}}
```

此程序首先将 `x` 绑定到值 `5`。然后，它通过重复 `let x =` 创建一个新变量 `x`，取原始值并添加 `1`，使 `x` 的值为 `6`。然后，在使用花括号创建的内部作用域内，第三个 `let` 语句也遮蔽 `x` 并创建一个新变量，将前一个值乘以 `2`，使 `x` 的值为 `12`。当该作用域结束时，内部遮蔽结束，`x` 恢复为 `6`。当我们运行此程序时，它将输出以下内容：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/output.txt}}
```

遮蔽与将变量标记为 `mut` 不同，因为如果我们意外尝试在不使用 `let` 关键字的情况下重新分配给此变量，我们将获得编译时错误。通过使用 `let`，我们可以对值执行一些转换，但在这些转换完成后，变量是不可变的。

`mut` 和遮蔽之间的另一个区别是，因为当我们再次使用 `let` 关键字时，我们实际上是在创建一个新变量，所以我们可以更改值的类型但重用相同的名称。例如，假设我们的程序要求用户通过输入空格字符来显示他们想要在某些文本之间有多少空格，然后我们希望将该输入存储为数字：

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-04-shadowing-can-change-types/src/main.rs:here}}
```

第一个 `spaces` 变量是字符串类型，第二个 `spaces` 变量是数字类型。因此，遮蔽使我们免于必须想出不同的名称，例如 `spaces_str` 和 `spaces_num`；相反，我们可以重用更简单的 `spaces` 名称。但是，如果我们尝试为此使用 `mut`，如下所示，我们将获得编译时错误：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/src/main.rs:here}}
```

错误说我们不允许改变变量的类型：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/output.txt}}
```

现在我们已经探讨了变量的工作方式，让我们看看它们可以拥有的更多数据类型。

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[data-types]: ch03-02-data-types.html#data-types
[storing-values-with-variables]: ch02-00-guessing-game-tutorial.html#storing-values-with-variables
[const-eval]: https://doc.rust-lang.org/reference/const_eval.html
