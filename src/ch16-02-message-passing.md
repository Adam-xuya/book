<!-- Old headings. Do not remove or links may break. -->

<a id="using-message-passing-to-transfer-data-between-threads"></a>

## 使用消息传递在线程之间传输数据

确保安全并发的一种越来越流行的方法是消息传递，其中线程或参与者通过相互发送包含数据的消息进行通信。这是来自 [Go 语言文档](https://golang.org/doc/effective_go.html#concurrency) 的口号中的想法："不要通过共享内存进行通信；相反，通过通信共享内存。"

要实现消息发送并发，Rust 的标准库提供了通道的实现。_通道_ 是一种通用编程概念，数据通过它从一个线程发送到另一个线程。

你可以将编程中的通道想象为像单向水道，如溪流或河流。如果你将像橡皮鸭这样的东西放入河流中，它会向下游到达水道的末端。

通道有两半：发送器和接收器。发送器一半是上游位置，你将橡皮鸭放入河流中，接收器一半是橡皮鸭最终到达的下游位置。你的代码的一部分在发送器上调用方法，使用你想要发送的数据，另一部分检查接收端是否有到达的消息。如果发送器或接收器一半被丢弃，则通道被称为 _关闭_。

在这里，我们将构建一个程序，它有一个线程生成值并通过通道发送它们，另一个线程将接收值并打印它们。我们将使用通道在线程之间发送简单值来说明该功能。一旦你熟悉了该技术，你可以将通道用于任何需要相互通信的线程，例如聊天系统或许多线程执行计算的一部分并将部分发送到一个聚合结果的线程的系统。

首先，在代码清单 16-6 中，我们将创建一个通道但不使用它做任何事情。注意，这还不会编译，因为 Rust 无法判断我们想通过通道发送什么类型的值。

<Listing number="16-6" file-name="src/main.rs" caption="创建通道并将两半分配给 `tx` 和 `rx`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-06/src/main.rs}}
```

</Listing>

我们使用 `mpsc::channel` 函数创建新通道；`mpsc` 代表 _multiple producer, single consumer_（多生产者，单消费者）。简而言之，Rust 标准库实现通道的方式意味着通道可以有多个 _发送_ 端产生值，但只有一个 _接收_ 端消费这些值。想象多个溪流汇合在一起形成一条大河：从任何溪流发送下来的任何东西最终都会在一端进入一条河流。我们现在从单个生产者开始，但当我们使此示例工作时，我们将添加多个生产者。

`mpsc::channel` 函数返回一个元组，其第一个元素是发送端——发送器——第二个元素是接收端——接收器。缩写 `tx` 和 `rx` 传统上在许多领域中分别用于 _transmitter_（发送器）和 _receiver_（接收器），所以我们这样命名变量以指示每一端。我们使用带模式的 `let` 语句来解构元组；我们将在第 19 章讨论在 `let` 语句中使用模式和解构。现在，知道以这种方式使用 `let` 语句是从 `mpsc::channel` 返回的元组中提取部分的便捷方法。

让我们将发送端移动到生成线程中，并让它发送一个字符串，以便生成线程与主线程通信，如代码清单 16-7 所示。这就像在上游将橡皮鸭放入河流中或从一个线程向另一个线程发送聊天消息。

<Listing number="16-7" file-name="src/main.rs" caption='将 `tx` 移动到生成线程并发送 `"hi"`'>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-07/src/main.rs}}
```

</Listing>

同样，我们使用 `thread::spawn` 创建新线程，然后使用 `move` 将 `tx` 移动到闭包中，以便生成线程拥有 `tx`。生成线程需要拥有发送器才能通过通道发送消息。

发送器有一个 `send` 方法，它接受我们想要发送的值。`send` 方法返回 `Result<T, E>` 类型，所以如果接收器已被丢弃并且没有地方发送值，发送操作将返回错误。在此示例中，我们在出错时调用 `unwrap` 以 panic。但在真实应用程序中，我们会正确处理它：返回第 9 章以回顾适当的错误处理策略。

在代码清单 16-8 中，我们将从主线程中的接收器获取值。这就像从河流末端的水中检索橡皮鸭或接收聊天消息。

<Listing number="16-8" file-name="src/main.rs" caption='在主线程中接收值 `"hi"` 并打印它'>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-08/src/main.rs}}
```

</Listing>

接收器有两个有用的方法：`recv` 和 `try_recv`。我们使用 `recv`，_receive_ 的缩写，它将阻塞主线程的执行并等待，直到值通过通道发送。一旦值被发送，`recv` 将在 `Result<T, E>` 中返回它。当发送器关闭时，`recv` 将返回错误以指示不会再有值到来。

`try_recv` 方法不会阻塞，但会立即返回 `Result<T, E>`：如果有可用消息，则为 `Ok` 值，如果这次没有任何消息，则为 `Err` 值。如果此线程在等待消息时有其他工作要做，使用 `try_recv` 很有用：我们可以编写一个循环，定期调用 `try_recv`，如果有可用消息则处理它，否则做其他工作一会儿，直到再次检查。

我们在此示例中使用 `recv` 是为了简单起见；除了等待消息之外，主线程没有其他工作要做，所以阻塞主线程是合适的。

当我们运行代码清单 16-8 中的代码时，我们将看到从主线程打印的值：

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
```

完美！

<!-- Old headings. Do not remove or links may break. -->

<a id="channels-and-ownership-transference"></a>

### 通过通道转移所有权

所有权规则在消息发送中发挥重要作用，因为它们帮助你编写安全、并发的代码。防止并发编程中的错误是在整个 Rust 程序中思考所有权的优势。让我们做一个实验来展示通道和所有权如何协同工作以防止问题：我们将尝试在我们将 `val` 值通过通道发送 _之后_ 在生成线程中使用它。尝试编译代码清单 16-9 中的代码，看看为什么不允许此代码。

<Listing number="16-9" file-name="src/main.rs" caption="尝试在通过通道发送 `val` 之后使用它">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-09/src/main.rs}}
```

</Listing>

这里，我们尝试在通过 `tx.send` 将其通过通道发送后打印 `val`。允许这将是一个坏主意：一旦值被发送到另一个线程，该线程可以在我们尝试再次使用该值之前修改或丢弃它。可能，另一个线程的修改可能由于不一致或不存在的数据而导致错误或意外结果。但是，如果我们尝试编译代码清单 16-9 中的代码，Rust 会给我们一个错误：

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-09/output.txt}}
```

我们的并发错误导致了编译时错误。`send` 函数获取其参数的所有权，当值被移动时，接收器获取它的所有权。这阻止我们在发送后意外再次使用该值；所有权系统检查一切正常。

<!-- Old headings. Do not remove or links may break. -->

<a id="sending-multiple-values-and-seeing-the-receiver-waiting"></a>

### 发送多个值

代码清单 16-8 中的代码编译并运行，但它没有清楚地显示两个独立线程通过通道相互通信。

在代码清单 16-10 中，我们进行了一些修改，这些修改将证明代码清单 16-8 中的代码正在并发运行：生成线程现在将发送多条消息，并在每条消息之间暂停一秒。

<Listing number="16-10" file-name="src/main.rs" caption="发送多条消息，并在每条消息之间暂停">

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-10/src/main.rs}}
```

</Listing>

这次，生成线程有一个我们想要发送到主线程的字符串向量。我们遍历它们，单独发送每个，并通过使用 `Duration` 值一秒调用 `thread::sleep` 函数在它们之间暂停。

在主线程中，我们不再显式调用 `recv` 函数：相反，我们将 `rx` 视为迭代器。对于接收到的每个值，我们打印它。当通道关闭时，迭代将结束。

运行代码清单 16-10 中的代码时，你应该看到以下输出，每行之间有一秒的暂停：

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
Got: from
Got: the
Got: thread
```

因为我们在主线程的 `for` 循环中没有任何暂停或延迟的代码，我们可以告诉主线程正在等待从生成线程接收值。

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-multiple-producers-by-cloning-the-transmitter"></a>

### 创建多个生产者

早些时候我们提到 `mpsc` 是 _multiple producer, single consumer_（多生产者，单消费者）的首字母缩写。让我们使用 `mpsc` 并扩展代码清单 16-10 中的代码，以创建多个线程，它们都向同一接收器发送值。我们可以通过克隆发送器来做到这一点，如代码清单 16-11 所示。

<Listing number="16-11" file-name="src/main.rs" caption="从多个生产者发送多条消息">

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-11/src/main.rs:here}}
```

</Listing>

这次，在我们创建第一个生成线程之前，我们在发送器上调用 `clone`。这将给我们一个新的发送器，我们可以将其传递给第一个生成线程。我们将原始发送器传递给第二个生成线程。这给了我们两个线程，每个线程向一个接收器发送不同的消息。

运行代码时，你的输出应该看起来像这样：

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
Got: more
Got: from
Got: messages
Got: for
Got: the
Got: thread
Got: you
```

你可能看到值的顺序不同，这取决于你的系统。这就是使并发有趣且困难的原因。如果你尝试使用 `thread::sleep`，在不同线程中给它各种值，每次运行都会更加不确定，并且每次都会创建不同的输出。

现在我们已经了解了通道的工作原理，让我们看看另一种并发方法。
