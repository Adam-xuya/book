## 控制流

根据条件是否为 `true` 运行某些代码的能力，以及在条件为 `true` 时重复运行某些代码的能力，是大多数编程语言中的基本构建块。让您控制 Rust 代码执行流程的最常见构造是 `if` 表达式和循环。

### `if` 表达式

`if` 表达式允许您根据条件分支代码。您提供一个条件，然后说明："如果满足此条件，运行此代码块。如果不满足条件，不要运行此代码块。"

在您的 _projects_ 目录中创建一个名为 _branches_ 的新项目以探索 `if` 表达式。在 _src/main.rs_ 文件中，输入以下内容：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/src/main.rs}}
```

所有 `if` 表达式都以关键字 `if` 开头，后跟条件。在这种情况下，条件检查变量 `number` 的值是否小于 5。我们将条件为 `true` 时要执行的代码块放在条件之后的花括号内。与 `if` 表达式中的条件关联的代码块有时称为_分支_，就像我们在第 2 章的["将猜测与秘密数字进行比较"][comparing-the-guess-to-the-secret-number]<!--
ignore -->部分中讨论的 `match` 表达式中的分支一样。

可选地，我们还可以包含 `else` 表达式，我们在这里选择这样做，以在条件求值为 `false` 时为程序提供要执行的替代代码块。如果您不提供 `else` 表达式且条件为 `false`，程序将跳过 `if` 块并继续执行下一段代码。

尝试运行此代码；您应该看到以下输出：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/output.txt}}
```

让我们尝试将 `number` 的值更改为使条件为 `false` 的值，看看会发生什么：

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/src/main.rs:here}}
```

再次运行程序，查看输出：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/output.txt}}
```

还值得注意的是，此代码中的条件_必须_是 `bool`。如果条件不是 `bool`，我们将得到错误。例如，尝试运行以下代码：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/src/main.rs}}
```

这次 `if` 条件求值为 `3`，Rust 抛出错误：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/output.txt}}
```

错误表明 Rust 期望 `bool` 但得到了整数。与 Ruby 和 JavaScript 等语言不同，Rust 不会自动尝试将非布尔类型转换为布尔值。您必须明确并始终为 `if` 提供布尔值作为其条件。例如，如果我们希望 `if` 代码块仅在数字不等于 `0` 时运行，我们可以将 `if` 表达式更改为以下内容：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-29-if-not-equal-0/src/main.rs}}
```

运行此代码将打印 `number was something other than zero`。

#### 使用 `else if` 处理多个条件

您可以通过在 `else if` 表达式中组合 `if` 和 `else` 来使用多个条件。例如：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/src/main.rs}}
```

此程序有四个可能的路径。运行后，您应该看到以下输出：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/output.txt}}
```

当此程序执行时，它依次检查每个 `if` 表达式，并执行条件求值为 `true` 的第一个主体。请注意，即使 6 可以被 2 整除，我们也看不到输出 `number is divisible by 2`，也看不到 `else` 块中的 `number is not divisible by 4, 3, or 2` 文本。这是因为 Rust 只执行第一个 `true` 条件的块，一旦找到一个，它甚至不会检查其余部分。

使用太多 `else if` 表达式会使您的代码混乱，因此如果您有多个，您可能想要重构代码。第 6 章描述了一个强大的 Rust 分支构造，称为 `match`，用于这些情况。

#### 在 `let` 语句中使用 `if`

因为 `if` 是表达式，我们可以在 `let` 语句的右侧使用它，将结果赋值给变量，如清单 3-2 所示。

<Listing number="3-2" file-name="src/main.rs" caption="将 `if` 表达式的结果赋值给变量">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-02/src/main.rs}}
```

</Listing>

`number` 变量将根据 `if` 表达式的结果绑定到值。运行此代码以查看会发生什么：

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-02/output.txt}}
```

请记住，代码块求值为其中的最后一个表达式，数字本身也是表达式。在这种情况下，整个 `if` 表达式的值取决于执行哪个代码块。这意味着可能作为 `if` 的每个分支结果的值的类型必须相同；在清单 3-2 中，`if` 分支和 `else` 分支的结果都是 `i32` 整数。如果类型不匹配，如下面的示例，我们将得到错误：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/src/main.rs}}
```

当我们尝试编译此代码时，我们将得到错误。`if` 和 `else` 分支的值类型不兼容，Rust 准确地指出了在程序中查找问题的位置：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/output.txt}}
```

`if` 块中的表达式求值为整数，`else` 块中的表达式求值为字符串。这不会工作，因为变量必须具有单一类型，并且 Rust 需要在编译时明确知道 `number` 变量的类型。知道 `number` 的类型让编译器验证该类型在我们使用 `number` 的任何地方都是有效的。如果 `number` 的类型只在运行时确定，Rust 将无法做到这一点；如果编译器必须跟踪任何变量的多个假设类型，它将更加复杂，并且对代码的保证会更少。

### 使用循环重复

多次执行代码块通常很有用。为此任务，Rust 提供了几种_循环_，它们将运行循环体内的代码直到结束，然后立即从头开始。为了试验循环，让我们创建一个名为 _loops_ 的新项目。

Rust 有三种循环：`loop`、`while` 和 `for`。让我们逐一尝试。

#### 使用 `loop` 重复代码

`loop` 关键字告诉 Rust 一遍又一遍地执行代码块，要么永远执行，要么直到您明确告诉它停止。

例如，将您的 _loops_ 目录中的 _src/main.rs_ 文件更改为如下所示：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-loop/src/main.rs}}
```

当我们运行此程序时，我们将看到 `again!` 一遍又一遍地连续打印，直到我们手动停止程序。大多数终端支持键盘快捷键 <kbd>ctrl</kbd>-<kbd>C</kbd> 来中断陷入持续循环的程序。试试看：

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-32-loop
cargo run
CTRL-C
-->

```console
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.08s
     Running `target/debug/loops`
again!
again!
again!
again!
^Cagain!
```

符号 `^C` 表示您按 <kbd>ctrl</kbd>-<kbd>C</kbd> 的位置。

您可能会或可能不会在 `^C` 之后看到 `again!` 打印，这取决于代码在收到中断信号时在循环中的位置。

幸运的是，Rust 还提供了一种使用代码跳出循环的方法。您可以在循环内放置 `break` 关键字，告诉程序何时停止执行循环。回想一下，我们在第 2 章的["在正确猜测后退出"][quitting-after-a-correct-guess]<!-- ignore
-->部分中的猜数字游戏中这样做了，当用户通过猜测正确数字赢得游戏时退出程序。

我们还在猜数字游戏中使用了 `continue`，它在循环中告诉程序跳过此循环迭代中的任何剩余代码并转到下一次迭代。

#### 从循环返回值

`loop` 的用途之一是重试您知道可能失败的操作，例如检查线程是否已完成其工作。您可能还需要将该操作的结果从循环传递到代码的其余部分。为此，您可以在用于停止循环的 `break` 表达式之后添加您想要返回的值；该值将从循环中返回，以便您可以使用它，如下所示：

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-33-return-value-from-loop/src/main.rs}}
```

在循环之前，我们声明一个名为 `counter` 的变量并将其初始化为 `0`。然后，我们声明一个名为 `result` 的变量来保存从循环返回的值。在循环的每次迭代中，我们将 `1` 添加到 `counter` 变量，然后检查 `counter` 是否等于 `10`。当它等于时，我们使用 `break` 关键字，值为 `counter * 2`。循环后，我们使用分号结束将值赋值给 `result` 的语句。最后，我们打印 `result` 中的值，在这种情况下是 `20`。

您也可以从循环内部 `return`。虽然 `break` 只退出当前循环，但 `return` 总是退出当前函数。

<!-- Old headings. Do not remove or links may break. -->
<a id="loop-labels-to-disambiguate-between-multiple-loops"></a>

#### 使用循环标签消除歧义

如果您有循环内的循环，`break` 和 `continue` 适用于该点的最内层循环。您可以选择在循环上指定_循环标签_，然后可以将其与 `break` 或 `continue` 一起使用，以指定这些关键字适用于标记的循环而不是最内层循环。循环标签必须以单引号开头。以下是两个嵌套循环的示例：

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/src/main.rs}}
```

外部循环有标签 `'counting_up`，它将从 0 计数到 2。没有标签的内部循环从 10 倒数到 9。第一个不指定标签的 `break` 只会退出内部循环。`break
'counting_up;` 语句将退出外部循环。此代码打印：

```console
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/output.txt}}
```

<!-- Old headings. Do not remove or links may break. -->
<a id="conditional-loops-with-while"></a>

#### 使用 while 简化条件循环

程序通常需要在循环内评估条件。当条件为 `true` 时，循环运行。当条件不再为 `true` 时，程序调用 `break`，停止循环。可以使用 `loop`、`if`、`else` 和 `break` 的组合来实现这样的行为；如果您愿意，现在可以在程序中尝试。但是，这种模式非常常见，以至于 Rust 为它内置了语言构造，称为 `while` 循环。在清单 3-3 中，我们使用 `while` 循环程序三次，每次倒计时，然后在循环后打印消息并退出。

<Listing number="3-3" file-name="src/main.rs" caption="使用 `while` 循环在条件求值为 `true` 时运行代码">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-03/src/main.rs}}
```

</Listing>

此构造消除了如果您使用 `loop`、`if`、`else` 和 `break` 将需要的许多嵌套，并且更清晰。当条件求值为 `true` 时，代码运行；否则，它退出循环。

#### 使用 `for` 循环遍历集合

您可以选择使用 `while` 构造来遍历集合的元素，例如数组。例如，清单 3-4 中的循环打印数组 `a` 中的每个元素。

<Listing number="3-4" file-name="src/main.rs" caption="使用 `while` 循环遍历集合的每个元素">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-04/src/main.rs}}
```

</Listing>

这里，代码向上计数数组中的元素。它从索引 `0` 开始，然后循环直到到达数组中的最终索引（即，当 `index < 5` 不再为 `true` 时）。运行此代码将打印数组中的每个元素：

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-04/output.txt}}
```

所有五个数组值都按预期出现在终端中。即使 `index` 在某个点会达到值 `5`，循环也会在尝试从数组中获取第六个值之前停止执行。

但是，这种方法容易出错；如果索引值或测试条件不正确，我们可能会导致程序 panic。例如，如果您将 `a` 数组的定义更改为有四个元素，但忘记将条件更新为 `while index < 4`，代码将 panic。它也很慢，因为编译器在每次循环迭代中添加运行时代码来执行索引是否在数组边界内的条件检查。

作为更简洁的替代方案，您可以使用 `for` 循环并为集合中的每个项目执行一些代码。`for` 循环看起来像清单 3-5 中的代码。

<Listing number="3-5" file-name="src/main.rs" caption="使用 `for` 循环遍历集合的每个元素">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-05/src/main.rs}}
```

</Listing>

当我们运行此代码时，我们将看到与清单 3-4 相同的输出。更重要的是，我们现在提高了代码的安全性，并消除了可能因超出数组末尾或不够远而遗漏某些项目而导致的错误。从 `for` 循环生成的机器代码也可能更高效，因为索引不需要在每次迭代时与数组长度进行比较。

使用 `for` 循环，如果您更改数组中的值数量，您不需要记住更改任何其他代码，就像您使用清单 3-4 中使用的方法一样。

`for` 循环的安全性和简洁性使它们成为 Rust 中最常用的循环构造。即使在您想要运行某些代码一定次数的情况下，如清单 3-3 中使用 `while` 循环的倒计时示例，大多数 Rustaceans 也会使用 `for` 循环。这样做的方法是使用标准库提供的 `Range`，它生成从一个数字开始并在另一个数字之前结束的所有数字序列。

以下是使用 `for` 循环和我们尚未讨论的另一种方法 `rev` 来反转范围的倒计时：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-34-for-range/src/main.rs}}
```

这段代码更好一点，不是吗？

## 总结

您做到了！这是一个相当大的章节：您学习了变量、标量和复合数据类型、函数、注释、`if` 表达式和循环！要练习本章讨论的概念，请尝试构建程序来执行以下操作：

- 在 Fahrenheit 和 Celsius 之间转换温度。
- 生成第 *n* 个斐波那契数。
- 打印圣诞颂歌"圣诞节的十二天"的歌词，利用歌曲中的重复。

当您准备好继续前进时，我们将讨论 Rust 中一个在其他编程语言中_不_常见存在的概念：所有权。

[comparing-the-guess-to-the-secret-number]: ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[quitting-after-a-correct-guess]: ch02-00-guessing-game-tutorial.html#quitting-after-a-correct-guess
