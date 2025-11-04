## 定义枚举

结构体为你提供了一种将相关字段和数据组合在一起的方法，比如一个带有 `width` 和 `height` 的 `Rectangle`，而枚举为你提供了一种表示一个值可能是多个可能值中的一个的方法。例如，我们可能想说 `Rectangle` 是一组可能形状中的一个，这组形状还包括 `Circle` 和 `Triangle`。为了实现这一点，Rust 允许我们将这些可能性编码为枚举。

让我们看看一个我们可能想在代码中表达的情况，并了解为什么枚举在这种情况下有用且比结构体更合适。假设我们需要处理 IP 地址。目前，用于 IP 地址的两个主要标准是：版本四和版本六。因为这是我们程序可能遇到的 IP 地址的唯一可能性，我们可以 _枚举_ 所有可能的变体，这就是枚举名称的由来。

任何 IP 地址都可以是版本四或版本六地址之一，但不能同时是两者。IP 地址的这一特性使得枚举数据结构合适，因为枚举值只能是其变体之一。版本四和版本六地址本质上仍然是 IP 地址，所以当代码处理适用于任何类型 IP 地址的情况时，它们应该被视为同一类型。

我们可以通过在代码中定义一个 `IpAddrKind` 枚举并列出 IP 地址可能的类型 `V4` 和 `V6` 来表达这个概念。这些是枚举的变体：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:def}}
```

`IpAddrKind` 现在是一个自定义数据类型，我们可以在代码的其他地方使用。

### 枚举值

我们可以像这样创建 `IpAddrKind` 的两个变体的实例：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:instance}}
```

注意，枚举的变体位于其标识符的命名空间下，我们使用双冒号来分隔两者。这很有用，因为现在 `IpAddrKind::V4` 和 `IpAddrKind::V6` 这两个值都是同一类型：`IpAddrKind`。然后，例如，我们可以定义一个接受任何 `IpAddrKind` 的函数：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn}}
```

我们可以用任一变体调用此函数：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn_call}}
```

使用枚举还有更多优势。进一步思考我们的 IP 地址类型，目前我们没有存储实际 IP 地址 _数据_ 的方法；我们只知道它的 _类型_。鉴于你刚刚在第 5 章学习了结构体，你可能会想用结构体来解决这个问题，如代码清单 6-1 所示。

<Listing number="6-1" caption="使用 `struct` 存储 IP 地址的数据和 `IpAddrKind` 变体">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-01/src/main.rs:here}}
```

</Listing>

这里，我们定义了一个结构体 `IpAddr`，它有两个字段：一个是类型为 `IpAddrKind`（我们之前定义的枚举）的 `kind` 字段，另一个是类型为 `String` 的 `address` 字段。我们有两个这个结构体的实例。第一个是 `home`，它的 `kind` 值为 `IpAddrKind::V4`，关联的地址数据为 `127.0.0.1`。第二个实例是 `loopback`。它的 `kind` 值是 `IpAddrKind` 的另一个变体 `V6`，关联的地址是 `::1`。我们使用结构体将 `kind` 和 `address` 值捆绑在一起，所以现在变体与值关联在一起了。

然而，仅使用枚举来表示相同的概念更简洁：我们不必在结构体中放置枚举，而是可以直接将数据放入每个枚举变体中。这个新的 `IpAddr` 枚举定义表示 `V4` 和 `V6` 变体都将具有关联的 `String` 值：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-02-enum-with-data/src/main.rs:here}}
```

我们直接将数据附加到枚举的每个变体上，因此不需要额外的结构体。在这里，更容易看到枚举工作的另一个细节：我们定义的每个枚举变体的名称也会成为一个构造枚举实例的函数。也就是说，`IpAddr::V4()` 是一个函数调用，它接受一个 `String` 参数并返回一个 `IpAddr` 类型的实例。我们自动获得这个构造函数定义，这是定义枚举的结果。

使用枚举而不是结构体还有另一个优势：每个变体可以具有不同类型和数量的关联数据。版本四的 IP 地址总是有四个数值组件，其值在 0 到 255 之间。如果我们想将 `V4` 地址存储为四个 `u8` 值，但仍将 `V6` 地址表示为一个 `String` 值，我们无法用结构体做到这一点。枚举可以轻松处理这种情况：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-03-variants-with-different-data/src/main.rs:here}}
```

我们已经展示了定义数据结构来存储版本四和版本六 IP 地址的几种不同方法。然而，事实证明，想要存储 IP 地址并编码它们是什么类型是如此常见，以至于[标准库有一个我们可以使用的定义！][IpAddr]<!-- ignore --> 让我们看看标准库如何定义 `IpAddr`。它有我们定义和使用的确切枚举和变体，但它将地址数据嵌入到变体中，以两个不同结构体的形式，为每个变体定义不同：

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

这段代码说明你可以将任何类型的数据放入枚举变体中：字符串、数值类型或结构体，例如。你甚至可以包含另一个枚举！另外，标准库类型通常不会比你可能想出的更复杂。

注意，即使标准库包含 `IpAddr` 的定义，我们仍然可以创建和使用我们自己的定义而不会冲突，因为我们没有将标准库的定义引入我们的作用域。我们将在第 7 章中详细讨论将类型引入作用域。

让我们看看代码清单 6-2 中的另一个枚举示例：这个枚举在其变体中嵌入了多种类型。

<Listing number="6-2" caption="一个 `Message` 枚举，其变体各自存储不同数量和类型的值">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

</Listing>

这个枚举有四个具有不同类型的变体：

- `Quit`：完全没有关联数据
- `Move`：具有命名字段，就像结构体一样
- `Write`：包含单个 `String`
- `ChangeColor`：包含三个 `i32` 值

定义具有如代码清单 6-2 中变体的枚举类似于定义不同类型的结构体定义，除了枚举不使用 `struct` 关键字并且所有变体都分组在 `Message` 类型下。以下结构体可以保存与前述枚举变体相同的数据：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-04-structs-similar-to-message-enum/src/main.rs:here}}
```

但是如果我们使用不同的结构体，每个结构体都有自己的类型，我们就不能像使用代码清单 6-2 中定义的单个类型的 `Message` 枚举那样轻松地定义一个函数来接受这些类型的消息。

枚举和结构体之间还有一个相似之处：就像我们能够使用 `impl` 在结构体上定义方法一样，我们也可以在枚举上定义方法。这是一个名为 `call` 的方法，我们可以在 `Message` 枚举上定义它：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-05-methods-on-enums/src/main.rs:here}}
```

方法的主体将使用 `self` 来获取我们调用该方法的那个值。在这个例子中，我们创建了一个变量 `m`，它的值为 `Message::Write(String::from("hello"))`，这就是当 `m.call()` 运行时 `call` 方法体中 `self` 的值。

让我们看看标准库中另一个非常常见且有用的枚举：`Option`。

<!-- Old headings. Do not remove or links may break. -->

<a id="the-option-enum-and-its-advantages-over-null-values"></a>

### `Option` 枚举

本节探讨 `Option` 的案例研究，这是标准库定义的另一个枚举。`Option` 类型编码了一个非常常见的场景：一个值可能是某个值，也可能什么都没有。

例如，如果你请求非空列表中的第一项，你会得到一个值。如果你请求空列表中的第一项，你会什么也得不到。用类型系统的术语表达这个概念意味着编译器可以检查你是否处理了你应该处理的所有情况；这个功能可以防止在其他编程语言中极其常见的错误。

编程语言设计通常被认为是关于你包含哪些功能，但你排除的功能也很重要。Rust 没有许多其他语言具有的 null 功能。_Null_ 是一个值，表示那里没有值。在具有 null 的语言中，变量总是可以处于两种状态之一：null 或非 null。

在 2009 年的演讲“空引用：十亿美元的错误”中，null 的发明者 Tony Hoare 这样说：

> 我称它为我的十亿美元错误。当时，我正在为面向对象语言中的引用设计第一个全面的类型系统。我的目标是确保所有引用的使用都应该是绝对安全的，由编译器自动执行检查。但我无法抗拒放入空引用的诱惑，仅仅因为它太容易实现了。这导致了无数的错误、漏洞和系统崩溃，可能在过去的四十年中造成了十亿美元的痛苦和损失。

空值的问题是，如果你尝试将空值用作非空值，你会遇到某种错误。因为这个空或非空属性是普遍存在的，所以很容易犯这种错误。

然而，null 试图表达的概念仍然是有用的：null 是由于某种原因当前无效或缺失的值。

问题不在于概念本身，而在于特定的实现。因此，Rust 没有 null，但它确实有一个可以编码值存在或不存在概念的枚举。这个枚举是 `Option<T>`，它由[标准库定义][option]<!-- ignore -->如下：

```rust
enum Option<T> {
    None,
    Some(T),
}
```

`Option<T>` 枚举非常有用，它甚至包含在预导入模块中；你不需要显式地将其引入作用域。它的变体也包含在预导入模块中：你可以直接使用 `Some` 和 `None`，而不需要 `Option::` 前缀。`Option<T>` 枚举仍然只是一个常规枚举，`Some(T)` 和 `None` 仍然是 `Option<T>` 类型的变体。

`<T>` 语法是 Rust 的一个我们尚未讨论的功能。它是一个泛型类型参数，我们将在第 10 章中更详细地介绍泛型。现在，你只需要知道 `<T>` 意味着 `Option` 枚举的 `Some` 变体可以保存任意类型的一条数据，而每个替代 `T` 使用的具体类型使整个 `Option<T>` 类型成为不同的类型。以下是一些使用 `Option` 值来保存数字类型和 char 类型的示例：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-06-option-examples/src/main.rs:here}}
```

`some_number` 的类型是 `Option<i32>`。`some_char` 的类型是 `Option<char>`，这是一个不同的类型。Rust 可以推断这些类型，因为我们在 `Some` 变体中指定了一个值。对于 `absent_number`，Rust 要求我们注释整个 `Option` 类型：编译器无法仅通过查看 `None` 值来推断相应的 `Some` 变体将保存的类型。在这里，我们告诉 Rust 我们希望 `absent_number` 的类型为 `Option<i32>`。

当我们有一个 `Some` 值时，我们知道存在一个值，并且该值保存在 `Some` 中。当我们有一个 `None` 值时，在某种意义上它意味着与 null 相同的事情：我们没有有效值。那么，为什么拥有 `Option<T>` 比拥有 null 更好呢？

简而言之，因为 `Option<T>` 和 `T`（其中 `T` 可以是任何类型）是不同的类型，编译器不会让我们像使用确定的有效值一样使用 `Option<T>` 值。例如，这段代码不会编译，因为它试图将 `i8` 加到 `Option<i8>`：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/src/main.rs:here}}
```

如果我们运行这段代码，我们会得到类似这样的错误消息：

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/output.txt}}
```

强烈！实际上，这个错误消息意味着 Rust 不知道如何将 `i8` 和 `Option<i8>` 相加，因为它们是不同的类型。当我们在 Rust 中有一个像 `i8` 这样的类型的值时，编译器将确保我们始终有一个有效值。我们可以自信地继续，而不必在使用该值之前检查 null。只有当我们在处理 `Option<i8>`（或我们正在处理的任何类型的值）时，我们才需要担心可能没有值，编译器将确保我们在使用该值之前处理这种情况。

换句话说，你必须先将 `Option<T>` 转换为 `T`，然后才能对其执行 `T` 操作。一般来说，这有助于捕获 null 最常见的问题之一：假设某个值不是 null，而实际上它是 null。

消除错误假设非空值的风险有助于你对自己的代码更有信心。为了拥有一个可能为 null 的值，你必须通过将该值的类型设为 `Option<T>` 来明确选择加入。然后，当你使用该值时，你必须明确处理该值为 null 的情况。在值具有不是 `Option<T>` 类型的任何地方，你 _可以_ 安全地假设该值不是 null。这是 Rust 的一项刻意设计决策，以限制 null 的普遍性并提高 Rust 代码的安全性。

那么，当你有一个 `Option<T>` 类型的值时，如何从 `Some` 变体中获取 `T` 值以便可以使用该值？`Option<T>` 枚举有大量在各种情况下都很有用的方法；你可以在[其文档][docs]<!-- ignore -->中查看它们。熟悉 `Option<T>` 上的方法在你使用 Rust 的旅程中将非常有用。

一般来说，为了使用 `Option<T>` 值，你需要有处理每个变体的代码。你需要一些代码只在有 `Some(T)` 值时运行，并且这段代码可以使用内部的 `T`。你需要一些其他代码只在有 `None` 值时运行，并且那段代码没有可用的 `T` 值。`match` 表达式是一个控制流构造，当与枚举一起使用时正好能做到这一点：它将根据它拥有的枚举变体运行不同的代码，并且该代码可以使用匹配值内的数据。

[IpAddr]: ../std/net/enum.IpAddr.html
[option]: ../std/option/enum.Option.html
[docs]: ../std/option/enum.Option.html
