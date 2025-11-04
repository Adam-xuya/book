
<!-- Old headings. Do not remove or links may break. -->

<a id="yielding"></a>

### 将控制权让给运行时

回想一下[“我们的第一个 Async 程序”][async-program]<!-- ignore -->部分，在每个等待点，如果正在等待的 future 还没有准备好，Rust 会给运行时一个暂停任务并切换到另一个任务的机会。反之亦然：Rust _只_在等待点暂停 async 块并将控制权交还给运行时。等待点之间的所有内容都是同步的。

这意味着如果你在 async 块中做大量工作而没有等待点，该 future 将阻塞任何其他 futures 取得进展。你可能有时会听到这被称为一个 future _饿死_其他 futures。在某些情况下，这可能不是什么大问题。但是，如果你正在进行某种昂贵的设置或长时间运行的工作，或者如果你有一个 future 将无限期地继续执行某个特定任务，你需要考虑何时以及在哪里将控制权交还给运行时。

让我们模拟一个长时间运行的操作来说明饥饿问题，然后探索如何解决它。代码清单17-14引入了一个 `slow` 函数。

<Listing number="17-14" caption="使用 `thread::sleep` 模拟慢操作" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-14/src/main.rs:slow}}
```

</Listing>

这段代码使用 `std::thread::sleep` 而不是 `trpl::sleep`，以便调用 `slow` 将阻塞当前线程一段时间（以毫秒为单位）。我们可以使用 `slow` 来代表既长时间运行又阻塞的现实世界操作。

在代码清单17-15中，我们使用 `slow` 来模拟在一对 futures 中执行这种 CPU 密集型工作。

<Listing number="17-15" caption="调用 `slow` 模拟运行慢操作" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-15/src/main.rs:slow-futures}}
```

</Listing>

每个 future 只在执行大量慢操作_之后_才将控制权交还给运行时。如果你运行这段代码，你将看到这个输出：

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-15/
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'a' ran for 10ms
'a' ran for 20ms
'b' started.
'b' ran for 75ms
'b' ran for 10ms
'b' ran for 15ms
'b' ran for 350ms
'a' finished.
```

与代码清单17-5中我们使用 `trpl::select` 来竞争获取两个 URL 的 futures 一样，`select` 在 `a` 完成后仍然完成。但是，在两个 futures 中对 `slow` 的调用之间没有交错。`a` future 完成所有工作，直到等待 `trpl::sleep` 调用，然后 `b` future 完成所有工作，直到它自己的 `trpl::sleep` 调用被等待，最后 `a` future 完成。为了允许两个 futures 在它们的慢任务之间取得进展，我们需要等待点，以便我们可以将控制权交还给运行时。这意味着我们需要一些我们可以等待的东西！

我们已经在代码清单17-15中看到了这种移交：如果我们在 `a` future 的末尾移除了 `trpl::sleep`，它将在 `b` future 根本_不_运行的情况下完成。让我们尝试使用 `trpl::sleep` 函数作为让操作切换进展的起点，如代码清单17-16所示。

<Listing number="17-16" caption="使用 `trpl::sleep` 让操作切换进展" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-16/src/main.rs:here}}
```

</Listing>

我们在每次调用 `slow` 之间添加了带有等待点的 `trpl::sleep` 调用。现在两个 futures 的工作交错进行：

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-16
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'b' started.
'b' ran for 75ms
'a' ran for 10ms
'b' ran for 10ms
'a' ran for 20ms
'b' ran for 15ms
'a' finished.
```

`a` future 仍然在将控制权移交给 `b` 之前运行一段时间，因为它在调用 `trpl::sleep` 之前调用 `slow`，但之后 futures 每次它们中的一个到达等待点时都会来回交换。在这种情况下，我们在每次调用 `slow` 之后都这样做，但我们可以以对我们最有意义的方式分解工作。

但我们真的不想在这里_睡眠_：我们想尽可能快地取得进展。我们只需要将控制权交还给运行时。我们可以直接使用 `trpl::yield_now` 函数来做到这一点。在代码清单17-17中，我们将所有这些 `trpl::sleep` 调用替换为 `trpl::yield_now`。

<Listing number="17-17" caption="使用 `yield_now` 让操作切换进展" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-17/src/main.rs:yields}}
```

</Listing>

这段代码既更清楚地表达了实际意图，也可以比使用 `sleep` 快得多，因为像 `sleep` 使用的计时器通常对它们可以有多细粒度有限制。例如，我们正在使用的 `sleep` 版本将始终至少睡眠一毫秒，即使我们传递给它一纳秒的 `Duration`。同样，现代计算机_很快_：它们在一毫秒内可以做很多事情！

这意味着 async 对于计算密集型任务也可能有用，取决于你的程序还在做什么，因为它为构建程序不同部分之间的关系提供了有用的工具（但代价是 async 状态机的开销）。这是一种_协作多任务处理_形式，其中每个 future 都有权通过等待点决定何时移交控制权。因此，每个 future 也有责任避免阻塞太久。在一些基于 Rust 的嵌入式操作系统中，这是_唯一_一种多任务处理！

在现实世界的代码中，你通常不会在每一行上交替函数调用和等待点，当然。虽然以这种方式让出控制权相对便宜，但它不是免费的。在许多情况下，试图分解计算密集型任务可能会使其显著变慢，所以有时让操作短暂阻塞对_整体_性能更好。始终测量以查看代码的实际性能瓶颈是什么。但是，如果你_确实_看到大量工作以串行方式发生，而你期望它们并发发生，那么记住底层动态是很重要的！

### 构建我们自己的 Async 抽象

我们还可以将 futures 组合在一起以创建新模式。例如，我们可以使用我们已经拥有的 async 构建块构建一个 `timeout` 函数。当我们完成时，结果将是另一个构建块，我们可以用它来创建更多的 async 抽象。

代码清单17-18显示了我们期望这个 `timeout` 如何与慢 future 一起工作。

<Listing number="17-18" caption="使用我们想象的 `timeout` 运行带有时间限制的慢操作" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-18/src/main.rs:here}}
```

</Listing>

让我们实现它！首先，让我们考虑 `timeout` 的 API：

- 它本身需要是一个 async 函数，以便我们可以等待它。
- 它的第一个参数应该是一个要运行的 future。我们可以使其泛型以允许它适用于任何 future。
- 它的第二个参数将是最大等待时间。如果我们使用 `Duration`，这将使将其传递给 `trpl::sleep` 变得容易。
- 它应该返回一个 `Result`。如果 future 成功完成，`Result` 将是 `Ok` 和 future 产生的值。如果超时首先到期，`Result` 将是 `Err` 和超时等待的持续时间。

代码清单17-19显示了这个声明。

<!-- This is not tested because it intentionally does not compile. -->

<Listing number="17-19" caption="定义 `timeout` 的签名" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-19/src/main.rs:declaration}}
```

</Listing>

这满足了我们对类型的目标。现在让我们考虑我们需要的行为：我们希望将传入的 future 与持续时间竞争。我们可以使用 `trpl::sleep` 从持续时间创建一个计时器 future，并使用 `trpl::select` 将该计时器与调用者传入的 future 一起运行。

在代码清单17-20中，我们通过匹配等待 `trpl::select` 的结果来实现 `timeout`。

<Listing number="17-20" caption="使用 `select` 和 `sleep` 定义 `timeout`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-20/src/main.rs:implementation}}
```

</Listing>

`trpl::select` 的实现不是公平的：它总是按照传入的顺序轮询参数（其他 `select` 实现将随机选择先轮询哪个参数）。因此，我们将 `future_to_try` 首先传递给 `select`，以便即使 `max_time` 是非常短的持续时间，它也有机会完成。如果 `future_to_try` 先完成，`select` 将返回 `Left` 和 `future_to_try` 的输出。如果 `timer` 先完成，`select` 将返回 `Right` 和计时器的输出 `()`。

如果 `future_to_try` 成功并且我们得到 `Left(output)`，我们返回 `Ok(output)`。如果睡眠计时器改为到期并且我们得到 `Right(())`，我们用 `_` 忽略 `()`，并改为返回 `Err(max_time)`。

这样，我们有了一个由两个其他 async 辅助函数构建的可工作的 `timeout`。如果我们运行我们的代码，它将在超时后打印失败模式：

```text
Failed after 2 seconds
```

因为 futures 与其他 futures 组合，你可以使用较小的 async 构建块构建真正强大的工具。例如，你可以使用相同的方法将超时与重试结合起来，然后与网络调用等操作一起使用（例如代码清单17-5中的那些）。

在实践中，你通常会直接使用 `async` 和 `await`，其次使用函数（如 `select`）和宏（如 `join!` 宏）来控制最外层 futures 的执行方式。

我们现在已经看到了多种同时处理多个 futures 的方法。接下来，我们将看看如何使用_流_随着时间的推移以序列方式处理多个 futures。

[async-program]: ch17-01-futures-and-syntax.html#our-first-async-program
