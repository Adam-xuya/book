## 使用线程同时运行代码

在大多数当前操作系统中，已执行程序的代码在 _进程_ 中运行，操作系统将同时管理多个进程。在程序中，你也可以有同时运行的独立部分。运行这些独立部分的功能称为 _线程_。例如，Web 服务器可以有多个线程，以便它可以同时响应多个请求。

将程序中的计算拆分为多个线程以同时运行多个任务可以提高性能，但它也增加了复杂性。因为线程可以同时运行，所以没有关于在不同线程上运行的代码部分的执行顺序的固有保证。这可能导致问题，例如：

- 竞争条件，其中线程以不一致的顺序访问数据或资源
- 死锁，其中两个线程相互等待，阻止两个线程继续
- 仅在特定情况下发生的错误，难以可靠地重现和修复

Rust 试图减轻使用线程的负面影响，但在多线程上下文中编程仍然需要仔细思考，并且需要与在单线程中运行的程序不同的代码结构。

编程语言以几种不同的方式实现线程，许多操作系统提供编程语言可以调用以创建新线程的 API。Rust 标准库使用线程实现的 _1:1_ 模型，其中程序为每个语言线程使用一个操作系统线程。有一些 crate 实现了其他线程模型，对 1:1 模型做出了不同的权衡。（Rust 的异步系统，我们将在下一章看到，也提供了另一种并发方法。）

### 使用 `spawn` 创建新线程

要创建新线程，我们调用 `thread::spawn` 函数并向其传递一个闭包（我们在第 13 章讨论过闭包），其中包含我们想在新线程中运行的代码。代码清单 16-1 中的示例从主线程打印一些文本，从新线程打印其他文本。

<Listing number="16-1" file-name="src/main.rs" caption="创建新线程以打印一件事，而主线程打印其他内容">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-01/src/main.rs}}
```

</Listing>

注意，当 Rust 程序的主线程完成时，所有生成的线程都会关闭，无论它们是否完成运行。此程序的输出可能每次都有点不同，但它看起来类似于以下内容：

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

对 `thread::sleep` 的调用强制线程停止执行一小段时间，允许不同的线程运行。线程可能会轮流执行，但不能保证：这取决于你的操作系统如何调度线程。在这次运行中，主线程首先打印，尽管生成线程的打印语句在代码中首先出现。即使我们告诉生成线程打印直到 `i` 是 `9`，它只在主线程关闭之前到达 `5`。

如果你运行此代码并只看到主线程的输出，或没有看到任何重叠，尝试增加范围中的数字，为操作系统在线程之间切换创造更多机会。

<!-- Old headings. Do not remove or links may break. -->

<a id="waiting-for-all-threads-to-finish-using-join-handles"></a>

### 等待所有线程完成

代码清单 16-1 中的代码不仅由于主线程结束而在大多数情况下过早停止生成线程，而且因为无法保证线程运行的顺序，我们也无法保证生成线程会运行！

我们可以通过将 `thread::spawn` 的返回值保存在变量中来修复生成线程不运行或过早结束的问题。`thread::spawn` 的返回类型是 `JoinHandle<T>`。`JoinHandle<T>` 是一个拥有的值，当我们在其上调用 `join` 方法时，它将等待其线程完成。代码清单 16-2 显示了如何使用我们在代码清单 16-1 中创建的线程的 `JoinHandle<T>`，以及如何调用 `join` 以确保生成线程在 `main` 退出之前完成。

<Listing number="16-2" file-name="src/main.rs" caption="保存来自 `thread::spawn` 的 `JoinHandle<T>` 以保证线程运行到完成">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-02/src/main.rs}}
```

</Listing>

在句柄上调用 `join` 会阻塞当前运行的线程，直到句柄表示的线程终止。_阻塞_ 线程意味着该线程被阻止执行工作或退出。因为我们将 `join` 的调用放在主线程的 `for` 循环之后，运行代码清单 16-2 应该产生与此类似的输出：

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

两个线程继续交替，但主线程由于对 `handle.join()` 的调用而等待，并且在生成线程完成之前不会结束。

但是让我们看看当我们改为将 `handle.join()` 移动到 `main` 中的 `for` 循环之前时会发生什么，如下所示：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/no-listing-01-join-too-early/src/main.rs}}
```

</Listing>

主线程将等待生成线程完成，然后运行其 `for` 循环，所以输出不再交错，如下所示：

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

小细节，例如 `join` 的调用位置，可以影响你的线程是否同时运行。

### 在线程中使用 `move` 闭包

我们经常将 `move` 关键字与传递给 `thread::spawn` 的闭包一起使用，因为闭包然后将从环境中使用的值获取所有权，从而将这些值的所有权从一个线程转移到另一个线程。在第 13 章的["捕获引用或移动所有权"][capture]<!-- ignore -->中，我们在闭包的上下文中讨论了 `move`。现在我们将更多地关注 `move` 和 `thread::spawn` 之间的交互。

注意代码清单 16-1 中我们传递给 `thread::spawn` 的闭包不接受参数：我们在生成线程的代码中不使用主线程的任何数据。要在生成线程中使用主线程的数据，生成线程的闭包必须捕获它需要的值。代码清单 16-3 显示尝试在主线程中创建向量并在生成线程中使用它。但是，这还不起作用，正如你稍后会看到的。

<Listing number="16-3" file-name="src/main.rs" caption="尝试在另一个线程中使用主线程创建的向量">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-03/src/main.rs}}
```

</Listing>

闭包使用 `v`，所以它将捕获 `v` 并使其成为闭包环境的一部分。因为 `thread::spawn` 在新线程中运行此闭包，我们应该能够在该新线程内访问 `v`。但是当我们编译此示例时，我们得到以下错误：

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-03/output.txt}}
```

Rust _推断_ 如何捕获 `v`，因为 `println!` 只需要对 `v` 的引用，闭包尝试借用 `v`。但是，有一个问题：Rust 无法判断生成线程将运行多长时间，所以它不知道对 `v` 的引用是否始终有效。

代码清单 16-4 提供了一个更可能具有对 `v` 的无效引用的场景。

<Listing number="16-4" file-name="src/main.rs" caption="一个线程，其闭包尝试从丢弃 `v` 的主线程捕获对 `v` 的引用">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-04/src/main.rs}}
```

</Listing>

如果 Rust 允许我们运行此代码，生成线程可能会立即被放到后台而不运行。生成线程内部有一个对 `v` 的引用，但主线程立即使用我们在第 15 章讨论的 `drop` 函数丢弃 `v`。然后，当生成线程开始执行时，`v` 不再有效，所以对它的引用也无效。哦不！

为了修复代码清单 16-3 中的编译器错误，我们可以使用错误消息的建议：

<!-- manual-regeneration
after automatic regeneration, look at listings/ch16-fearless-concurrency/listing-16-03/output.txt and copy the relevant part
-->

```text
help: to force the closure to take ownership of `v` (and any other referenced variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++
```

通过在闭包之前添加 `move` 关键字，我们强制闭包获取它使用的值的所有权，而不是允许 Rust 推断它应该借用值。对代码清单 16-3 的修改如代码清单 16-5 所示，将按我们的意图编译和运行。

<Listing number="16-5" file-name="src/main.rs" caption="使用 `move` 关键字强制闭包获取它使用的值的所有权">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-05/src/main.rs}}
```

</Listing>

我们可能想尝试同样的事情来修复代码清单 16-4 中的代码，其中主线程通过使用 `move` 闭包调用 `drop`。但是，此修复不会起作用，因为代码清单 16-4 尝试做的事情因不同的原因而被禁止。如果我们将 `move` 添加到闭包，我们会将 `v` 移动到闭包的环境中，我们不能再在主线程中对其调用 `drop`。相反，我们会得到此编译器错误：

```console
{{#include ../listings/ch16-fearless-concurrency/output-only-01-move-drop/output.txt}}
```

Rust 的所有权规则再次拯救了我们！我们从代码清单 16-3 的代码中得到错误，因为 Rust 是保守的，只为线程借用 `v`，这意味着主线程理论上可能使生成线程的引用无效。通过告诉 Rust 将 `v` 的所有权移动到生成线程，我们向 Rust 保证主线程不会再使用 `v`。如果我们以相同方式更改代码清单 16-4，当我们尝试在主线程中使用 `v` 时，我们会违反所有权规则。`move` 关键字覆盖 Rust 借用的保守默认值；它不允许我们违反所有权规则。

现在我们已经了解了线程是什么以及线程 API 提供的方法，让我们看看一些可以使用线程的情况。

[capture]: ch13-01-closures.html#capturing-references-or-moving-ownership
