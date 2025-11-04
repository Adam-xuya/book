<!-- Old headings. Do not remove or links may break. -->

<a id="streams"></a>

## Streams：按顺序的 Futures

回想一下我们在本章前面在[“消息传递”][17-02-messages]<!-- ignore -->部分中如何使用我们的 async channel 的接收器。async `recv` 方法产生随时间推移的一系列项目。这是一个更通用模式的实例，称为_流_。许多概念自然地表示为流：队列中可用的项目，当完整数据集对于计算机的内存来说太大时从文件系统增量拉取的块数据，或随时间推移通过网络到达的数据。因为流是 futures，我们可以将它们与任何其他类型的 future 一起使用，并以有趣的方式组合它们。例如，我们可以批量处理事件以避免触发太多网络调用，在长时间运行的操作序列上设置超时，或限制用户界面事件以避免做不必要的工作。

当我们在第13章查看 Iterator trait 时，我们在[“Iterator Trait 和 `next` 方法”][iterator-trait]<!-- ignore -->部分中看到了一个项目序列，但迭代器和 async channel 接收器之间有两个差异。第一个差异是时间：迭代器是同步的，而 channel 接收器是异步的。第二个差异是 API。直接使用 `Iterator` 时，我们调用其同步 `next` 方法。特别是对于 `trpl::Receiver` 流，我们调用了一个异步 `recv` 方法。否则，这些 API 感觉非常相似，这种相似性并非巧合。流就像迭代的异步形式。然而，虽然 `trpl::Receiver` 专门等待接收消息，但通用流 API 要广泛得多：它提供下一个项目的方式与 `Iterator` 一样，但是异步的。

Rust 中迭代器和流之间的相似性意味着我们实际上可以从任何迭代器创建流。与迭代器一样，我们可以通过调用其 `next` 方法然后等待输出来处理流，如代码清单17-21所示，它还不能编译。

<Listing number="17-21" caption="从迭代器创建流并打印其值" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-21/src/main.rs:stream}}
```

</Listing>

我们从一个数字数组开始，我们将其转换为迭代器，然后在其上调用 `map` 以将所有值加倍。然后我们使用 `trpl::stream_from_iter` 函数将迭代器转换为流。接下来，我们使用 `while let` 循环遍历流中的项目，因为它们到达。

不幸的是，当我们尝试运行代码时，它不会编译，而是报告没有可用的 `next` 方法：

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-21
cargo build
copy only the error output
-->

```text
error[E0599]: no method named `next` found for struct `tokio_stream::iter::Iter` in the current scope
  --> src/main.rs:10:40
   |
10 |         while let Some(value) = stream.next().await {
   |                                        ^^^^
   |
   = help: items from traits can only be used if the trait is in scope
help: the following traits which provide `next` are implemented but not in scope; perhaps you want to import one of them
   |
1  + use crate::trpl::StreamExt;
   |
1  + use futures_util::stream::stream::StreamExt;
   |
1  + use std::iter::Iterator;
   |
1  + use std::str::pattern::Searcher;
   |
help: there is a method `try_next` with a similar name
   |
10 |         while let Some(value) = stream.try_next().await {
   |                                     ~~~~~~~~
```

正如这个输出所解释的，编译器错误的原因是我们需要正确的 trait 在作用域中才能使用 `next` 方法。根据我们到目前为止的讨论，你可能合理地期望该 trait 是 `Stream`，但它实际上是 `StreamExt`。`Ext` 是“扩展”的缩写，`Ext` 是 Rust 社区中用一个 trait 扩展另一个 trait 的常见模式。

`Stream` trait 定义了一个低级接口，有效地结合了 `Iterator` 和 `Future` traits。`StreamExt` 在 `Stream` 之上提供了一组更高级的 API，包括 `next` 方法以及类似于 `Iterator` trait 提供的其他实用方法。`Stream` 和 `StreamExt` 还不是 Rust 标准库的一部分，但大多数生态系统 crate 使用类似的定义。

编译器错误的修复是添加 `trpl::StreamExt` 的 `use` 语句，如代码清单17-22所示。

<Listing number="17-22" caption="成功使用迭代器作为流的基础" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-22/src/main.rs:all}}
```

</Listing>

当所有这些部分组合在一起时，这段代码按照我们想要的方式工作！更重要的是，现在我们在作用域中有 `StreamExt`，我们可以使用它的所有实用方法，就像使用迭代器一样。

[17-02-messages]: ch17-02-concurrency-with-async.html#message-passing
[iterator-trait]: ch13-02-iterators.html#the-iterator-trait-and-the-next-method
