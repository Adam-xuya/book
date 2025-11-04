## 可以使用模式的所有位置

模式在 Rust 的许多地方出现，你一直在使用它们却没有意识到！本节讨论模式有效的所有位置。

### `match` 分支

如第6章所述，我们在 `match` 表达式的分支中使用模式。正式地，`match` 表达式定义为关键字 `match`、要匹配的值，以及一个或多个匹配分支，每个分支由一个模式和一个表达式组成，如果值匹配该分支的模式，则运行该表达式，如下所示：

<!--
  Manually formatted rather than using Markdown intentionally: Markdown does not
  support italicizing code in the body of a block like this!
-->

<pre><code>match <em>VALUE</em> {
    <em>PATTERN</em> => <em>EXPRESSION</em>,
    <em>PATTERN</em> => <em>EXPRESSION</em>,
    <em>PATTERN</em> => <em>EXPRESSION</em>,
}</code></pre>

例如，这是代码清单6-5中匹配变量 `x` 中 `Option<i32>` 值的 `match` 表达式：

```rust,ignore
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

这个 `match` 表达式中的模式是每个箭头左侧的 `None` 和 `Some(i)`。

`match` 表达式的一个要求是它们需要是穷尽的，即 `match` 表达式中的值的所有可能性都必须被考虑。确保你覆盖了每种可能性的一种方法是为最后一个分支提供一个捕获所有内容的模式：例如，匹配任何值的变量名永远不会失败，因此覆盖了所有剩余情况。

特定模式 `_` 将匹配任何内容，但它永远不会绑定到变量，因此它经常在最后一个匹配分支中使用。例如，当你想要忽略任何未指定的值时，`_` 模式很有用。我们将在本章后面的[“忽略模式中的值”][ignoring-values-in-a-pattern]<!-- ignore -->部分更详细地介绍 `_` 模式。

### `let` 语句

在本章之前，我们只明确讨论了将模式与 `match` 和 `if let` 一起使用，但实际上，我们也在其他地方使用了模式，包括在 `let` 语句中。例如，考虑这个使用 `let` 的简单变量赋值：

```rust
let x = 5;
```

每次你使用这样的 `let` 语句时，你都在使用模式，尽管你可能没有意识到！更正式地，`let` 语句如下所示：

<!--
  Manually formatted rather than using Markdown intentionally: Markdown does not
  support italicizing code in the body of a block like this!
-->

<pre>
<code>let <em>PATTERN</em> = <em>EXPRESSION</em>;</code>
</pre>

在像 `let x = 5;` 这样在 PATTERN 槽中有变量名的语句中，变量名只是模式的特别简单形式。Rust 将表达式与模式进行比较，并分配它找到的任何名称。因此，在 `let x = 5;` 示例中，`x` 是一个模式，意思是“将这里匹配的内容绑定到变量 `x`”。因为名称 `x` 是整个模式，所以这个模式实际上意味着“将所有内容绑定到变量 `x`，无论值是什么。”

要更清楚地看到 `let` 的模式匹配方面，请考虑代码清单19-1，它使用带有 `let` 的模式来解构元组。


<Listing number="19-1" caption="使用模式解构元组并一次性创建三个变量">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-01/src/main.rs:here}}
```

</Listing>

在这里，我们将元组与模式匹配。Rust 将值 `(1, 2, 3)` 与模式 `(x, y, z)` 进行比较，并看到值匹配模式——也就是说，它看到元素的数量在两者中是相同的——所以 Rust 将 `1` 绑定到 `x`，`2` 绑定到 `y`，`3` 绑定到 `z`。你可以将此元组模式视为在其中嵌套三个单独的变量模式。

如果模式中的元素数量与元组中的元素数量不匹配，整体类型将不匹配，我们将得到编译器错误。例如，代码清单19-2显示了尝试将具有三个元素的元组解构为两个变量的尝试，这不会工作。

<Listing number="19-2" caption="错误地构造一个模式，其变量与元组中的元素数量不匹配">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-02/src/main.rs:here}}
```

</Listing>

尝试编译此代码会导致此类型错误：

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-02/output.txt}}
```

要修复错误，我们可以使用 `_` 或 `..` 忽略元组中的一个或多个值，正如你将在[“忽略模式中的值”][ignoring-values-in-a-pattern]<!-- ignore -->部分中看到的。如果问题是我们模式中的变量太多，解决方案是通过删除变量来使类型匹配，以便变量的数量等于元组中元素的数量。

### 条件 `if let` 表达式

在第6章中，我们讨论了如何使用 `if let` 表达式主要是作为编写仅匹配一种情况的 `match` 等效项的较短方式。可选地，`if let` 可以有一个对应的 `else`，包含在 `if let` 中的模式不匹配时运行的代码。

代码清单19-3显示混合和匹配 `if let`、`else if` 和 `else if let` 表达式也是可能的。这样做比 `match` 表达式给我们更多的灵活性，在 `match` 表达式中我们只能表达一个值与模式进行比较。此外，Rust 不要求一系列 `if let`、`else if` 和 `else if let` 分支中的条件彼此相关。

代码清单19-3中的代码基于对几个条件的一系列检查来确定背景颜色。对于此示例，我们创建了具有硬编码值的变量，真实程序可能会从用户输入接收这些值。

<Listing number="19-3" file-name="src/main.rs" caption="混合 `if let`、`else if`、`else if let` 和 `else`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-03/src/main.rs}}
```

</Listing>

如果用户指定了最喜欢的颜色，该颜色将用作背景。如果没有指定最喜欢的颜色并且今天是星期二，背景颜色是绿色。否则，如果用户将其年龄指定为字符串并且我们可以成功将其解析为数字，颜色是紫色或橙色，取决于数字的值。如果这些条件都不适用，背景颜色是蓝色。

这种条件结构使我们能够支持复杂的需求。使用我们这里的硬编码值，此示例将打印 `Using purple as the background color`。

你可以看到 `if let` 也可以引入新的变量，这些变量会以与 `match` 分支相同的方式遮蔽现有变量：行 `if let Ok(age) = age` 引入一个新的 `age` 变量，该变量包含 `Ok` 变体内的值，遮蔽现有的 `age` 变量。这意味着我们需要将 `if age > 30` 条件放在该块内：我们不能将这两个条件合并为 `if let Ok(age) = age && age > 30`。我们想要与 30 比较的新 `age` 在新的作用域以大括号开始之前是无效的。

使用 `if let` 表达式的缺点是编译器不检查穷尽性，而 `match` 表达式会检查。如果我们省略了最后一个 `else` 块，因此错过了处理某些情况，编译器不会警告我们可能的逻辑错误。

### `while let` 条件循环

在构造上类似于 `if let`，`while let` 条件循环允许 `while` 循环在模式继续匹配时运行。在代码清单19-4中，我们显示了一个 `while let` 循环，它等待在线程之间发送的消息，但在这种情况下检查 `Result` 而不是 `Option`。

<Listing number="19-4" caption="使用 `while let` 循环打印值，只要 `rx.recv()` 返回 `Ok`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-04/src/main.rs:here}}
```

</Listing>

此示例打印 `1`、`2`，然后打印 `3`。`recv` 方法从通道的接收端取出第一条消息并返回 `Ok(value)`。当我们第一次在第16章中看到 `recv` 时，我们直接解包错误，或者使用 `for` 循环将其作为迭代器进行交互。但是，如代码清单19-4所示，我们也可以使用 `while let`，因为 `recv` 方法在每次消息到达时返回 `Ok`，只要发送者存在，然后在发送端断开连接后产生 `Err`。

### `for` 循环

在 `for` 循环中，直接跟在关键字 `for` 后面的值是模式。例如，在 `for x in y` 中，`x` 是模式。代码清单19-5演示了如何在 `for` 循环中使用模式来解构或拆分元组作为 `for` 循环的一部分。


<Listing number="19-5" caption="在 `for` 循环中使用模式解构元组">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-05/src/main.rs:here}}
```

</Listing>

代码清单19-5中的代码将打印以下内容：


```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-05/output.txt}}
```

我们使用 `enumerate` 方法调整迭代器，以便它产生一个值该值的索引，放入元组中。产生的第一个值是元组 `(0, 'a')`。当此值匹配模式 `(index, value)` 时，index 将是 `0`，value 将是 `'a'`，打印输出的第一行。


### 函数参数

函数参数也可以是模式。代码清单19-6中的代码声明了一个名为 `foo` 的函数，它接受一个名为 `x` 的类型为 `i32` 的参数，现在应该看起来很熟悉。

<Listing number="19-6" caption="在参数中使用模式的函数签名">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-06/src/main.rs:here}}
```

</Listing>

`x` 部分是一个模式！正如我们对 `let` 所做的那样，我们可以将函数参数中的元组与模式匹配。代码清单19-7在将其传递给函数时拆分元组中的值。

<Listing number="19-7" file-name="src/main.rs" caption="一个具有解构元组参数的函数">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-07/src/main.rs}}
```

</Listing>

此代码将打印 `Current location: (3, 5)`。值 `&(3, 5)` 匹配模式 `&(x, y)`，所以 `x` 是值 `3`，`y` 是值 `5`。

我们也可以在闭包参数列表中以与函数参数列表相同的方式使用模式，因为闭包类似于函数，如第13章中讨论的。

此时，你已经看到了几种使用模式的方法，但模式在我们可以使用它们的每个地方的工作方式并不相同。在某些地方，模式必须是不可反驳的；在其他情况下，它们可以是可反驳的。我们接下来将讨论这两个概念。

[ignoring-values-in-a-pattern]: ch19-03-pattern-syntax.html#ignoring-values-in-a-pattern
