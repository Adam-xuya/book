<!-- Old headings. Do not remove or links may break. -->

<a id="concurrency-with-async"></a>

## 使用 Async 应用并发

在本节中，我们将把 async 应用到一些我们在第16章中使用线程解决的相同并发挑战中。因为我们已经在那里讨论了很多关键思想，在本节中，我们将专注于线程和 futures 之间的不同之处。

在许多情况下，使用 async 处理并发的 API 与使用线程的 API 非常相似。在其他情况下，它们最终会非常不同。即使 API 在线程和 async 之间_看起来_相似，它们通常具有不同的行为——它们几乎总是具有不同的性能特征。

<!-- Old headings. Do not remove or links may break. -->

<a id="counting"></a>

### 使用 `spawn_task` 创建新任务

我们在第16章的[“使用 `spawn` 创建新线程”][thread-spawn]<!-- ignore -->部分中处理的第一个操作是在两个单独的线程上计数。让我们使用 async 做同样的事情。`trpl` crate 提供了一个 `spawn_task` 函数，它看起来与 `thread::spawn` API 非常相似，以及一个 `sleep` 函数，它是 `thread::sleep` API 的 async 版本。我们可以一起使用这些来实现计数示例，如代码清单17-6所示。

<Listing number="17-6" caption="创建新任务以在主任务打印某些内容的同时打印另一内容" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-06/src/main.rs:all}}
```

</Listing>

作为我们的起点，我们使用 `trpl::block_on` 设置我们的 `main` 函数，以便我们的顶层函数可以是 async 的。

> 注意：从本章的这一点开始，每个示例都将包含相同的 `trpl::block_on` 包装代码在 `main` 中，所以我们将经常跳过它，就像我们对 `main` 所做的那样。记住在你的代码中包含它！

然后我们在该块内编写两个循环，每个循环包含一个 `trpl::sleep` 调用，它在发送下一条消息之前等待半秒（500毫秒）。我们将一个循环放在 `trpl::spawn_task` 的主体中，另一个放在顶层 `for` 循环中。我们还在 `sleep` 调用之后添加一个 `await`。

这段代码的行为类似于基于线程的实现——包括你可能在自己的终端中看到消息以不同顺序出现的事实：

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
```

这个版本在 `main` async 块的主体中的 `for` 循环完成时立即停止，因为由 `spawn_task` 生成的任务在 `main` 函数结束时关闭。如果你希望它一直运行到任务完成，你需要使用 join handle 来等待第一个任务完成。使用线程时，我们使用 `join` 方法来“阻塞”，直到线程完成运行。在代码清单17-7中，我们可以使用 `await` 来做同样的事情，因为任务 handle 本身就是一个 future。它的 `Output` 类型是一个 `Result`，所以我们在等待它之后也 unwrap 它。

<Listing number="17-7" caption="使用 `await` 和 join handle 运行任务直到完成" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-07/src/main.rs:handle}}
```

</Listing>

这个更新版本运行直到_两个_循环都完成：

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

到目前为止，看起来 async 和线程给了我们相似的结果，只是语法不同：使用 `await` 而不是在 join handle 上调用 `join`，以及等待 `sleep` 调用。

更大的区别是我们不需要生成另一个操作系统线程来执行此操作。事实上，我们甚至不需要在这里生成任务。因为 async 块编译成匿名 futures，我们可以将每个循环放在一个 async 块中，并让运行时使用 `trpl::join` 函数运行它们直到完成。

在第16章的[“等待所有线程完成”][join-handles]<!-- ignore -->部分中，我们展示了如何在你调用 `std::thread::spawn` 时返回的 `JoinHandle` 类型上使用 `join` 方法。`trpl::join` 函数类似，但用于 futures。当你给它两个 futures 时，它产生一个单一的新 future，其输出是一个元组，包含一旦它们_都_完成时你传入的每个 future 的输出。因此，在代码清单17-8中，我们使用 `trpl::join` 等待 `fut1` 和 `fut2` 都完成。我们_不_等待 `fut1` 和 `fut2`，而是等待由 `trpl::join` 产生的新 future。我们忽略输出，因为它只是一个包含两个单元值的元组。

<Listing number="17-8" caption="使用 `trpl::join` 等待两个匿名 futures" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-08/src/main.rs:join}}
```

</Listing>

当我们运行这个时，我们看到两个 futures 都运行到完成：

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the first task!
hi number 1 from the second task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

现在，你将每次都看到完全相同的顺序，这与我们在线程和代码清单17-7中的 `trpl::spawn_task` 中看到的情况非常不同。这是因为 `trpl::join` 函数是_公平的_，意味着它同样频繁地检查每个 future，在它们之间交替，并且如果另一个准备就绪，永远不会让一个跑在前面。对于线程，操作系统决定检查哪个线程以及让它运行多长时间。对于 async Rust，运行时决定检查哪个任务。（在实践中，细节变得复杂，因为 async 运行时可能会在底层使用操作系统线程作为它管理并发的一部分，因此保证公平性可能对运行时来说需要更多工作——但它仍然是可能的！）运行时不必为任何给定操作保证公平性，它们通常提供不同的 API 来让你选择是否想要公平性。

尝试这些等待 futures 的变体，看看它们做什么：

- 从循环中移除一个或两个 async 块。
- 在定义后立即等待每个 async 块。
- 仅将第一个循环包装在 async 块中，并在第二个循环的主体之后等待结果 future。

作为额外的挑战，看看你是否能在运行代码_之前_弄清楚每种情况下的输出是什么！

<!-- Old headings. Do not remove or links may break. -->

<a id="message-passing"></a>
<a id="counting-up-on-two-tasks-using-message-passing"></a>

### 使用消息传递在两个任务之间发送数据

在 futures 之间共享数据也会很熟悉：我们将再次使用消息传递，但这次使用类型和函数的 async 版本。我们将采取与在第16章的[“使用消息传递在线程之间传输数据”][message-passing-threads]<!-- ignore -->部分中稍有不同的路径，以说明基于线程和基于 futures 的并发之间的一些关键差异。在代码清单17-9中，我们将从单个 async 块开始——_不_生成单独的任务，就像我们生成单独的线程一样。

<Listing number="17-9" caption="创建 async channel 并将两部分分配给 `tx` 和 `rx`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-09/src/main.rs:channel}}
```

</Listing>

在这里，我们使用 `trpl::channel`，这是我们在第16章中与线程一起使用的多生产者、单消费者 channel API 的 async 版本。async 版本的 API 与基于线程的版本只有一点不同：它使用可变而不是不可变的接收器 `rx`，并且其 `recv` 方法产生一个我们需要等待的 future，而不是直接产生值。现在我们可以从发送器向接收器发送消息。请注意，我们不必生成单独的线程甚至任务；我们只需要等待 `rx.recv` 调用。

`std::mpsc::channel` 中的同步 `Receiver::recv` 方法会阻塞直到它接收到消息。`trpl::Receiver::recv` 方法不会阻塞，因为它是 async 的。它不是阻塞，而是将控制权交还给运行时，直到接收到消息或 channel 的发送端关闭。相比之下，我们不等待 `send` 调用，因为它不会阻塞。它不需要阻塞，因为我们发送到的 channel 是无界的。

> 注意：因为所有这些 async 代码都在 `trpl::block_on` 调用中的 async 块内运行，所以其中的所有内容都可以避免阻塞。但是，它_之外的_代码将阻塞在 `block_on` 函数返回上。这就是 `trpl::block_on` 函数的全部要点：它让你_选择_在哪里阻塞某些 async 代码集，从而在哪里在同步和 async 代码之间转换。

请注意这个示例的两件事。首先，消息会立即到达。其次，尽管我们在这里使用 future，但还没有并发。列表中的所有内容都按顺序发生，就像没有涉及 futures 一样。

让我们通过发送一系列消息并在它们之间休眠来解决第一部分，如代码清单17-10所示。

<!-- We cannot test this one because it never stops! -->

<Listing number="17-10" caption="通过 async channel 发送和接收多条消息，并在每条消息之间使用 `await` 休眠" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-10/src/main.rs:many-messages}}
```

</Listing>

除了发送消息，我们还需要接收它们。在这种情况下，因为我们知道有多少消息要来，我们可以通过调用 `rx.recv().await` 四次来手动完成。但在现实世界中，我们通常会等待一些_未知_数量的消息，所以我们需要继续等待，直到我们确定没有更多消息。

在代码清单16-10中，我们使用 `for` 循环来处理从同步 channel 接收的所有项目。然而，Rust 还没有办法使用 `for` 循环与_异步产生_的项目序列，所以我们需要使用一个我们之前没有见过的循环：`while let` 条件循环。这是我们在第6章的[“使用 `if let` 和 `let else` 进行简洁控制流”][if-let]<!-- ignore -->部分中看到的 `if let` 构造的循环版本。只要它指定的模式继续匹配值，循环就会继续执行。

`rx.recv` 调用产生一个 future，我们等待它。运行时将暂停 future，直到它准备好。一旦消息到达，future 将解析为 `Some(message)`，到达多少次就解析多少次。当 channel 关闭时，无论_任何_消息是否已到达，future 将改为解析为 `None`，以指示没有更多值，因此我们应该停止轮询——即停止等待。

`while let` 循环将所有这一切组合在一起。如果调用 `rx.recv().await` 的结果是 `Some(message)`，我们可以访问消息并可以在循环体中使用它，就像我们可以使用 `if let` 一样。如果结果是 `None`，循环结束。每次循环完成时，它再次到达等待点，所以运行时再次暂停它，直到另一条消息到达。

代码现在成功发送和接收所有消息。不幸的是，仍然有几个问题。首先，消息不是以半秒间隔到达。它们都在我们启动程序后2秒（2,000毫秒）同时到达。其次，这个程序也永远不会退出！相反，它永远等待新消息。你需要使用 <kbd>ctrl</kbd>-<kbd>C</kbd> 来关闭它。

#### 一个 Async 块内的代码按线性执行

让我们首先检查为什么消息在完整延迟后一次全部到达，而不是在每条消息之间有延迟。在给定的 async 块内，代码中 `await` 关键字出现的顺序也是程序运行时它们执行的顺序。

代码清单17-10中只有一个 async 块，所以其中的所有内容都线性运行。仍然没有并发。所有 `tx.send` 调用都发生，穿插着所有 `trpl::sleep` 调用及其相关的等待点。只有到那时，`while let` 循环才能通过 `recv` 调用上的任何等待点。

为了获得我们想要的行为，即在每条消息之间发生睡眠延迟，我们需要将 `tx` 和 `rx` 操作放在它们自己的 async 块中，如代码清单17-11所示。然后运行时可以使用 `trpl::join` 分别执行它们，就像在代码清单17-8中一样。再次，我们等待调用 `trpl::join` 的结果，而不是单独的 futures。如果我们按顺序等待单独的 futures，我们将只是回到顺序流——这正是我们试图_不_做的。

<!-- We cannot test this one because it never stops! -->

<Listing number="17-11" caption="将 `send` 和 `recv` 分离到它们自己的 `async` 块中，并等待这些块的 futures" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-11/src/main.rs:futures}}
```

</Listing>

使用代码清单17-11中更新的代码，消息以500毫秒间隔打印，而不是在2秒后全部匆忙到达。

#### 将所有权移动到 Async 块中

然而，程序仍然永远不会退出，因为 `while let` 循环与 `trpl::join` 交互的方式：

- 从 `trpl::join` 返回的 future 只有在传递给它的_两个_ futures 都完成后才完成。
- `tx_fut` future 在它完成在 `vals` 中发送最后一条消息后的睡眠后完成。
- `rx_fut` future 在 `while let` 循环结束之前不会完成。
- `while let` 循环在等待 `rx.recv` 产生 `None` 之前不会结束。
- 等待 `rx.recv` 只有在 channel 的另一端关闭时才会返回 `None`。
- channel 只有在调用 `rx.close` 或发送端 `tx` 被丢弃时才会关闭。
- 我们不在任何地方调用 `rx.close`，并且 `tx` 直到传递给 `trpl::block_on` 的最外层 async 块结束才会被丢弃。
- 该块无法结束，因为它被阻塞在 `trpl::join` 完成上，这让我们回到了这个列表的顶部。

现在，我们发送消息的 async 块只_借用_ `tx`，因为发送消息不需要所有权，但如果我们可以将 `tx` _移动_到该 async 块中，它将在该块结束时被丢弃。在第13章的[“捕获引用或移动所有权”][capture-or-move]<!-- ignore -->部分中，你学习了如何将 `move` 关键字与闭包一起使用，并且，如第16章的[“在线程中使用 `move` 闭包”][move-threads]<!-- ignore -->部分中讨论的，我们在使用线程时经常需要将数据移动到闭包中。相同的基本动态适用于 async 块，所以 `move` 关键字与 async 块一起工作，就像它与闭包一样。

在代码清单17-12中，我们将用于发送消息的块从 `async` 更改为 `async move`。

<Listing number="17-12" caption="代码清单17-11的修订版本，完成后正确关闭" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-12/src/main.rs:with-move}}
```

</Listing>

当我们运行_这个_版本的代码时，它会在最后一条消息发送和接收后优雅地关闭。接下来，让我们看看需要改变什么才能从多个 future 发送数据。

#### 使用 `join!` 宏连接多个 Futures

这个 async channel 也是一个多生产者 channel，所以如果我们想从多个 futures 发送消息，我们可以在 `tx` 上调用 `clone`，如代码清单17-13所示。

<Listing number="17-13" caption="使用 async 块的多生产者" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-13/src/main.rs:here}}
```

</Listing>

首先，我们在第一个 async 块外克隆 `tx`，创建 `tx1`。我们将 `tx1` 移动到该块中，就像我们之前对 `tx` 所做的那样。然后，稍后，我们将原始 `tx` 移动到一个_新的_ async 块中，我们在其中以稍慢的延迟发送更多消息。我们碰巧将这个新的 async 块放在接收消息的 async 块之后，但它也可以放在它之前。关键是等待 futures 的顺序，而不是创建它们的顺序。

发送消息的两个 async 块都需要是 `async move` 块，以便 `tx` 和 `tx1` 在那些块完成时都被丢弃。否则，我们将回到我们开始时的相同无限循环。

最后，我们从 `trpl::join` 切换到 `trpl::join!` 来处理额外的 future：`join!` 宏等待任意数量的 futures，其中我们在编译时知道 futures 的数量。我们将在本章后面讨论等待未知数量的 futures 集合。

现在我们看到来自两个发送 futures 的所有消息，并且因为发送 futures 在发送后使用 slightly 不同的延迟，消息也以这些不同的间隔接收：

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
received 'hi'
received 'more'
received 'from'
received 'the'
received 'messages'
received 'future'
received 'for'
received 'you'
```

我们已经探索了如何使用消息传递在 futures 之间发送数据，async 块内的代码如何顺序运行，如何将所有权移动到 async 块中，以及如何连接多个 futures。接下来，让我们讨论如何以及为什么告诉运行时它可以切换到另一个任务。

[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[join-handles]: ch16-01-threads.html#waiting-for-all-threads-to-finish
[message-passing-threads]: ch16-02-message-passing.html
[if-let]: ch06-03-if-let.html
[capture-or-move]: ch13-01-closures.html#capturing-references-or-moving-ownership
[move-threads]: ch16-01-threads.html#using-move-closures-with-threads
