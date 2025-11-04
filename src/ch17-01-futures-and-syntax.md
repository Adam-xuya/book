## Futures 和 Async 语法

Rust 中异步编程的关键元素是_futures_和 Rust 的 `async` 和 `await` 关键字。

一个_future_是一个可能现在还没有准备好，但会在将来的某个时刻准备好的值。（这个概念在许多语言中都会出现，有时使用其他名称，如_task_或_promise_。）Rust 提供了 `Future` trait 作为构建块，以便不同的异步操作可以用不同的数据结构实现，但具有通用接口。在 Rust 中，futures 是实现 `Future` trait 的类型。每个 future 都持有关于已取得的进展以及“准备好”意味着什么的信息。

你可以将 `async` 关键字应用于块和函数，以指定它们可以被中断和恢复。在 async 块或 async 函数中，你可以使用 `await` 关键字来_等待一个 future_（即，等待它变得准备好）。在 async 块或函数中等待 future 的任何点都是该块或函数可以暂停和恢复的潜在位置。检查 future 以查看其值是否可用的过程称为_轮询_。

其他一些语言，如 C# 和 JavaScript，也使用 `async` 和 `await` 关键字进行异步编程。如果你熟悉这些语言，你可能会注意到 Rust 处理语法的方式有一些显著差异。这是有充分理由的，正如我们将看到的！

在编写 async Rust 时，我们大部分时间都使用 `async` 和 `await` 关键字。Rust 将它们编译成使用 `Future` trait 的等效代码，就像它将 `for` 循环编译成使用 `Iterator` trait 的等效代码一样。然而，因为 Rust 提供了 `Future` trait，当你需要时，你也可以为自己的数据类型实现它。我们在本章中看到的许多函数返回具有自己的 `Future` 实现的类型。我们将在本章末尾回到 trait 的定义，并深入了解它的工作原理，但这足以让我们继续前进。

这可能都感觉有点抽象，所以让我们编写我们的第一个 async 程序：一个小型网络爬虫。我们将从命令行传入两个 URL，并发地获取它们，并返回先完成的结果。这个例子会有相当多的新语法，但不用担心——我们会解释你需要的所有知识。

## 我们的第一个 Async 程序

为了保持本章的重点在学习 async 而不是管理生态系统的各个部分，我们创建了 `trpl` crate（`trpl` 是 "The Rust Programming Language" 的缩写）。它重新导出了你将需要的所有类型、trait 和函数，主要来自 [`futures`][futures-crate]<!-- ignore --> 和 [`tokio`][tokio]<!-- ignore --> crate。`futures` crate 是 Rust 异步代码实验的官方家园，实际上 `Future` trait 最初就是在这里设计的。Tokio 是当今 Rust 中使用最广泛的 async 运行时，特别是对于 Web 应用程序。还有其他优秀的运行时，它们可能更适合你的目的。我们在 `trpl` 的底层使用 `tokio` crate，因为它经过充分测试且广泛使用。

在某些情况下，`trpl` 还会重命名或包装原始 API，以让你专注于本章相关的细节。如果你想了解这个 crate 的功能，我们鼓励你查看[其源代码][crate-source]。你将能够看到每个重新导出来自哪个 crate，我们留下了大量注释来解释这个 crate 的功能。

创建一个名为 `hello-async` 的新二进制项目，并将 `trpl` crate 添加为依赖：

```console
$ cargo new hello-async
$ cd hello-async
$ cargo add trpl
```

现在我们可以使用 `trpl` 提供的各种部分来编写我们的第一个 async 程序。我们将构建一个小的命令行工具，它获取两个网页，从每个页面中提取 `<title>` 元素，并打印出先完成整个过程的页面的标题。

### 定义 page_title 函数

让我们首先编写一个函数，它接受一个页面 URL 作为参数，向其发出请求，并返回 `<title>` 元素的文本（见代码清单17-1）。

<Listing number="17-1" file-name="src/main.rs" caption="定义一个 async 函数以从 HTML 页面获取 title 元素">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-01/src/main.rs:all}}
```

</Listing>

首先，我们定义一个名为 `page_title` 的函数，并用 `async` 关键字标记它。然后我们使用 `trpl::get` 函数获取传入的任何 URL，并添加 `await` 关键字来等待响应。为了获取 `response` 的文本，我们调用其 `text` 方法，并再次使用 `await` 关键字等待它。这两个步骤都是异步的。对于 `get` 函数，我们必须等待服务器发送回其响应的第一部分，这将包括 HTTP 标头、cookie 等，并且可以与响应体分开传递。特别是如果主体非常大，所有数据到达可能需要一些时间。因为我们必须等待响应的_全部_到达，所以 `text` 方法也是 async 的。

我们必须显式地等待这两个 futures，因为 Rust 中的 futures 是_惰性的_：它们在你用 `await` 关键字询问它们之前不会做任何事情。（事实上，如果你不使用 future，Rust 会显示编译器警告。）这可能会让你想起第13章中[“使用迭代器处理一系列项目”][iterators-lazy]<!-- ignore -->部分关于迭代器的讨论。迭代器除非你调用它们的 `next` 方法——无论是直接调用还是通过使用 `for` 循环或使用 `next` 底层的方法（如 `map`）——否则什么也不做。同样，futures 除非你显式询问它们，否则什么也不做。这种惰性允许 Rust 避免在实际需要之前运行 async 代码。

> 注意：这与我们在第16章的[“使用 spawn 创建新线程”][thread-spawn]<!-- ignore -->部分中使用 `thread::spawn` 时看到的行为不同，在那里我们传递给另一个线程的闭包立即开始运行。它也与许多其他语言处理 async 的方式不同。但对于 Rust 能够提供其性能保证来说，这很重要，就像迭代器一样。

一旦我们有了 `response_text`，我们可以使用 `Html::parse` 将其解析为 `Html` 类型的实例。我们现在有一个数据类型，可以用来将 HTML 作为更丰富的数据结构进行处理，而不是原始字符串。特别是，我们可以使用 `select_first` 方法来查找给定 CSS 选择器的第一个实例。通过传递字符串 `"title"`，我们将获得文档中的第一个 `<title>` 元素（如果有的话）。因为可能没有任何匹配的元素，`select_first` 返回一个 `Option<ElementRef>`。最后，我们使用 `Option::map` 方法，它让我们在 `Option` 中存在项目时处理它，如果不存在则什么也不做。（我们也可以在这里使用 `match` 表达式，但 `map` 更符合习惯。）在我们提供给 `map` 的函数体中，我们在 `title` 上调用 `inner_html` 以获取其内容，这是一个 `String`。当一切完成时，我们有一个 `Option<String>`。

请注意，Rust 的 `await` 关键字位于你要等待的表达式_之后_，而不是之前。也就是说，它是一个_后缀_关键字。如果你在其他语言中使用过 `async`，这可能与你习惯的不同，但在 Rust 中，它使方法链更容易使用。因此，我们可以将 `page_title` 的主体更改为将 `trpl::get` 和 `text` 函数调用与它们之间的 `await` 链接在一起，如代码清单17-2所示。

<Listing number="17-2" file-name="src/main.rs" caption="使用 `await` 关键字链接">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-02/src/main.rs:chaining}}
```

</Listing>

这样，我们就成功编写了我们的第一个 async 函数！在我们在 `main` 中添加一些代码来调用它之前，让我们更多地讨论一下我们编写的内容及其含义。

当 Rust 看到一个用 `async` 关键字标记的_块_时，它将其编译成实现 `Future` trait 的唯一匿名数据类型。当 Rust 看到一个用 `async` 标记的_函数_时，它将其编译成一个非 async 函数，其主体是一个 async 块。async 函数的返回类型是编译器为该 async 块创建的匿名数据类型的类型。

因此，编写 `async fn` 等价于编写一个返回返回类型的_future_的函数。对于编译器来说，像代码清单17-1中的 `async fn page_title` 这样的函数定义大致等价于像这样定义的非 async 函数：

```rust
# extern crate trpl; // required for mdbook test
use std::future::Future;
use trpl::Html;

fn page_title(url: &str) -> impl Future<Output = Option<String>> {
    async move {
        let text = trpl::get(url).await.text().await;
        Html::parse(&text)
            .select_first("title")
            .map(|title| title.inner_html())
    }
}
```

让我们逐步了解转换后的版本的每个部分：

- 它使用我们在第10章的[“作为参数的 Traits”][impl-trait]<!-- ignore -->部分中讨论的 `impl Trait` 语法。
- 返回的值实现了具有关联类型 `Output` 的 `Future` trait。请注意，`Output` 类型是 `Option<String>`，这与 `async fn` 版本的 `page_title` 的原始返回类型相同。
- 原始函数体中调用的所有代码都包装在一个 `async move` 块中。记住，块是表达式。这整个块是从函数返回的表达式。
- 这个 async 块产生一个类型为 `Option<String>` 的值，正如刚才描述的。该值与返回类型中的 `Output` 类型匹配。这就像你看到的其他块一样。
- 新函数主体是一个 `async move` 块，因为它如何使用 `url` 参数。（我们将在本章后面更多地讨论 `async` 与 `async move`。）

现在我们可以在 `main` 中调用 `page_title`。

<!-- Old headings. Do not remove or links may break. -->

<a id ="determining-a-single-pages-title"></a>

### 使用运行时执行 Async 函数

首先，我们将获取单个页面的标题，如代码清单17-3所示。不幸的是，这段代码还不能编译。

<Listing number="17-3" file-name="src/main.rs" caption="从 `main` 调用 `page_title` 函数，使用用户提供的参数">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-03/src/main.rs:main}}
```

</Listing>

我们遵循与在第12章的[“接受命令行参数”][cli-args]<!-- ignore -->部分中获取命令行参数相同的模式。然后我们将 URL 参数传递给 `page_title` 并等待结果。因为 future 产生的值是 `Option<String>`，我们使用 `match` 表达式打印不同的消息，以说明页面是否有 `<title>`。

我们可以使用 `await` 关键字的唯一位置是在 async 函数或块中，而 Rust 不允许我们将特殊的 `main` 函数标记为 `async`。

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-03
cargo build
copy just the compiler error
-->

```text
error[E0752]: `main` function is not allowed to be `async`
 --> src/main.rs:6:1
  |
6 | async fn main() {
  | ^^^^^^^^^^^^^^^ `main` function is not allowed to be `async`
```

`main` 不能标记为 `async` 的原因是 async 代码需要一个_运行时_：一个管理执行异步代码细节的 Rust crate。程序的 `main` 函数可以_初始化_一个运行时，但它本身不是一个运行时_本身_。（我们稍后会看到更多关于为什么是这样的原因。）每个执行 async 代码的 Rust 程序至少有一个地方它设置一个运行时来执行 futures。

大多数支持 async 的语言都捆绑了一个运行时，但 Rust 没有。相反，有许多不同的 async 运行时可用，每个运行时都针对其目标用例做出不同的权衡。例如，具有许多 CPU 核心和大量 RAM 的高吞吐量 Web 服务器与具有单个核心、少量 RAM 且没有堆分配能力的微控制器有非常不同的需求。提供这些运行时的 crate 通常还提供常见功能的 async 版本，例如文件或网络 I/O。

在这里，以及在本章其余部分，我们将使用 `trpl` crate 中的 `block_on` 函数，它接受一个 future 作为参数，并阻塞当前线程，直到这个 future 运行完成。在幕后，调用 `block_on` 使用 `tokio` crate 设置一个运行时，用于运行传入的 future（`trpl` crate 的 `block_on` 行为与其他运行时 crate 的 `block_on` 函数类似）。一旦 future 完成，`block_on` 返回 future 产生的任何值。

我们可以将 `page_title` 返回的 future 直接传递给 `block_on`，一旦它完成，我们可以匹配结果 `Option<String>`，就像我们在代码清单17-3中尝试做的那样。然而，对于本章中的大多数示例（以及现实世界中的大多数 async 代码），我们将做的不仅仅是单个 async 函数调用，所以我们将传递一个 `async` 块并显式等待 `page_title` 调用的结果，如代码清单17-4所示。

<Listing number="17-4" caption="使用 `trpl::block_on` 等待 async 块" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook test does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-04/src/main.rs:run}}
```

</Listing>

当我们运行这段代码时，我们得到了最初预期的行为：

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-04
cargo build # skip all the build noise
cargo run -- "https://www.rust-lang.org"
# copy the output here
-->

```console
$ cargo run -- "https://www.rust-lang.org"
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.05s
     Running `target/debug/async_await 'https://www.rust-lang.org'`
The title for https://www.rust-lang.org was
            Rust Programming Language
```

太好了——我们终于有了一些可工作的 async 代码！但在我们添加代码来让两个网站相互竞争之前，让我们简要地将注意力转回 futures 的工作原理。

每个_等待点_——即代码使用 `await` 关键字的每个地方——代表一个将控制权交还给运行时的位置。为了使这工作，Rust 需要跟踪 async 块中涉及的状态，以便运行时可以启动一些其他工作，然后在准备好再次尝试推进第一个工作时回来。这是一个不可见的状态机，就像你编写一个像这样的枚举来在每个等待点保存当前状态：

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-state-machine/src/lib.rs:enum}}
```

手动编写代码来在每个状态之间转换将是乏味且容易出错的，特别是当你需要稍后添加更多功能和更多状态到代码时。幸运的是，Rust 编译器自动为 async 代码创建和管理状态机数据结构。围绕数据结构的正常借用和所有权规则仍然适用，令人高兴的是，编译器也为我们处理检查这些并提供有用的错误消息。我们将在本章后面处理其中的一些。

最终，必须有一些东西来执行这个状态机，那个东西就是一个运行时。（这就是为什么你在研究运行时可能会遇到_executors_的提及：executor 是运行时负责执行 async 代码的部分。）

现在你可以看到为什么编译器在代码清单17-3中阻止我们将 `main` 本身设为 async 函数。如果 `main` 是一个 async 函数，其他东西需要管理 `main` 返回的任何 future 的状态机，但 `main` 是程序的起点！相反，我们在 `main` 中调用 `trpl::block_on` 函数来设置运行时，并运行 `async` 块返回的 future，直到它完成。

> 注意：一些运行时提供宏，这样你_可以_编写一个 async `main` 函数。这些宏将 `async fn main() { ... }` 重写为普通的 `fn main`，它执行与我们在代码清单17-4中手动执行相同的操作：调用一个函数，以 `trpl::block_on` 的方式运行 future 直到完成。

现在让我们将这些部分组合在一起，看看如何编写并发代码。

<!-- Old headings. Do not remove or links may break. -->

<a id="racing-our-two-urls-against-each-other"></a>

### 并发地让两个 URL 相互竞争

在代码清单17-5中，我们使用从命令行传入的两个不同 URL 调用 `page_title`，并通过选择先完成的 future 来让它们竞争。

<Listing number="17-5" caption="为两个 URL 调用 `page_title` 以查看哪个先返回" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-05/src/main.rs:all}}
```

</Listing>

我们首先为每个用户提供的 URL 调用 `page_title`。我们将结果 futures 保存为 `title_fut_1` 和 `title_fut_2`。记住，这些现在还什么也不做，因为 futures 是惰性的，我们还没有等待它们。然后我们将 futures 传递给 `trpl::select`，它返回一个值来指示传递给它的哪个 future 先完成。

> 注意：在底层，`trpl::select` 建立在 `futures` crate 中定义的更通用的 `select` 函数之上。`futures` crate 的 `select` 函数可以做很多 `trpl::select` 函数无法做的事情，但它也有一些我们现在可以跳过的额外复杂性。

任何一个 future 都可以合法地“获胜”，所以返回 `Result` 没有意义。相反，`trpl::select` 返回一个我们之前没有见过的类型，`trpl::Either`。`Either` 类型在某种程度上类似于 `Result`，因为它有两个情况。然而，与 `Result` 不同，`Either` 中没有内置成功或失败的概念。相反，它使用 `Left` 和 `Right` 来表示“一个或另一个”：

```rust
enum Either<A, B> {
    Left(A),
    Right(B),
}
```

如果第一个参数获胜，`select` 函数返回 `Left` 和该 future 的输出，如果_那个_获胜，则返回 `Right` 和第二个 future 参数 outputs。这与调用函数时参数出现的顺序匹配：第一个参数在第二个参数的左侧。

我们还更新了 `page_title` 以返回传入的相同 URL。这样，如果先返回的页面没有我们可以解析的 `<title>`，我们仍然可以打印有意义的消息。有了这些信息，我们通过更新我们的 `println!` 输出来结束，以指示哪个 URL 先完成，以及该 URL 的网页（如果有的话）的 `<title>` 是什么。

你现在已经构建了一个小的可工作的网络爬虫！选择几个 URL 并运行命令行工具。你可能会发现某些网站始终比其他网站快，而在其他情况下，较快的网站会因运行而异。更重要的是，你已经学习了使用 futures 的基础知识，所以现在我们可以深入了解我们可以用 async 做什么。

[impl-trait]: ch10-02-traits.html#traits-as-parameters
[iterators-lazy]: ch13-02-iterators.html
[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[cli-args]: ch12-01-accepting-command-line-arguments.html

<!-- TODO: map source link version to version of Rust? -->

[crate-source]: https://github.com/rust-lang/book/tree/main/packages/trpl
[futures-crate]: https://crates.io/crates/futures
[tokio]: https://tokio.rs
