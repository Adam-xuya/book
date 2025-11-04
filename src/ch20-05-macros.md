## 宏

我们在整本书中使用了像 `println!` 这样的宏，但我们还没有完全探索什么是宏以及它如何工作。术语_宏_指的是 Rust 中的一系列特性——使用 `macro_rules!` 的声明性宏和三种过程宏：

- 自定义 `#[derive]` 宏，指定与结构体和枚举上使用的 `derive` 属性一起添加的代码
- 类似属性的宏，定义可在任何项目上使用的自定义属性
- 类似函数的宏，看起来像函数调用，但操作指定为其参数的标记

我们将依次讨论这些，但首先，让我们看看为什么当我们已经有函数时还需要宏。

### 宏和函数之间的区别

从根本上说，宏是一种编写编写其他代码的代码的方法，这被称为_元编程_。在附录 C 中，我们讨论了 `derive` 属性，它为你生成各种 traits 的实现。我们还在整本书中使用了 `println!` 和 `vec!` 宏。所有这些宏都会_展开_以产生比你手动编写的代码更多的代码。

元编程对于减少你必须编写和维护的代码量很有用，这也是函数的角色之一。但是，宏具有函数没有的一些额外功能。

函数签名必须声明函数具有的参数的数量和类型。另一方面，宏可以接受可变数量的参数：我们可以用一个参数调用 `println!("hello")` 或用两个参数调用 `println!("hello {}", name)`。此外，宏在编译器解释代码的含义之前展开，所以宏可以，例如，在给定类型上实现 trait。函数不能，因为它在运行时被调用，而 trait 需要在编译时实现。

实现宏而不是函数的缺点在于宏定义比函数定义更复杂，因为你正在编写编写 Rust 代码的 Rust 代码。由于这种间接性，宏定义通常比函数定义更难阅读、理解和维护。

宏和函数之间的另一个重要区别是，你必须在使用宏之前定义宏或将它们引入作用域，这与函数不同，你可以在任何地方定义并在任何地方调用。

<!-- Old headings. Do not remove or links may break. -->

<a id="declarative-macros-with-macro_rules-for-general-metaprogramming"></a>

### 用于通用元编程的声明性宏

Rust 中使用最广泛的宏形式是_声明性宏_。这些有时也被称为“示例宏”、“`macro_rules!` 宏”或只是“宏”。在它们的核心，声明性宏允许你编写类似于 Rust `match` 表达式的代码。如第6章所讨论的，`match` 表达式是控制结构，它们接受表达式，将表达式的结果值与模式进行比较，然后运行与匹配模式关联的代码。宏也将值与与特定代码关联的模式进行比较：在这种情况下，值是传递给宏的字面 Rust 源代码；模式与该源代码的结构进行比较；与每个模式关联的代码，当匹配时，替换传递给宏的代码。这一切都发生在编译期间。

要定义宏，你使用 `macro_rules!` 构造。让我们通过查看 `vec!` 宏的定义来探索如何使用 `macro_rules!`。第8章介绍了如何使用 `vec!` 宏创建具有特定值的新向量。例如，以下宏创建一个包含三个整数的新向量：

```rust
let v: Vec<u32> = vec![1, 2, 3];
```

我们也可以使用 `vec!` 宏创建两个整数的向量或五个字符串切片的向量。我们无法使用函数做同样的事情，因为我们事先不知道值的数量或类型。

代码清单20-35显示了 `vec!` 宏的稍微简化的定义。

<Listing number="20-35" file-name="src/lib.rs" caption="`vec!` 宏定义的简化版本">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-35/src/lib.rs}}
```

</Listing>

> 注意：标准库中 `vec!` 宏的实际定义包括预先分配正确内存量的代码。该代码是一个优化，我们不在此处包含，以使示例更简单。

`#[macro_export]` 注释表示，每当定义宏的 crate 被引入作用域时，都应该使此宏可用。如果没有此注释，宏无法被引入作用域。

然后我们用 `macro_rules!` 开始宏定义，以及我们正在定义的宏的名称，_不带_感叹号。在这种情况下，名称 `vec` 后面跟着花括号，表示宏定义的主体。

`vec!` 主体中的结构类似于 `match` 表达式的结构。这里我们有一个分支，模式为 `( $( $x:expr ),* )`，后面跟着 `=>` 和与此模式关联的代码块。如果模式匹配，将发出关联的代码块。鉴于这是此宏中唯一的模式，只有一种有效的匹配方式；任何其他模式都会导致错误。更复杂的宏将有不只一个分支。

宏定义中的有效模式语法与第19章涵盖的模式语法不同，因为宏模式是针对 Rust 代码结构匹配的，而不是值。让我们逐步了解代码清单20-29中模式部分的含义；有关完整的宏模式语法，请参阅[Rust 参考文档][ref]。

首先，我们使用一组括号来包含整个模式。我们使用美元符号（`$`）在宏系统中声明一个变量，该变量将包含匹配模式的 Rust 代码。美元符号清楚地表明这是宏变量而不是常规 Rust 变量。接下来是一组括号，捕获与括号内模式匹配的值，以便在替换代码中使用。在 `$()` 内是 `$x:expr`，它匹配任何 Rust 表达式并将表达式命名为 `$x`。

`$()` 后面的逗号表示在匹配 `$()` 中代码的每个代码实例之间必须出现字面逗号分隔符。`*` 指定模式匹配 `*` 前面的任何内容的零次或多次。

当我们用 `vec![1, 2, 3];` 调用此宏时，`$x` 模式匹配三次，分别匹配三个表达式 `1`、`2` 和 `3`。

现在让我们看看与此分支关联的代码主体中的模式：`temp_vec.push()` 在 `$()*` 内为匹配 `$()` 的部分生成零次或多次，具体取决于模式匹配的次数。`$x` 被替换为每个匹配的表达式。当我们用 `vec![1, 2, 3];` 调用此宏时，生成的替换此宏调用的代码将是以下内容：

```rust,ignore
{
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
```

我们定义了一个宏，它可以接受任何数量的任何类型的参数，并可以生成代码来创建包含指定元素的向量。

要了解更多关于如何编写宏的信息，请查阅在线文档或其他资源，例如由 Daniel Keep 开始并由 Lukas Wirth 继续的[“The Little Book of Rust Macros”][tlborm]。

### 用于从属性生成代码的过程宏

宏的第二种形式是过程宏，它更像函数（并且是一种过程）。_过程宏_接受一些代码作为输入，对该代码进行操作，并产生一些代码作为输出，而不是匹配模式并像声明性宏那样用其他代码替换代码。三种过程宏是自定义 `derive`、类似属性的和类似函数的，它们都以类似的方式工作。

创建过程宏时，定义必须驻留在它们自己的 crate 中，具有特殊的 crate 类型。这是由于复杂的技术原因，我们希望在未来消除。在代码清单20-36中，我们展示了如何定义过程宏，其中 `some_attribute` 是使用特定宏种类的占位符。

<Listing number="20-36" file-name="src/lib.rs" caption="定义过程宏的示例">

```rust,ignore
use proc_macro::TokenStream;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```

</Listing>

定义过程宏的函数接受 `TokenStream` 作为输入并产生 `TokenStream` 作为输出。`TokenStream` 类型由 Rust 附带的 `proc_macro` crate 定义，表示一系列标记。这是宏的核心：宏操作的源代码构成输入 `TokenStream`，宏产生的代码是输出 `TokenStream`。函数还有一个附加到它的属性，指定我们正在创建哪种过程宏。我们可以在同一个 crate 中拥有多种过程宏。

让我们看看不同类型的过程宏。我们将从自定义 `derive` 宏开始，然后解释使其他形式不同的微小差异。

<!-- Old headings. Do not remove or links may break. -->

<a id="how-to-write-a-custom-derive-macro"></a>

### 自定义 `derive` 宏

让我们创建一个名为 `hello_macro` 的 crate，它定义一个名为 `HelloMacro` 的 trait，具有一个名为 `hello_macro` 的关联函数。与其让我们的用户为他们的每个类型实现 `HelloMacro` trait，我们将提供一个过程宏，以便用户可以使用 `#[derive(HelloMacro)]` 注释他们的类型，以获得 `hello_macro` 函数的默认实现。默认实现将打印 `Hello, Macro! My name is TypeName!`，其中 `TypeName` 是定义此 trait 的类型的名称。换句话说，我们将编写一个 crate，使另一个程序员能够使用我们的 crate 编写如代码清单20-37所示的代码。

<Listing number="20-37" file-name="src/main.rs" caption="当我们使用过程宏时，我们 crate 的用户将能够编写的代码">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-37/src/main.rs}}
```

</Listing>

当我们完成时，此代码将打印 `Hello, Macro! My name is Pancakes!`。第一步是创建一个新的库 crate，如下所示：

```console
$ cargo new hello_macro --lib
```

接下来，在代码清单20-38中，我们将定义 `HelloMacro` trait 及其关联函数。

<Listing file-name="src/lib.rs" number="20-38" caption="我们将与 `derive` 宏一起使用的简单 trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-38/hello_macro/src/lib.rs}}
```

</Listing>

我们有一个 trait 及其函数。此时，我们的 crate 用户可以实现 trait 以实现所需的功能，如代码清单20-39所示。

<Listing number="20-39" file-name="src/main.rs" caption="如果用户编写 `HelloMacro` trait 的手动实现会是什么样子">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-39/pancakes/src/main.rs}}
```

</Listing>

但是，他们需要为他们想要与 `hello_macro` 一起使用的每个类型编写实现块；我们希望使他们免于必须这样做。

此外，我们还不能为 `hello_macro` 函数提供默认实现，该实现将打印实现 trait 的类型的名称：Rust 没有反射功能，所以它无法在运行时查找类型的名称。我们需要一个宏在编译时生成代码。

下一步是定义过程宏。在撰写本文时，过程宏需要位于它们自己的 crate 中。最终，此限制可能会被取消。构建 crate 和宏 crate 的约定如下：对于名为 `foo` 的 crate，自定义 `derive` 过程宏 crate 称为 `foo_derive`。让我们在 `hello_macro` 项目内启动一个名为 `hello_macro_derive` 的新 crate：

```console
$ cargo new hello_macro_derive --lib
```

我们的两个 crate 紧密相关，所以我们在 `hello_macro` crate 的目录内创建过程宏 crate。如果我们在 `hello_macro` 中更改 trait 定义，我们也必须更改 `hello_macro_derive` 中过程宏的实现。这两个 crate 需要单独发布，使用这些 crate 的程序员需要将两者都添加为依赖项并将它们都引入作用域。我们可以改为让 `hello_macro` crate 使用 `hello_macro_derive` 作为依赖项并重新导出过程宏代码。但是，我们构建项目的方式使得程序员即使不想要 `derive` 功能也可以使用 `hello_macro`。

我们需要将 `hello_macro_derive` crate 声明为过程宏 crate。我们还需要 `syn` 和 `quote` crates 的功能，正如你将看到的，所以我们需要将它们添加为依赖项。将以下内容添加到 `hello_macro_derive` 的 _Cargo.toml_ 文件中：

<Listing file-name="hello_macro_derive/Cargo.toml">

```toml
{{#include ../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/Cargo.toml:6:12}}
```

</Listing>

要开始定义过程宏，请将代码清单20-40中的代码放入你的 `hello_macro_derive` crate 的 _src/lib.rs_ 文件中。请注意，在我们为 `impl_hello_macro` 函数添加定义之前，此代码不会编译。

<Listing number="20-40" file-name="hello_macro_derive/src/lib.rs" caption="大多数过程宏 crate 需要处理 Rust 代码的代码">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/src/lib.rs}}
```

</Listing>

请注意，我们已经将代码拆分为 `hello_macro_derive` 函数，它负责解析 `TokenStream`，以及 `impl_hello_macro` 函数，它负责转换语法树：这使得编写过程宏更加方便。外部函数（在这种情况下为 `hello_macro_derive`）中的代码对于你看到或创建的几乎每个过程宏 crate 都是相同的。你在内部函数（在这种情况下为 `impl_hello_macro`）主体中指定的代码将根据你的过程宏的目的而不同。

我们引入了三个新 crate：`proc_macro`、[`syn`][syn]<!-- ignore -->和 [`quote`][quote]<!-- ignore -->。`proc_macro` crate 随 Rust 一起提供，所以我们不需要将其添加到 _Cargo.toml_ 中的依赖项。`proc_macro` crate 是编译器的 API，允许我们从代码中读取和操作 Rust 代码。

`syn` crate 将 Rust 代码从字符串解析为我们可以执行操作的数据结构。`quote` crate 将 `syn` 数据结构转换回 Rust 代码。这些 crate 使解析我们可能想要处理的任何类型的 Rust 代码变得简单得多：为 Rust 代码编写完整的解析器不是简单的任务。

当我们的库用户在其类型上指定 `#[derive(HelloMacro)]` 时，将调用 `hello_macro_derive` 函数。这是可能的，因为我们在这里用 `proc_macro_derive` 注释了 `hello_macro_derive` 函数，并指定了名称 `HelloMacro`，它与我们的 trait 名称匹配；这是大多数过程宏遵循的约定。

`hello_macro_derive` 函数首先将 `input` 从 `TokenStream` 转换为我们可以解释和执行操作的数据结构。这就是 `syn` 发挥作用的地方。`syn` 中的 `parse` 函数接受 `TokenStream` 并返回表示已解析的 Rust 代码的 `DeriveInput` 结构体。代码清单20-41显示了从解析 `struct Pancakes;` 字符串得到的 `DeriveInput` 结构体的相关部分。

<Listing number="20-41" caption="当解析在代码清单20-37中具有宏属性的代码时，我们得到的 `DeriveInput` 实例">

```rust,ignore
DeriveInput {
    // --snip--

    ident: Ident {
        ident: "Pancakes",
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```

</Listing>

此结构体的字段显示我们解析的 Rust 代码是一个单元结构体，`ident`（_标识符_，意思是名称）为 `Pancakes`。此结构体上还有更多字段用于描述各种 Rust 代码；有关更多信息，请查看 [`syn` 文档中的 `DeriveInput`][syn-docs]。

很快我们将定义 `impl_hello_macro` 函数，这是我们构建想要包含的新 Rust 代码的地方。但在我们这样做之前，请注意我们的 `derive` 宏的输出也是 `TokenStream`。返回的 `TokenStream` 被添加到我们 crate 用户编写的代码中，因此当他们编译他们的 crate 时，他们将在修改后的 `TokenStream` 中获得我们提供的额外功能。

你可能已经注意到我们调用 `unwrap` 以使 `hello_macro_derive` 函数在调用 `syn::parse` 函数失败时 panic。我们的过程宏在错误时 panic 是必要的，因为 `proc_macro_derive` 函数必须返回 `TokenStream` 而不是 `Result` 以符合过程宏 API。我们通过使用 `unwrap` 简化了此示例；在生产代码中，你应该通过使用 `panic!` 或 `expect` 提供关于出了什么问题的更具体的错误消息。

现在我们已经有了将注释的 Rust 代码从 `TokenStream` 转换为 `DeriveInput` 实例的代码，让我们生成在注释类型上实现 `HelloMacro` trait 的代码，如代码清单20-42所示。

<Listing number="20-42" file-name="hello_macro_derive/src/lib.rs" caption="使用解析的 Rust 代码实现 `HelloMacro` trait">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-42/hello_macro/hello_macro_derive/src/lib.rs:here}}
```

</Listing>

我们使用 `ast.ident` 获得包含注释类型名称（标识符）的 `Ident` 结构体实例。代码清单20-41中的结构体显示，当我们在代码清单20-37中的代码上运行 `impl_hello_macro` 函数时，我们得到的 `ident` 将具有值为 `"Pancakes"` 的 `ident` 字段。因此，代码清单20-42中的 `name` 变量将包含一个 `Ident` 结构体实例，当打印时，它将是字符串 `"Pancakes"`，代码清单20-37中结构体的名称。

`quote!` 宏让我们定义想要返回的 Rust 代码。编译器期望与 `quote!` 宏执行的直接结果不同的东西，所以我们需要将其转换为 `TokenStream`。我们通过调用 `into` 方法来做到这一点，该方法消费此中间表示并返回所需的 `TokenStream` 类型的值。

`quote!` 宏还提供了一些非常酷的模板机制：我们可以输入 `#name`，`quote!` 将用变量 `name` 中的值替换它。你甚至可以做一些类似于常规宏工作的重复。查看 [`quote` crate 的文档][quote-docs]以获取详细介绍。

我们希望我们的过程宏为用户注释的类型生成 `HelloMacro` trait 的实现，我们可以通过使用 `#name` 获得。trait 实现有一个函数 `hello_macro`，其主体包含我们想要提供的功能：打印 `Hello, Macro! My name is`，然后是注释类型的名称。

这里使用的 `stringify!` 宏是 Rust 内置的。它接受 Rust 表达式，例如 `1 + 2`，并在编译时将表达式转换为字符串字面量，例如 `"1 + 2"`。这与 `format!` 或 `println!` 不同，它们是评估表达式然后将结果转换为 `String` 的宏。`#name` 输入可能是要字面打印的表达式，所以我们使用 `stringify!`。使用 `stringify!` 还可以通过在编译时将 `#name` 转换为字符串字面量来节省分配。

此时，`cargo build` 应该在 `hello_macro` 和 `hello_macro_derive` 中都成功完成。让我们将这些 crate 连接到代码清单20-37中的代码，看看过程宏的实际效果！在你的 _projects_ 目录中使用 `cargo new pancakes` 创建一个新的二进制项目。我们需要在 `pancakes` crate 的 _Cargo.toml_ 中将 `hello_macro` 和 `hello_macro_derive` 添加为依赖项。如果你将 `hello_macro` 和 `hello_macro_derive` 的版本发布到 [crates.io](https://crates.io/)<!-- ignore -->，它们将是常规依赖项；如果没有，你可以将它们指定为 `path` 依赖项，如下所示：

```toml
{{#include ../listings/ch20-advanced-features/no-listing-21-pancakes/pancakes/Cargo.toml:6:8}}
```

将代码清单20-37中的代码放入 _src/main.rs_，并运行 `cargo run`：它应该打印 `Hello, Macro! My name is Pancakes!`。来自过程宏的 `HelloMacro` trait 的实现被包含，而不需要 `pancakes` crate 实现它；`#[derive(HelloMacro)]` 添加了 trait 实现。

接下来，让我们探索其他类型的过程宏与自定义 `derive` 宏的不同之处。

### 类似属性的宏

类似属性的宏类似于自定义 `derive` 宏，但它们不是为 `derive` 属性生成代码，而是允许你创建新属性。它们也更灵活：`derive` 仅适用于结构体和枚举；属性也可以应用于其他项目，例如函数。以下是使用类似属性宏的示例。假设你有一个名为 `route` 的属性，在使用 Web 应用程序框架时注释函数：

```rust,ignore
#[route(GET, "/")]
fn index() {
```

此 `#[route]` 属性将由框架定义为过程宏。宏定义函数的签名将如下所示：

```rust,ignore
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

在这里，我们有两个类型为 `TokenStream` 的参数。第一个用于属性的内容：`GET, "/"` 部分。第二个是属性附加到的项目的主体：在这种情况下，`fn index() {}` 和函数的其余主体。

除此之外，类似属性的宏的工作方式与自定义 `derive` 宏相同：你创建一个具有 `proc-macro` crate 类型的 crate，并实现一个生成你想要的代码的函数！

### 类似函数的宏

类似函数的宏定义看起来像函数调用的宏。类似于 `macro_rules!` 宏，它们比函数更灵活；例如，它们可以接受未知数量的参数。但是，`macro_rules!` 宏只能使用我们之前在[“用于通用元编程的声明性宏”][decl]<!-- ignore -->部分中讨论的类似匹配的语法定义。类似函数的宏接受 `TokenStream` 参数，它们的定义使用 Rust 代码操作该 `TokenStream`，就像其他两种过程宏一样。类似函数宏的一个示例是 `sql!` 宏，可能这样调用：

```rust,ignore
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

此宏将解析其中的 SQL 语句并检查它在语法上是否正确，这比 `macro_rules!` 宏可以做的处理要复杂得多。`sql!` 宏将这样定义：

```rust,ignore
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```

此定义类似于自定义 `derive` 宏的签名：我们接收括号内的标记，并返回我们想要生成的代码。

## 总结

呼！现在你的工具箱中有一些 Rust 特性，你可能不会经常使用，但你会知道它们在非常特定的情况下可用。我们介绍了几个复杂的主题，以便当你在错误消息建议或其他人的代码中遇到它们时，你将能够识别这些概念和语法。使用本章作为参考来指导你找到解决方案。

接下来，我们将把我们在整本书中讨论的所有内容付诸实践，并再做一個项目！

[ref]: ../reference/macros-by-example.html
[tlborm]: https://veykril.github.io/tlborm/
[syn]: https://crates.io/crates/syn
[quote]: https://crates.io/crates/quote
[syn-docs]: https://docs.rs/syn/2.0/syn/struct.DeriveInput.html
[quote-docs]: https://docs.rs/quote
[decl]: #declarative-macros-with-macro_rules-for-general-metaprogramming
