## 共享状态并发

消息传递是处理并发的好方法，但它不是唯一的方法。另一种方法将是多个线程访问相同的共享数据。再次考虑来自 Go 语言文档的口号中的这一部分："不要通过共享内存进行通信。"

通过共享内存进行通信会是什么样子？此外，为什么消息传递爱好者警告不要使用内存共享？

在某种程度上，任何编程语言中的通道都类似于单一所有权，因为一旦你将值通过通道传输，你就不应该再使用该值。共享内存并发就像多个所有权：多个线程可以同时访问相同的内存位置。正如你在第 15 章中看到的，智能指针使多个所有权成为可能，多个所有权可能增加复杂性，因为需要管理这些不同的所有者。Rust 的类型系统和所有权规则极大地帮助正确进行此管理。作为示例，让我们看看互斥锁，这是共享内存的更常见并发原语之一。

<!-- Old headings. Do not remove or links may break. -->

<a id="using-mutexes-to-allow-access-to-data-from-one-thread-at-a-time"></a>

### 使用互斥锁控制访问

_Mutex_ 是 _mutual exclusion_（互斥）的缩写，因为互斥锁只允许一个线程在任何给定时间访问某些数据。要访问互斥锁中的数据，线程必须首先通过请求获取互斥锁的锁来发出它想要访问的信号。_锁_ 是互斥锁的一部分的数据结构，它跟踪当前谁对数据具有独占访问权限。因此，互斥锁被描述为通过锁定系统 _保护_ 它持有的数据。

互斥锁以难以使用而闻名，因为你必须记住两条规则：

1. 在使用数据之前，你必须尝试获取锁。
2. 当你完成互斥锁保护的数据时，你必须解锁数据，以便其他线程可以获取锁。

对于互斥锁的真实世界比喻，想象一下会议上的小组讨论，只有一个麦克风。在小组成员可以说话之前，他们必须要求或发出他们想要使用麦克风的信号。当他们得到麦克风时，他们可以想说话多久就说话多久，然后将麦克风交给下一个请求说话的小组成员。如果小组成员在完成时忘记交出麦克风，其他人就无法说话。如果共享麦克风的管理出错，小组将无法按计划工作！

互斥锁的管理可能非常棘手，这就是为什么这么多人热衷于通道。但是，由于 Rust 的类型系统和所有权规则，你不能在锁定和解锁上出错。

#### `Mutex<T>` 的 API

作为如何使用互斥锁的示例，让我们首先在单线程上下文中使用互斥锁，如代码清单 16-12 所示。

<Listing number="16-12" file-name="src/main.rs" caption="为简单起见，在单线程上下文中探索 `Mutex<T>` 的 API">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-12/src/main.rs}}
```

</Listing>

与许多类型一样，我们使用关联函数 `new` 创建 `Mutex<T>`。要访问互斥锁内部的数据，我们使用 `lock` 方法获取锁。此调用将阻塞当前线程，使其无法执行任何工作，直到轮到我们拥有锁。

如果持有锁的另一个线程 panic，对 `lock` 的调用将失败。在这种情况下，没有人能够获得锁，所以我们选择 `unwrap` 并在这种情况下让此线程 panic。

在我们获取锁之后，我们可以将返回值（在这种情况下命名为 `num`）视为对内部数据的可变引用。类型系统确保我们在使用 `m` 中的值之前获取锁。`m` 的类型是 `Mutex<i32>`，不是 `i32`，所以我们 _必须_ 调用 `lock` 才能使用 `i32` 值。我们不能忘记；类型系统不会让我们以其他方式访问内部 `i32`。

对 `lock` 的调用返回一个名为 `MutexGuard` 的类型，包装在我们通过调用 `unwrap` 处理的 `LockResult` 中。`MutexGuard` 类型实现 `Deref` 指向我们的内部数据；该类型还有一个 `Drop` 实现，当 `MutexGuard` 超出作用域时自动释放锁，这发生在内部作用域结束时。因此，我们不冒忘记释放锁并阻止其他线程使用互斥锁的风险，因为锁释放会自动发生。

在丢弃锁之后，我们可以打印互斥锁值，并看到我们能够将内部 `i32` 更改为 `6`。

<!-- Old headings. Do not remove or links may break. -->

<a id="sharing-a-mutext-between-multiple-threads"></a>

#### 对 `Mutex<T>` 的共享访问

现在让我们尝试使用 `Mutex<T>` 在多个线程之间共享值。我们将生成 10 个线程，让它们每个将计数器值增加 1，所以计数器从 0 变为 10。代码清单 16-13 中的示例将有一个编译器错误，我们将使用该错误来了解有关使用 `Mutex<T>` 以及 Rust 如何帮助我们正确使用它的更多信息。

<Listing number="16-13" file-name="src/main.rs" caption="十个线程，每个线程增加由 `Mutex<T>` 保护的计数器">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-13/src/main.rs}}
```

</Listing>

我们创建一个 `counter` 变量来在 `Mutex<T>` 内保存 `i32`，就像我们在代码清单 16-12 中所做的那样。接下来，我们通过迭代数字范围来创建 10 个线程。我们使用 `thread::spawn` 并给所有线程相同的闭包：一个将 `counter` 移动到线程中，通过调用 `lock` 方法获取 `Mutex<T>` 上的锁，然后将互斥锁中的值加 1。当线程完成运行其闭包时，`num` 将超出作用域并释放锁，以便另一个线程可以获取它。

在主线程中，我们收集所有 join 句柄。然后，就像我们在代码清单 16-2 中所做的那样，我们在每个句柄上调用 `join` 以确保所有线程完成。此时，主线程将获取锁并打印此程序的结果。

我们暗示此示例不会编译。现在让我们找出原因！

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-13/output.txt}}
```

错误消息说明 `counter` 值在循环的前一次迭代中被移动。Rust 告诉我们，我们不能将锁 `counter` 的所有权移动到多个线程。让我们使用我们在第 15 章讨论的多所有权方法来修复编译器错误。

#### 多线程的多所有权

在第 15 章中，我们通过使用智能指针 `Rc<T>` 创建引用计数值来给值多个所有者。让我们在这里做同样的事情，看看会发生什么。我们将在代码清单 16-14 中将 `Mutex<T>` 包装在 `Rc<T>` 中，并在移动所有权到线程之前克隆 `Rc<T>`。

<Listing number="16-14" file-name="src/main.rs" caption="尝试使用 `Rc<T>` 允许多个线程拥有 `Mutex<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-14/src/main.rs}}
```

</Listing>

再次，我们编译并得到...不同的错误！编译器教给我们很多：

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-14/output.txt}}
```

哇，那个错误消息非常冗长！这是要关注的重要部分：`` `Rc<Mutex<i32>>` cannot be sent between threads safely ``。编译器还告诉我们原因：`` the trait `Send` is not implemented for `Rc<Mutex<i32>>` ``。我们将在下一节讨论 `Send`：它是确保我们与线程一起使用的类型用于并发情况的 trait 之一。

不幸的是，`Rc<T>` 不能安全地在线程之间共享。当 `Rc<T>` 管理引用计数时，它为每次调用 `clone` 增加计数，并在每个克隆被丢弃时从计数中减去。但它不使用任何并发原语来确保对计数的更改不能被另一个线程中断。这可能导致错误的计数——可能导致内存泄漏或值在我们完成使用之前被丢弃的细微错误。我们需要的是一个类型，它完全像 `Rc<T>`，但以线程安全的方式对引用计数进行更改。

#### 使用 `Arc<T>` 进行原子引用计数

幸运的是，`Arc<T>` _是_ 一个像 `Rc<T>` 的类型，可以安全地在并发情况中使用。_a_ 代表 _atomic_（原子），意味着它是一个 _原子引用计数_ 类型。原子是另一种并发原语，我们不会在这里详细介绍：有关更多详细信息，请参阅 [`std::sync::atomic`][atomic]<!-- ignore --> 的标准库文档。此时，你只需要知道原子像原始类型一样工作，但可以安全地在线程之间共享。

你可能想知道为什么所有原始类型都不是原子的，为什么标准库类型没有默认实现使用 `Arc<T>`。原因是线程安全会带来性能损失，你只想在真正需要时才支付。如果你只是在单线程内对值执行操作，如果你的代码不必强制执行原子提供的保证，它可以运行得更快。

让我们回到我们的示例：`Arc<T>` 和 `Rc<T>` 具有相同的 API，所以我们可以通过更改 `use` 行、`new` 的调用和 `clone` 的调用来修复程序。代码清单 16-15 中的代码最终将编译并运行。

<Listing number="16-15" file-name="src/main.rs" caption="使用 `Arc<T>` 包装 `Mutex<T>` 以便能够在多个线程之间共享所有权">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-15/src/main.rs}}
```

</Listing>

此代码将打印以下内容：

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Result: 10
```

我们做到了！我们从 0 数到 10，这可能看起来不是很令人印象深刻，但它确实教给我们很多关于 `Mutex<T>` 和线程安全的知识。你也可以使用此程序的结构来执行比仅仅增加计数器更复杂的操作。使用此策略，你可以将计算划分为独立部分，将这些部分拆分到线程，然后使用 `Mutex<T>` 让每个线程用其部分更新最终结果。

注意，如果你正在执行简单的数值操作，有比 [`std::sync::atomic` 模块的标准库][atomic]<!-- ignore --> 提供的 `Mutex<T>` 类型更简单的类型。这些类型提供对原始类型的安全、并发、原子访问。我们选择在此示例中使用带原始类型的 `Mutex<T>`，以便我们可以专注于 `Mutex<T>` 的工作原理。

<!-- Old headings. Do not remove or links may break. -->

<a id="similarities-between-refcelltrct-and-mutextarct"></a>

### 比较 `RefCell<T>`/`Rc<T>` 和 `Mutex<T>`/`Arc<T>`

你可能已经注意到 `counter` 是不可变的，但我们可以获得对其内部值的可变引用；这意味着 `Mutex<T>` 提供内部可变性，就像 `Cell` 系列一样。就像我们在第 15 章中使用 `RefCell<T>` 允许我们在 `Rc<T>` 内部改变内容一样，我们使用 `Mutex<T>` 在 `Arc<T>` 内部改变内容。

另一个需要注意的细节是，当你使用 `Mutex<T>` 时，Rust 无法保护你免受所有类型的逻辑错误。回想第 15 章，使用 `Rc<T>` 会带来创建引用循环的风险，其中两个 `Rc<T>` 值相互引用，导致内存泄漏。类似地，`Mutex<T>` 会带来创建 _死锁_ 的风险。当操作需要锁定两个资源并且两个线程各自获取了其中一个锁，导致它们永远相互等待时，就会发生这些情况。如果你对死锁感兴趣，尝试创建一个有死锁的 Rust 程序；然后，研究任何语言中互斥锁的死锁缓解策略，并尝试在 Rust 中实现它们。`Mutex<T>` 和 `MutexGuard` 的标准库 API 文档提供了有用的信息。

我们将通过讨论 `Send` 和 `Sync` trait 以及如何将它们与自定义类型一起使用来结束本章。

[atomic]: ../std/sync/atomic/index.html
