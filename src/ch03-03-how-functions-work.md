## 函数

函数在 Rust 代码中很常见。您已经看到了该语言中最重要的函数之一：`main` 函数，它是许多程序的入口点。您还看到了 `fn` 关键字，它允许您声明新函数。

Rust 代码使用_蛇形命名法_作为函数和变量名的约定样式，其中所有字母都是小写，下划线分隔单词。这是一个包含示例函数定义的程序：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-16-functions/src/main.rs}}
```

我们通过在 Rust 中输入 `fn` 后跟函数名和一组括号来定义函数。花括号告诉编译器函数体的开始和结束位置。

我们可以通过输入函数名后跟一组括号来调用我们定义的任何函数。因为 `another_function` 在程序中定义，所以可以从 `main` 函数内部调用它。请注意，我们在源代码中的 `main` 函数_之后_定义了 `another_function`；我们也可以在此之前定义它。Rust 不关心您在哪里定义函数，只关心它们在调用者可以看到的作用域中的某个地方定义。

让我们启动一个名为 _functions_ 的新二进制项目以进一步探索函数。将 `another_function` 示例放在 _src/main.rs_ 中并运行它。您应该看到以下输出：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-16-functions/output.txt}}
```

这些行按照它们在 `main` 函数中出现的顺序执行。首先打印 "Hello, world!" 消息，然后调用 `another_function` 并打印其消息。

### 参数

我们可以定义具有_参数_的函数，参数是函数签名一部分的特殊变量。当函数有参数时，您可以为这些参数提供具体值。从技术上讲，具体值称为_参数_，但在非正式对话中，人们倾向于互换使用_参数_和_参数_这两个词，用于函数定义中的变量或在调用函数时传入的具体值。

在此版本的 `another_function` 中，我们添加了一个参数：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/src/main.rs}}
```

尝试运行此程序；您应该得到以下输出：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/output.txt}}
```

`another_function` 的声明有一个名为 `x` 的参数。`x` 的类型指定为 `i32`。当我们将 `5` 传递给 `another_function` 时，`println!` 宏将 `5` 放在格式字符串中包含 `x` 的花括号对的位置。

在函数签名中，您_必须_声明每个参数的类型。这是 Rust 设计中的有意决定：在函数定义中要求类型注释意味着编译器几乎不需要您在其他地方使用它们来推断您指的是什么类型。如果编译器知道函数期望的类型，它也能够提供更有用的错误消息。

定义多个参数时，用逗号分隔参数声明，如下所示：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/src/main.rs}}
```

此示例创建一个名为 `print_labeled_measurement` 的函数，有两个参数。第一个参数名为 `value`，类型为 `i32`。第二个参数名为 `unit_label`，类型为 `char`。然后该函数打印包含 `value` 和 `unit_label` 的文本。

让我们尝试运行此代码。将您的 _functions_ 项目的 _src/main.rs_ 文件中当前的程序替换为前面的示例，并使用 `cargo run` 运行它：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/output.txt}}
```

因为我们用 `5` 作为 `value` 的值，用 `'h'` 作为 `unit_label` 的值调用函数，所以程序输出包含这些值。

### 语句和表达式

函数体由一系列语句组成，可选地以表达式结尾。到目前为止，我们介绍的函数都没有包含结尾表达式，但您已经看到表达式作为语句的一部分。因为 Rust 是一种基于表达式的语言，这是一个需要理解的重要区别。其他语言没有相同的区别，所以让我们看看语句和表达式是什么，以及它们的差异如何影响函数体。

- _语句_是执行某些操作且不返回值的指令。
- _表达式_求值为结果值。

让我们看一些示例。

我们实际上已经使用了语句和表达式。使用 `let` 关键字创建变量并为其赋值是一条语句。在清单 3-1 中，`let y = 6;` 是一条语句。

<Listing number="3-1" file-name="src/main.rs" caption="包含一条语句的 `main` 函数声明">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-01/src/main.rs}}
```

</Listing>

函数定义也是语句；前面的整个示例本身就是一个语句。（正如我们很快会看到的，调用函数不是语句，尽管。）

语句不返回值。因此，您不能将 `let` 语句赋值给另一个变量，如下面代码尝试做的那样；您会得到错误：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/src/main.rs}}
```

当您运行此程序时，您会得到的错误如下所示：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/output.txt}}
```

`let y = 6` 语句不返回值，因此 `x` 没有任何东西可以绑定。这与其他语言（如 C 和 Ruby）中发生的情况不同，在这些语言中，赋值返回赋值的值。在这些语言中，您可以编写 `x = y = 6` 并使 `x` 和 `y` 都具有值 `6`；在 Rust 中情况并非如此。

表达式求值为值，并构成您在 Rust 中编写的其余大部分代码。考虑一个数学运算，例如 `5 + 6`，这是一个求值为值 `11` 的表达式。表达式可以是语句的一部分：在清单 3-1 中，语句 `let y = 6;` 中的 `6` 是求值为值 `6` 的表达式。调用函数是表达式。调用宏是表达式。使用花括号创建的新作用域块是表达式，例如：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-20-blocks-are-expressions/src/main.rs}}
```

此表达式：

```rust,ignore
{
    let x = 3;
    x + 1
}
```

是一个块，在这种情况下，它求值为 `4`。该值作为 `let` 语句的一部分绑定到 `y`。请注意，`x + 1` 行末尾没有分号，这与您到目前为止看到的大多数行不同。表达式不包括结尾分号。如果您在表达式末尾添加分号，您会将其转换为语句，然后它不会返回值。在接下来探索函数返回值和表达式时，请记住这一点。

### 具有返回值的函数

函数可以向调用它们的代码返回值。我们不命名返回值，但必须在箭头 (`->`) 之后声明它们的类型。在 Rust 中，函数的返回值与函数体块中最后一个表达式的值同义。您可以使用 `return` 关键字并指定值来提前从函数返回，但大多数函数隐式返回最后一个表达式。这是一个返回值的函数示例：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/src/main.rs}}
```

`five` 函数中没有函数调用、宏，甚至没有 `let` 语句——只有数字 `5` 本身。这是 Rust 中完全有效的函数。请注意，函数的返回类型也指定为 `-> i32`。尝试运行此代码；输出应该如下所示：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/output.txt}}
```

`five` 中的 `5` 是函数的返回值，这就是返回类型是 `i32` 的原因。让我们更详细地检查这一点。有两个重要的部分：首先，`let x = five();` 行显示我们使用函数的返回值来初始化变量。因为函数 `five` 返回 `5`，所以该行与以下内容相同：

```rust
let x = 5;
```

其次，`five` 函数没有参数并定义返回值的类型，但函数体是一个单独的 `5`，没有分号，因为它是一个我们希望返回其值的表达式。

让我们看另一个示例：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-22-function-parameter-and-return/src/main.rs}}
```

运行此代码将打印 `The value of x is: 6`。但是，如果我们在包含 `x + 1` 的行末尾放置分号，将其从表达式更改为语句，会发生什么？

<span class="filename">文件名：src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/src/main.rs}}
```

编译此代码将产生错误，如下所示：

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/output.txt}}
```

主要错误消息 `mismatched types` 揭示了此代码的核心问题。函数 `plus_one` 的定义说它将返回 `i32`，但语句不求值为值，这由 `()`（单元类型）表示。因此，没有返回任何内容，这与函数定义相矛盾并导致错误。在此输出中，Rust 提供了一条消息，可能有助于纠正此问题：它建议删除分号，这将修复错误。
