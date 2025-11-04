## 优雅关闭和清理

代码清单21-20中的代码通过使用线程池异步响应请求，正如我们所期望的。我们收到一些关于 `workers`、`id` 和 `thread` 字段的警告，这些字段我们没有以直接的方式使用，这提醒我们没有清理任何内容。当我们使用不太优雅的 <kbd>ctrl</kbd>-<kbd>C</kbd> 方法来停止主线程时，所有其他线程也会立即停止，即使它们正在处理请求。

接下来，我们将实现 `Drop` trait，以便在池中的每个线程上调用 `join`，以便它们可以在关闭之前完成正在处理的请求。然后，我们将实现一种方法来告诉线程它们应该停止接受新请求并关闭。为了看到这段代码的实际效果，我们将修改服务器，使其在优雅关闭其线程池之前只接受两个请求。

我们在进行中要注意的一件事：这些都不会影响处理执行闭包的部分代码，所以如果我们为异步运行时使用线程池，这里的一切都会相同。

### 在 `ThreadPool` 上实现 `Drop` Trait

让我们从在线程池上实现 `Drop` 开始。当池被丢弃时，我们的所有线程都应该 join 以确保它们完成工作。代码清单21-22显示了 `Drop` 实现的第一次尝试；这段代码还不会完全工作。

<Listing number="21-22" file-name="src/lib.rs" caption="当线程池超出作用域时 join 每个线程">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-22/src/lib.rs:here}}
```

</Listing>

首先，我们循环遍历线程池的每个 `workers`。我们为此使用 `&mut`，因为 `self` 是可变引用，我们还需要能够改变 `worker`。对于每个 `worker`，我们打印一条消息，说明这个特定的 `Worker` 实例正在关闭，然后我们在该 `Worker` 实例的线程上调用 `join`。如果对 `join` 的调用失败，我们使用 `unwrap` 使 Rust panic 并进入不优雅的关闭。

这是我们在编译此代码时得到的错误：

```console
{{#include ../listings/ch21-web-server/listing-21-22/output.txt}}
```

错误告诉我们无法调用 `join`，因为我们只有每个 `worker` 的可变借用，而 `join` 拥有其参数的所有权。为了解决这个问题，我们需要将线程从拥有 `thread` 的 `Worker` 实例中移出，以便 `join` 可以消费线程。一种方法是采用我们在代码清单18-15中采用的方法。如果 `Worker` 持有 `Option<thread::JoinHandle<()>>`，我们可以在 `Option` 上调用 `take` 方法以将值从 `Some` 变体中移出，并在其位置留下 `None` 变体。换句话说，正在运行的 `Worker` 在 `thread` 中会有 `Some` 变体，当我们想要清理 `Worker` 时，我们将用 `None` 替换 `Some`，以便 `Worker` 没有要运行的线程。

但是，_唯一_会出现这种情况的时候是在丢弃 `Worker` 时。作为交换，我们必须在访问 `worker.thread` 的任何地方处理 `Option<thread::JoinHandle<()>>`。惯用的 Rust 经常使用 `Option`，但当你发现自己将你知道会始终存在的东西包装在 `Option` 中作为这样的解决方法时，寻找替代方法以使代码更清晰、更不容易出错是个好主意。

在这种情况下，存在更好的替代方案：`Vec::drain` 方法。它接受范围参数以指定要从向量中删除哪些项目，并返回这些项目的迭代器。传递 `..` 范围语法将从向量中删除*每个*值。

因此，我们需要像这样更新 `ThreadPool` `drop` 实现：

<Listing file-name="src/lib.rs">

```rust
{{#rustdoc_include ../listings/ch21-web-server/no-listing-04-update-drop-definition/src/lib.rs:here}}
```

</Listing>

这解决了编译器错误，不需要对我们的代码进行任何其他更改。请注意，因为 drop 可能在 panic 时被调用，unwrap 也可能 panic 并导致双重 panic，这会立即崩溃程序并结束任何正在进行的清理。这对于示例程序来说是可以的，但不建议用于生产代码。

### 向线程发出停止监听作业的信号

通过我们进行的所有更改，我们的代码编译时没有任何警告。然而，坏消息是这段代码还没有按照我们想要的方式工作。关键是 `Worker` 实例的线程运行的闭包中的逻辑：目前，我们调用 `join`，但这不会关闭线程，因为它们永远`loop`查找作业。如果我们尝试使用当前的 `drop` 实现丢弃我们的 `ThreadPool`，主线程将永远阻塞，等待第一个线程完成。

为了修复这个问题，我们需要在 `ThreadPool` `drop` 实现中进行更改，然后在 `Worker` 循环中进行更改。

首先，我们将更改 `ThreadPool` `drop` 实现，以便在等待线程完成之前显式丢弃 `sender`。代码清单21-23显示了 `ThreadPool` 的更改，以显式丢弃 `sender`。与线程不同，这里我们_确实_需要使用 `Option` 才能使用 `Option::take` 将 `sender` 从 `ThreadPool` 中移出。

<Listing number="21-23" file-name="src/lib.rs" caption="在 join `Worker` 线程之前显式丢弃 `sender`">

```rust,noplayground,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-23/src/lib.rs:here}}
```

</Listing>

丢弃 `sender` 会关闭通道，这表示不会再发送更多消息。当发生这种情况时，`Worker` 实例在无限循环中执行的所有 `recv` 调用都将返回错误。在代码清单21-24中，我们更改 `Worker` 循环以在该情况下优雅地退出循环，这意味着当 `ThreadPool` `drop` 实现在其上调用 `join` 时，线程将完成。

<Listing number="21-24" file-name="src/lib.rs" caption="当 `recv` 返回错误时显式跳出循环">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-24/src/lib.rs:here}}
```

</Listing>

为了看到这段代码的实际效果，让我们修改 `main`，使其在优雅关闭服务器之前只接受两个请求，如代码清单21-25所示。

<Listing number="21-25" file-name="src/main.rs" caption="通过退出循环，在服务两个请求后关闭服务器">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/listing-21-25/src/main.rs:here}}
```

</Listing>

你不会希望真实的 Web 服务器在只服务两个请求后就关闭。这段代码只是演示优雅关闭和清理正在正常工作。

`take` 方法在 `Iterator` trait 中定义，并将迭代限制为最多前两个项目。`ThreadPool` 将在 `main` 结束时超出作用域，`drop` 实现将运行。

使用 `cargo run` 启动服务器并发出三个请求。第三个请求应该会出错，在你的终端中，你应该看到类似于以下内容的输出：

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-25
cargo run
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
third request will error because server will have shut down
copy output below
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.41s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Shutting down.
Shutting down worker 0
Worker 3 got a job; executing.
Worker 1 disconnected; shutting down.
Worker 2 disconnected; shutting down.
Worker 3 disconnected; shutting down.
Worker 0 disconnected; shutting down.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
```

你可能看到 `Worker` ID 和打印消息的不同顺序。我们可以从消息中看到这段代码是如何工作的：`Worker` 实例 0 和 3 收到了前两个请求。服务器在第二个连接后停止接受连接，`ThreadPool` 上的 `Drop` 实现甚至在 `Worker 3` 开始其作业之前就开始执行。丢弃 `sender` 会断开所有 `Worker` 实例的连接并告诉它们关闭。`Worker` 实例在断开连接时各自打印一条消息，然后线程池调用 `join` 以等待每个 `Worker` 线程完成。

注意这个特定执行的一个有趣方面：`ThreadPool` 丢弃了 `sender`，在任何 `Worker` 收到错误之前，我们尝试 join `Worker 0`。`Worker 0` 尚未从 `recv` 收到错误，所以主线程阻塞，等待 `Worker 0` 完成。同时，`Worker 3` 收到了一个作业，然后所有线程都收到了错误。当 `Worker 0` 完成时，主线程等待其余 `Worker` 实例完成。此时，它们都已退出循环并停止。

恭喜！我们现在已经完成了我们的项目；我们有一个基本的 Web 服务器，它使用线程池异步响应。我们能够执行服务器的优雅关闭，这会清理池中的所有线程。

以下是完整的代码供参考：

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/main.rs}}
```

</Listing>

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/lib.rs}}
```

</Listing>

我们可以在这里做更多的事情！如果你想继续增强这个项目，这里有一些想法：

- 向 `ThreadPool` 及其公共方法添加更多文档。
- 添加库功能的测试。
- 将对 `unwrap` 的调用更改为更健壮的错误处理。
- 使用 `ThreadPool` 执行除服务 Web 请求之外的其他任务。
- 在 [crates.io](https://crates.io/) 上找到一个线程池 crate，并使用该 crate 而不是我们实现的线程池实现类似的 Web 服务器。然后，将其 API 和健壮性与我们实现的线程池进行比较。

## 总结

做得好！你已经到达了本书的结尾！我们想感谢你加入我们这次 Rust 之旅。你现在已经准备好实现你自己的 Rust 项目并帮助其他人的项目。请记住，有一个欢迎其他 Rustacean 的社区，他们很乐意帮助你在 Rust 旅程中遇到的任何挑战。
