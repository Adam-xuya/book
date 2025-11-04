<!-- Old headings. Do not remove or links may break. -->

<a id="digging-into-the-traits-for-async"></a>

## 深入了解 Async 的 Traits

在本章中，我们以各种方式使用了 `Future`、`Stream` 和 `StreamExt` traits。但是，到目前为止，我们避免深入了解它们如何工作或它们如何组合在一起，这对于你的日常 Rust 工作来说大多数时候都很好。但是，有时你会遇到需要理解这些 traits 的更多细节的情况，以及 `Pin` 类型和 `Unpin` trait。在本节中，我们将深入了解足够的内容来帮助解决这些场景，仍然将_真正_的深入探讨留给其他文档。

<!-- Old headings. Do not remove or links may break. -->

<a id="future"></a>

### `Future` Trait

让我们首先更仔细地了解 `Future` trait 的工作原理。这是 Rust 如何定义它的：

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

这个 trait 定义包括一堆新类型以及一些我们之前没有见过的语法，所以让我们逐步了解这个定义的每个部分。

首先，`Future` 的关联类型 `Output` 说明了 future 解析为什么。这类似于 `Iterator` trait 的 `Item` 关联类型。其次，`Future` 有 `poll` 方法，它为其 `self` 参数接受一个特殊的 `Pin` 引用和对 `Context` 类型的可变引用，并返回 `Poll<Self::Output>`。我们稍后会更多地讨论 `Pin` 和 `Context`。现在，让我们专注于该方法返回的内容，`Poll` 类型：

```rust
pub enum Poll<T> {
    Ready(T),
    Pending,
}
```

这个 `Poll` 类型类似于 `Option`。它有一个有值的变体 `Ready(T)`，和一个没有值的变体 `Pending`。`Poll` 的含义与 `Option` 完全不同！`Pending` 变体表示 future 仍有工作要做，因此调用者需要稍后再检查。`Ready` 变体表示 `Future` 已完成其工作，并且 `T` 值可用。

> 注意：很少需要直接调用 `poll`，但如果你确实需要，请记住，对于大多数 futures，调用者不应该在 future 返回 `Ready` 之后再次调用 `poll`。许多 futures 如果在准备好后再次轮询会 panic。可以安全地再次轮询的 futures 会在其文档中明确说明这一点。这类似于 `Iterator::next` 的行为。

当你看到使用 `await` 的代码时，Rust 在底层将其编译为调用 `poll` 的代码。如果你回顾代码清单17-4，我们在其中打印出单个 URL 的页面标题（一旦它解析），Rust 将其编译为类似（虽然不完全）这样的内容：

```rust,ignore
match page_title(url).poll() {
    Ready(page_title) => match page_title {
        Some(title) => println!("The title for {url} was {title}"),
        None => println!("{url} had no title"),
    }
    Pending => {
        // 但是这里放什么？
    }
}
```

当 future 仍然是 `Pending` 时，我们应该做什么？我们需要某种方式再次尝试，一次又一次，直到 future 最终准备好。换句话说，我们需要一个循环：

```rust,ignore
let mut page_title_fut = page_title(url);
loop {
    match page_title_fut.poll() {
        Ready(value) => match page_title {
            Some(title) => println!("The title for {url} was {title}"),
            None => println!("{url} had no title"),
        }
        Pending => {
            // continue
        }
    }
}
```

但是，如果 Rust 将其编译为完全相同的代码，那么每个 `await` 都会阻塞——完全与我们想要做的相反！相反，Rust 确保循环可以将控制权移交给可以暂停这个 future 的工作以处理其他 futures，然后稍后再检查这个 future 的东西。正如我们所看到的，那个东西是一个 async 运行时，这种调度和协调工作是它的主要工作之一。

在[“使用消息传递在两个任务之间发送数据”][message-passing]<!-- ignore -->部分中，我们描述了等待 `rx.recv`。`recv` 调用返回一个 future，等待 future 会轮询它。我们注意到运行时将暂停 future，直到它准备好 `Some(message)` 或 `None`（当 channel 关闭时）。通过我们对 `Future` trait 的更深理解，特别是 `Future::poll`，我们可以看到这是如何工作的。运行时知道当它返回 `Poll::Pending` 时 future 还没有准备好。相反，运行时知道当 `poll` 返回 `Poll::Ready(Some(message))` 或 `Poll::Ready(None)` 时 future _已经_准备好并推进它。

运行时如何做到这一点的确切细节超出了本书的范围，但关键是了解 futures 的基本机制：运行时_轮询_它负责的每个 future，当它还没有准备好时让 future 回到睡眠状态。

<!-- Old headings. Do not remove or links may break. -->

<a id="pinning-and-the-pin-and-unpin-traits"></a>
<a id="the-pin-and-unpin-traits"></a>

### `Pin` 类型和 `Unpin` Trait

回到代码清单17-13，我们使用 `trpl::join!` 宏来等待三个 futures。但是，通常有一个集合（如向量）包含一些在运行时之前不知道数量的 futures。让我们将代码清单17-13更改为代码清单17-23中的代码，该代码将三个 futures 放入向量中并调用 `trpl::join_all` 函数，它还不能编译。

<Listing number="17-23" caption="等待集合中的 futures"  file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-23/src/main.rs:here}}
```

</Listing>

我们将每个 future 放在 `Box` 中，使它们成为_特征对象_，就像我们在第12章的“从 `run` 返回错误”部分中所做的那样。（我们将在第18章详细介绍特征对象。）使用特征对象让我们可以将这些类型产生的每个匿名 future 视为相同类型，因为它们都实现了 `Future` trait。

这可能令人惊讶。毕竟，这些 async 块都没有返回任何内容，所以每个都产生一个 `Future<Output = ()>`。但请记住，`Future` 是一个 trait，即使它们具有相同的输出类型，编译器也会为每个 async 块创建一个唯一的枚举。就像你不能将两个不同的手写结构体放入 `Vec` 一样，你不能混合编译器生成的枚举。

然后我们将 futures 集合传递给 `trpl::join_all` 函数并等待结果。但是，这不会编译；这是错误消息的相关部分。

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-23
cargo build
copy *only* the final `error` block from the errors
-->

```text
error[E0277]: `dyn Future<Output = ()>` cannot be unpinned
  --> src/main.rs:48:33
   |
48 |         trpl::join_all(futures).await;
   |                                 ^^^^^ the trait `Unpin` is not implemented for `dyn Future<Output = ()>`
   |
   = note: consider using the `pin!` macro
           consider using `Box::pin` if you need to access the pinned value outside of the current scope
   = note: required for `Box<dyn Future<Output = ()>>` to implement `Future`
note: required by a bound in `futures_util::future::join_all::JoinAll`
  --> file:///home/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/futures-util-0.3.30/src/future/join_all.rs:29:8
   |
27 | pub struct JoinAll<F>
   |            ------- required by a bound in this struct
28 | where
29 |     F: Future,
   |        ^^^^^^ required by this bound in `JoinAll`
```

此错误消息中的注释告诉我们应该使用 `pin!` 宏来_固定_值，这意味着将它们放入 `Pin` 类型中，该类型保证值不会在内存中移动。错误消息说固定是必需的，因为 `dyn Future<Output = ()>` 需要实现 `Unpin` trait，而它目前没有实现。

`trpl::join_all` 函数返回一个名为 `JoinAll` 的结构体。该结构体在类型 `F` 上是泛型的，`F` 被约束为实现 `Future` trait。直接用 `await` 等待 future 会隐式固定 future。这就是为什么我们不需要在每个要等待 future 的地方使用 `pin!`。

但是，我们这里不是直接等待 future。相反，我们通过将 futures 集合传递给 `join_all` 函数来构造一个新的 future JoinAll。`join_all` 的签名要求集合中项目的类型都实现 `Future` trait，并且 `Box<T>` 只有在它包装的 `T` 是实现 `Unpin` trait 的 future 时才实现 `Future`。

这需要吸收很多内容！为了真正理解它，让我们更深入地了解 `Future` trait 实际如何工作，特别是围绕固定。再次查看 `Future` trait 的定义：

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    // Required method
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

`cx` 参数及其 `Context` 类型是运行时实际知道何时检查任何给定 future 的关键，同时仍然保持惰性。同样，这如何工作的细节超出了本章的范围，当你编写自定义 `Future` 实现时通常只需要考虑这一点。我们将专注于 `self` 的类型，因为这是我们第一次看到 `self` 有类型注解的方法。`self` 的类型注解与其他函数参数的类型注解一样工作，但有两个关键差异：

- 它告诉 Rust `self` 必须是什么类型才能调用该方法。
- 它不能是任何类型。它被限制为该方法在其上实现的类型、对该类型的引用或智能指针，或者包装对该类型的引用的 `Pin`。

我们将在[第18章][ch-18]<!-- ignore -->中看到更多关于此语法的内容。现在，知道如果我们想轮询 future 以检查它是 `Pending` 还是 `Ready(Output)`，我们需要一个 `Pin` 包装的可变引用到该类型就足够了。

`Pin` 是类似于 `&`、`&mut`、`Box` 和 `Rc` 的指针类型的包装器。（从技术上讲，`Pin` 适用于实现 `Deref` 或 `DerefMut` traits 的类型，但这实际上等同于仅使用引用和智能指针。）`Pin` 本身不是指针，也没有像 `Rc` 和 `Arc` 那样自己的行为（带有引用计数）；它纯粹是编译器可以用来强制执行指针使用约束的工具。

回想一下 `await` 是用调用 `poll` 来实现的，这开始解释了我们在之前看到的错误消息，但那是关于 `Unpin` 的，而不是 `Pin`。那么 `Pin` 究竟如何与 `Unpin` 相关，以及为什么 `Future` 需要 `self` 在 `Pin` 类型中才能调用 `poll`？

记住本章前面，future 中的一系列等待点被编译成状态机，编译器确保该状态机遵循所有 Rust 的正常规则，包括借用和所有权。为了使其工作，Rust 查看在一个等待点和下一个等待点或 async 块的结束之间需要什么数据。然后它创建相应的变体在编译的状态机中。每个变体获得它需要的访问权限，用于在该源代码部分中使用的数据，无论是通过获取该数据的所有权还是通过获取可变或不可变引用。

到目前为止，一切都很好：如果我们在给定 async 块中关于所有权或引用的任何内容出错，借用检查器会告诉我们。当我们想要移动对应于该块的 future——比如将其移动到 `Vec` 以传递给 `join_all`——事情变得棘手。

当我们移动 future——无论是通过将其推入数据结构以与 `join_all` 一起用作迭代器，还是通过从函数返回它——这实际上意味着移动 Rust 为我们创建的状态机。与 Rust 中的大多数其他类型不同，Rust 为 async 块创建的 futures 最终可能在任何给定变体的字段中具有对自身的引用，如简化的图17-4所示。

<figure>

<img alt="A single-column, three-row table representing a future, fut1, which has data values 0 and 1 in the first two rows and an arrow pointing from the third row back to the second row, representing an internal reference within the future." src="img/trpl17-04.svg" class="center" />

<figcaption>图17-4：自引用数据类型</figcaption>

</figure>

但是，默认情况下，任何具有对自身引用的对象都是不安全的移动，因为引用总是指向它们引用的任何内容的实际内存地址（见图17-5）。如果你移动数据结构本身，那些内部引用将留下指向旧位置。但是，该内存位置现在无效。一方面，它的值不会在你对数据结构进行更改时更新。另一方面——更重要的是——计算机现在可以自由地将该内存用于其他目的！你最终可能会读取完全不相关的数据。

<figure>

<img alt="Two tables, depicting two futures, fut1 and fut2, each of which has one column and three rows, representing the result of having moved a future out of fut1 into fut2. The first, fut1, is grayed out, with a question mark in each index, representing unknown memory. The second, fut2, has 0 and 1 in the first and second rows and an arrow pointing from its third row back to the second row of fut1, representing a pointer that is referencing the old location in memory of the future before it was moved." src="img/trpl17-05.svg" class="center" />

<figcaption>图17-5：移动自引用数据类型的不安全结果</figcaption>

</figure>

理论上，Rust 编译器可以在对象移动时尝试更新每个引用，但这可能会增加大量性能开销，特别是如果需要更新整个引用网络。如果我们能够确保所讨论的数据结构_不会在内存中移动_，我们就不需要更新任何引用。这正是 Rust 的借用检查器的作用：在安全代码中，它防止你移动任何具有活动引用的项目。

`Pin` 在此基础上为我们提供了我们需要的精确保证。当我们通过将指向该值的指针包装在 `Pin` 中来_固定_一个值时，它就不能再移动了。因此，如果你有 `Pin<Box<SomeType>>`，你实际上固定的是 `SomeType` 值，_不是_ `Box` 指针。图17-6说明了这个过程。

<figure>

<img alt="Three boxes laid out side by side. The first is labeled "Pin", the second "b1", and the third "pinned". Within "pinned" is a table labeled "fut", with a single column; it represents a future with cells for each part of the data structure. Its first cell has the value "0", its second cell has an arrow coming out of it and pointing to the fourth and final cell, which has the value "1" in it, and the third cell has dashed lines and an ellipsis to indicate there may be other parts to the data structure. All together, the "fut" table represents a future which is self-referential. An arrow leaves the box labeled "Pin", goes through the box labeled "b1" and terminates inside the "pinned" box at the "fut" table." src="img/trpl17-06.svg" class="center" />

<figcaption>图17-6：固定指向自引用 future 类型的 `Box`</figcaption>

</figure>

事实上，`Box` 指针仍然可以自由移动。记住：我们关心的是确保最终被引用的数据保持在原位。如果指针移动，_但它指向的数据_在同一位置，如图17-7所示，没有潜在问题。（作为独立练习，查看类型以及 `std::pin` 模块的文档，并尝试弄清楚如何使用包装 `Box` 的 `Pin` 来实现这一点。）关键是自引用类型本身不能移动，因为它仍然被固定。

<figure>

<img alt="Four boxes laid out in three rough columns, identical to the previous diagram with a change to the second column. Now there are two boxes in the second column, labeled "b1" and "b2", "b1" is grayed out, and the arrow from "Pin" goes through "b2" instead of "b1", indicating that the pointer has moved from "b1" to "b2", but the data in "pinned" has not moved." src="img/trpl17-07.svg" class="center" />

<figcaption>图17-7：移动指向自引用 future 类型的 `Box`</figcaption>

</figure>

但是，大多数类型完全可以安全地移动，即使它们碰巧在 `Pin` 指针后面。我们只需要在项目具有内部引用时考虑固定。像数字和布尔值这样的原始值是安全的，因为它们显然没有任何内部引用。你通常在 Rust 中使用的大多数类型也没有。例如，你可以移动 `Vec` 而不用担心。根据我们到目前为止看到的情况，如果你有一个 `Pin<Vec<String>>`，你必须通过 `Pin` 提供的安全但受限制的 API 完成所有操作，即使 `Vec<String>` 在没有其他引用的情况下总是可以安全移动。我们需要一种方法来告诉编译器在这种情况下移动项目是可以的——这就是 `Unpin` 发挥作用的地方。

`Unpin` 是一个标记 trait，类似于我们在第16章中看到的 `Send` 和 `Sync` traits，因此它本身没有功能。标记 traits 的存在只是为了告诉编译器在特定上下文中使用实现给定 trait 的类型是安全的。`Unpin` 告知编译器给定类型_不需要_维护关于所讨论的值是否可以安全移动的任何保证。

<!--
  The inline `<code>` in the next block is to allow the inline `<em>` inside it,
  matching what NoStarch does style-wise, and emphasizing within the text here
  that it is something distinct from a normal type.
-->

就像 `Send` 和 `Sync` 一样，编译器会自动为所有它可以证明是安全的类型实现 `Unpin`。一个特殊情况，再次类似于 `Send` 和 `Sync`，是 `Unpin` 不_为类型实现的地方。这个表示法是 <code>impl !Unpin for <em>SomeType</em></code>，其中 <code><em>SomeType</em></code> 是_确实_需要维护这些保证的类型的名称，以便在 `Pin` 中使用指向该类型的指针时是安全的。

换句话说，关于 `Pin` 和 `Unpin` 之间的关系，有两件事要记住。首先，`Unpin` 是“正常”情况，`!Unpin` 是特殊情况。其次，类型是否实现 `Unpin` 或 `!Unpin` _仅_在你使用固定指针到该类型（如 <code>Pin<&mut <em>SomeType</em>></code>）时才重要。

为了使其具体化，考虑一个 `String`：它有一个长度和组成它的 Unicode 字符。我们可以将 `String` 包装在 `Pin` 中，如图17-8所示。但是，`String` 自动实现 `Unpin`，就像 Rust 中的大多数其他类型一样。

<figure>

<img alt="A box labeled "Pin" on the left with an arrow going from it to a box labeled "String" on the right. The "String" box contains the data 5usize, representing the length of the string, and the letters "h", "e", "l", "l", and "o" representing the characters of the string "hello" stored in this String instance. A dotted rectangle surrounds the "String" box and its label, but not the "Pin" box." src="img/trpl17-08.svg" class="center" />

<figcaption>图17-8：固定 `String`；虚线表示 `String` 实现 `Unpin` trait，因此未被固定</figcaption>

</figure>

因此，我们可以做一些如果 `String` 实现 `!Unpin` 而不是 `Unpin` 将会是非法的操作，例如在内存中的完全相同位置用另一个字符串替换一个字符串，如图17-9所示。这不会违反 `Pin` 契约，因为 `String` 没有使其不安全移动的内部引用。这正是为什么它实现 `Unpin` 而不是 `!Unpin` 的原因。

<figure>

<img alt="The same "hello" string data from the previous example, now labeled "s1" and grayed out. The "Pin" box from the previous example now points to a different String instance, one that is labeled "s2", is valid, has a length of 7usize, and contains the characters of the string "goodbye". s2 is surrounded by a dotted rectangle because it, too, implements the Unpin trait." src="img/trpl17-09.svg" class="center" />

<figcaption>图17-9：用完全不同的 `String` 替换内存中的 `String`</figcaption>

</figure>

现在我们知道足够了解从之前的代码清单17-23中的 `join_all` 调用报告的错误。我们最初尝试将由 async 块产生的 futures 移动到 `Vec<Box<dyn Future<Output = ()>>>` 中，但正如我们所看到的，这些 futures 可能具有内部引用，因此它们不会自动实现 `Unpin`。一旦我们固定它们，我们可以将结果 `Pin` 类型传递给 `Vec`，确信 futures 中的底层数据将_不会_被移动。代码清单17-24显示了如何通过在定义三个 futures 中的每一个时调用 `pin!` 宏并调整特征对象类型来修复代码。

<Listing number="17-24" caption="固定 futures 以启用将它们移动到向量中">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-24/src/main.rs:here}}
```

</Listing>

这个示例现在可以编译和运行，我们可以在运行时添加或删除 futures 向量并连接它们。

`Pin` 和 `Unpin` 主要用于构建低级库，或者当你自己构建运行时，而不是用于日常 Rust 代码。但是，当你在错误消息中看到这些 traits 时，现在你将更好地了解如何修复你的代码！

> 注意：`Pin` 和 `Unpin` 的这种组合使得在 Rust 中安全实现整个复杂类型类成为可能，否则它们会很有挑战性，因为它们是自引用的。需要 `Pin` 的类型今天在 async Rust 中最常见，但偶尔，你可能也会在其他上下文中看到它们。
>
> `Pin` 和 `Unpin` 如何工作的具体细节，以及它们需要维护的规则，在 `std::pin` 的 API 文档中有详细说明，所以如果你有兴趣了解更多，那是一个很好的起点。
>
> 如果你想更详细地了解底层的工作原理，请参阅 [_Rust 中的异步编程_][async-book] 的[第2章][under-the-hood]<!-- ignore -->和[第4章][pinning]<!-- ignore -->。

### `Stream` Trait

现在你对 `Future`、`Pin` 和 `Unpin` traits 有了更深入的了解，我们可以将注意力转向 `Stream` trait。正如你在本章前面学到的，流类似于异步迭代器。然而，与 `Iterator` 和 `Future` 不同，`Stream` 在撰写本文时在标准库中没有定义，但 `futures` crate 中有一个非常常见的定义，在整个生态系统中使用。

让我们在查看 `Stream` trait 如何将它们合并在一起之前，先回顾一下 `Iterator` 和 `Future` traits 的定义。从 `Iterator`，我们有序列的概念：它的 `next` 方法提供 `Option<Self::Item>`。从 `Future`，我们有随时间准备就绪的概念：它的 `poll` 方法提供 `Poll<Self::Output>`。为了表示随时间变得准备好的项目序列，我们定义一个 `Stream` trait，将这些特征组合在一起：

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

trait Stream {
    type Item;

    fn poll_next(
        self: Pin<&mut Self>,
        cx: &mut Context<'_>
    ) -> Poll<Option<Self::Item>>;
}
```

`Stream` trait 定义了一个名为 `Item` 的关联类型，用于流产生的项目类型。这类似于 `Iterator`，其中可能有零到多个项目，并且与 `Future` 不同，`Future` 总是有单个 `Output`，即使它是单元类型 `()`。

`Stream` 还定义了一个方法来获取这些项目。我们称之为 `poll_next`，以明确它像 `Future::poll` 一样轮询，并像 `Iterator::next` 一样产生项目序列。它的返回类型将 `Poll` 与 `Option` 组合在一起。外部类型是 `Poll`，因为它必须检查是否准备好，就像 future 一样。内部类型是 `Option`，因为它需要信号是否有更多消息，就像迭代器一样。

类似于此定义的某些内容最终可能会成为 Rust 标准库的一部分。与此同时，它是大多数运行时工具包的一部分，所以你可以依赖它，我们接下来涵盖的所有内容通常都应该适用！

然而，在我们看到的[“Streams：按顺序的 Futures”][streams]<!-- ignore -->部分的示例中，我们没有使用 `poll_next` _或_ `Stream`，而是使用了 `next` 和 `StreamExt`。我们_可以_直接使用 `poll_next` API 通过手动编写我们自己的 `Stream` 状态机来工作，当然，就像我们_可以_通过它们的 `poll` 方法直接使用 futures 一样。但是使用 `await` 要好得多，`StreamExt` trait 提供了 `next` 方法，所以我们可以这样做：

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-stream-ext/src/lib.rs:here}}
```

<!--
TODO: update this if/when tokio/etc. update their MSRV and switch to using async functions
in traits, since the lack thereof is the reason they do not yet have this.
-->

> 注意：我们在本章前面使用的实际定义看起来与此略有不同，因为它支持尚未支持在 traits 中使用 async 函数的 Rust 版本。因此，它看起来像这样：
>
> ```rust,ignore
> fn next(&mut self) -> Next<'_, Self> where Self: Unpin;
> ```
>
> 那个 `Next` 类型是一个实现 `Future` 的 `struct`，并允许我们用 `Next<'_, Self>` 命名对 `self` 的引用的生命周期，以便 `await` 可以与该方法一起工作。

`StreamExt` trait 也是所有可用于流的有趣方法的家园。`StreamExt` 自动为每个实现 `Stream` 的类型实现，但这些 traits 是分开定义的，以使社区能够迭代便利 API，而不影响基础 trait。

在 `trpl` crate 中使用的 `StreamExt` 版本中，trait 不仅定义了 `next` 方法，还提供了 `next` 的默认实现，它正确处理调用 `Stream::poll_next` 的细节。这意味着即使你需要编写自己的流式数据类型，你_只需要_实现 `Stream`，然后任何使用你的数据类型的人都可以自动使用 `StreamExt` 及其方法。

这就是我们要涵盖的这些 traits 的底层细节的全部内容。总结一下，让我们考虑 futures（包括 streams）、tasks 和 threads 如何组合在一起！

[message-passing]: ch17-02-concurrency-with-async.md#sending-data-between-two-tasks-using-message-passing
[ch-18]: ch18-00-oop.html
[async-book]: https://rust-lang.github.io/async-book/
[under-the-hood]: https://rust-lang.github.io/async-book/02_execution/01_chapter.html
[pinning]: https://rust-lang.github.io/async-book/04_pinning/01_chapter.html
[first-async]: ch17-01-futures-and-syntax.html#our-first-async-program
[any-number-futures]: ch17-03-more-futures.html#working-with-any-number-of-futures
[streams]: ch17-04-streams.html
