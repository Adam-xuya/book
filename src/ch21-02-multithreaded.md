<!-- Old headings. Do not remove or links may break. -->

<a id="turning-our-single-threaded-server-into-a-multithreaded-server"></a>
<a id="from-single-threaded-to-multithreaded-server"></a>

## 从单线程到多线程服务器

目前，服务器将依次处理每个请求，这意味着它不会处理第二个连接，直到第一个连接完成处理。如果服务器收到越来越多的请求，这种串行执行将越来越不理想。如果服务器收到一个需要很长时间处理的请求，后续请求将不得不等待长时间请求完成，即使新请求可以快速处理。我们需要修复这个问题，但首先我们来看看实际的问题。

<!-- Old headings. Do not remove or links may break. -->

<a id="simulating-a-slow-request-in-the-current-server-implementation"></a>

### 模拟慢速请求

我们将查看处理缓慢的请求如何影响对我们当前服务器实现的其他请求。代码清单21-10实现了对 _/sleep_ 的请求处理，该请求具有模拟的慢速响应，将导致服务器在响应之前睡眠五秒。

<Listing number="21-10" file-name="src/main.rs" caption="通过睡眠五秒模拟慢速请求">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-10/src/main.rs:here}}
```

</Listing>

现在我们有三个情况，所以从 `if` 切换到 `match`。我们需要显式匹配 `request_line` 的切片以对字符串字面量进行模式匹配；`match` 不像相等方法那样进行自动引用和解引用。

第一个分支与代码清单21-9中的 `if` 块相同。第二个分支匹配对 _/sleep_ 的请求。收到该请求时，服务器将在渲染成功的 HTML 页面之前睡眠五秒。第三个分支与代码清单21-9中的 `else` 块相同。

你可以看到我们的服务器是多么原始：真正的库会以更简洁的方式处理多个请求的识别！

使用 `cargo run` 启动服务器。然后，打开两个浏览器窗口：一个用于 _http://127.0.0.1:7878_，另一个用于 _http://127.0.0.1:7878/sleep_。如果你像以前一样多次输入 _/_ URI，你会看到它快速响应。但是如果你输入 _/sleep_ 然后加载 _/_，你会看到 _/_ 等待 `sleep` 睡眠完整的五秒后才加载。

我们可以使用多种技术来避免请求在慢速请求后面堆积，包括使用我们在第17章中使用的 async；我们将实现的是线程池。

### 使用线程池提高吞吐量

_线程池_是一组已生成并等待处理任务的线程。当程序收到新任务时，它将池中的一个线程分配给该任务，该线程将处理该任务。池中的剩余线程可用于处理第一个线程正在处理时进来的任何其他任务。当第一个线程完成处理其任务时，它返回到空闲线程池，准备处理新任务。线程池允许你并发处理连接，从而提高服务器的吞吐量。

我们将池中的线程数限制为少量，以保护我们免受 DoS 攻击；如果我们让程序为每个进来的请求创建新线程，有人向我们的服务器发出 1000 万请求可能会通过耗尽我们服务器的所有资源并使请求处理停止而造成严重破坏。

因此，我们将有固定数量的线程在池中等待，而不是生成无限线程。进来的请求被发送到池进行处理。池将维护传入请求的队列。池中的每个线程将从该队列中弹出一个请求，处理该请求，然后向队列请求另一个请求。使用这种设计，我们可以并发处理最多 _`N`_ 个请求，其中 _`N`_ 是线程数。如果每个线程都在响应长时间运行的请求，后续请求仍可能在队列中堆积，但我们已经增加了在达到该点之前可以处理的长时间运行请求的数量。

这种技术只是提高 Web 服务器吞吐量的多种方法之一。你可能探索的其他选项是 fork/join 模型、单线程异步 I/O 模型和多线程异步 I/O 模型。如果你对这个主题感兴趣，可以阅读更多关于其他解决方案的信息并尝试实现它们；使用像 Rust 这样的低级语言，所有这些选项都是可能的。

在我们开始实现线程池之前，让我们谈谈使用池应该是什么样子。当你尝试设计代码时，首先编写客户端接口可以帮助指导你的设计。编写代码的 API，使其以你想要调用它的方式结构化；然后，在该结构内实现功能，而不是实现功能然后设计公共 API。

类似于我们在第12章的项目中使用测试驱动开发的方式，我们将在这里使用编译器驱动开发。我们将编写调用我们想要的函数的代码，然后查看编译器的错误以确定下一步应该更改什么以使代码工作。但是，在此之前，我们将探索我们不打算作为起点的技术。

<!-- Old headings. Do not remove or links may break. -->

<a id="code-structure-if-we-could-spawn-a-thread-for-each-request"></a>

#### 为每个请求生成线程

首先，让我们探索如果我们的代码确实为每个连接创建新线程，代码可能是什么样子。如前所述，由于可能生成无限数量线程的问题，这不是我们的最终计划，但它是首先获得工作多线程服务器的起点。然后，我们将添加线程池作为改进，对比两种解决方案会更容易。

代码清单21-11显示了在 `for` 循环中生成新线程以处理每个流的更改。

<Listing number="21-11" file-name="src/main.rs" caption="为每个流生成新线程">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-11/src/main.rs:here}}
```

</Listing>

正如你在第16章中学到的，`thread::spawn` 将创建一个新线程，然后在新线程中运行闭包中的代码。如果你运行此代码并在浏览器中加载 _/sleep_，然后在另外两个浏览器选项卡中加载 _/_，你确实会看到对 _/_ 的请求不必等待 _/sleep_ 完成。但是，正如我们提到的，这最终会压垮系统，因为你会无限制地创建新线程。

你可能还记得第17章，这正是 async 和 await 真正发挥作用的情况！在我们构建线程池并思考使用 async 会如何不同或相同时，请记住这一点。

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-a-similar-interface-for-a-finite-number-of-threads"></a>

#### 创建有限数量的线程

我们希望我们的线程池以类似、熟悉的方式工作，以便从线程切换到线程池不需要对使用我们 API 的代码进行大的更改。代码清单21-12显示了我们想要使用的 `ThreadPool` 结构体的假设接口，而不是 `thread::spawn`。

<Listing number="21-12" file-name="src/main.rs" caption="我们理想的 `ThreadPool` 接口">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-12/src/main.rs:here}}
```

</Listing>

我们使用 `ThreadPool::new` 创建一个具有可配置数量线程的新线程池，在这种情况下是四个。然后，在 `for` 循环中，`pool.execute` 具有与 `thread::spawn` 相似的接口，因为它接受池应该为每个流运行的闭包。我们需要实现 `pool.execute`，以便它接受闭包并将其提供给池中的线程运行。此代码还不会编译，但我们会尝试，以便编译器可以指导我们如何修复它。

<!-- Old headings. Do not remove or links may break. -->

<a id="building-the-threadpool-struct-using-compiler-driven-development"></a>

#### 使用编译器驱动开发构建 `ThreadPool`

对 _src/main.rs_ 进行代码清单21-12中的更改，然后让我们使用 `cargo check` 的编译器错误来驱动我们的开发。这是我们得到的第一个错误：

```console
{{#include ../listings/ch21-web-server/listing-21-12/output.txt}}
```

太好了！此错误告诉我们我们需要一个 `ThreadPool` 类型或模块，所以我们现在构建一个。我们的 `ThreadPool` 实现将独立于我们的 Web 服务器正在做的工作类型。因此，让我们将 `hello` crate 从二进制 crate 切换为库 crate，以保存我们的 `ThreadPool` 实现。切换到库 crate 后，我们还可以为任何我们想要使用线程池执行的工作使用单独的线程池库，而不仅仅是为 Web 请求提供服务。

创建一个包含以下内容的 _src/lib.rs_ 文件，这是我们现在可以拥有的 `ThreadPool` 结构体的最简单定义：

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/lib.rs}}
```

</Listing>


然后，编辑 _main.rs_ 文件，通过将以下代码添加到 _src/main.rs_ 的顶部，将 `ThreadPool` 从库 crate 引入作用域：

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/main.rs:here}}
```

</Listing>

此代码仍然不会工作，但让我们再次检查它以获取我们需要解决的下一个错误：

```console
{{#include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/output.txt}}
```

此错误表明接下来我们需要为 `ThreadPool` 创建一个名为 `new` 的关联函数。我们还知道 `new` 需要有一个参数，该参数可以接受 `4` 作为参数，并且应该返回 `ThreadPool` 实例。让我们实现具有这些特征的最简单的 `new` 函数：

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/src/lib.rs}}
```

</Listing>

我们选择 `usize` 作为 `size` 参数的类型，因为我们知道负数线程没有任何意义。我们还知道我们将使用这个 `4` 作为线程集合中的元素数量，这就是 `usize` 类型的用途，如第3章中[“整数类型”][integer-types]<!-- ignore -->部分所讨论的。

让我们再次检查代码：

```console
{{#include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/output.txt}}
```

现在错误是因为我们在 `ThreadPool` 上没有 `execute` 方法。回想一下[“创建有限数量的线程”](#creating-a-finite-number-of-threads)<!-- ignore -->部分，我们决定我们的线程池应该具有与 `thread::spawn` 相似的接口。此外，我们将实现 `execute` 函数，以便它接受给定的闭包并将其提供给池中的空闲线程运行。

我们将在 `ThreadPool` 上定义 `execute` 方法，以接受闭包作为参数。回想一下第13章中的[“将捕获的值移出闭包”][moving-out-of-closures]<!-- ignore -->，我们可以使用三种不同的 traits 接受闭包作为参数：`Fn`、`FnMut` 和 `FnOnce`。我们需要决定在这里使用哪种闭包。我们知道我们最终会做一些类似于标准库 `thread::spawn` 实现的事情，所以我们可以查看 `thread::spawn` 的签名对其参数有什么限制。文档向我们显示以下内容：

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

`F` 类型参数是我们这里关心的；`T` 类型参数与返回值相关，我们不关心它。我们可以看到 `spawn` 使用 `FnOnce` 作为 `F` 的 trait 限制。这可能也是我们想要的，因为我们最终会将我们在 `execute` 中获得的参数传递给 `spawn`。我们可以进一步确信 `FnOnce` 是我们想要使用的 trait，因为运行请求的线程只会执行该请求的闭包一次，这与 `FnOnce` 中的 `Once` 匹配。

`F` 类型参数还具有 trait 限制 `Send` 和生命周期限制 `'static`，这在我们的情况下很有用：我们需要 `Send` 将闭包从一个线程传输到另一个线程，并且需要 `'static`，因为我们不知道线程执行需要多长时间。让我们在 `ThreadPool` 上创建一个 `execute` 方法，该方法将接受具有这些限制的类型 `F` 的通用参数：

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-03-define-execute/src/lib.rs:here}}
```

</Listing>

我们仍然在 `FnOnce` 之后使用 `()`，因为此 `FnOnce` 表示不接受参数并返回单元类型 `()` 的闭包。就像函数定义一样，可以从签名中省略返回类型，但即使我们没有参数，我们仍然需要括号。

同样，这是 `execute` 方法的最简单实现：它什么都不做，但我们只是试图使代码编译。让我们再次检查：

```console
{{#include ../listings/ch21-web-server/no-listing-03-define-execute/output.txt}}
```

它编译了！但请注意，如果你尝试 `cargo run` 并在浏览器中发出请求，你会在浏览器中看到我们在本章开头看到的错误。我们的库实际上还没有调用传递给 `execute` 的闭包！

> 注意：你可能听说过关于具有严格编译器的语言（如 Haskell 和 Rust）的说法：“如果代码编译，它就可以工作。”但这句话并不普遍正确。我们的项目编译了，但它绝对什么都不做！如果我们要构建一个真正的、完整的项目，这将是一个开始编写单元测试以检查代码编译_并且_具有我们想要的行为的好时机。

考虑：如果我们执行 future 而不是闭包，这里会有什么不同？

#### 在 `new` 中验证线程数

我们没有对 `new` 和 `execute` 的参数做任何事情。让我们用我们想要的行为实现这些函数的主体。首先，让我们考虑 `new`。之前我们为 `size` 参数选择了无符号类型，因为具有负数线程的池没有任何意义。但是，具有零个线程的池也没有意义，但零是完全有效的 `usize`。我们将在返回 `ThreadPool` 实例之前添加代码来检查 `size` 是否大于零，并且如果它收到零，我们将使用 `assert!` 宏使程序 panic，如代码清单21-13所示。

<Listing number="21-13" file-name="src/lib.rs" caption="如果 `size` 为零，实现 `ThreadPool::new` 以 panic">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-13/src/lib.rs:here}}
```

</Listing>

我们还使用文档注释为我们的 `ThreadPool` 添加了一些文档。请注意，我们遵循了良好的文档实践，添加了一个部分来说明我们的函数可能 panic 的情况，如第14章所讨论的。尝试运行 `cargo doc --open` 并单击 `ThreadPool` 结构体以查看为 `new` 生成的文档！

除了像我们在这里所做的那样添加 `assert!` 宏之外，我们可以将 `new` 更改为 `build` 并返回一个 `Result`，就像我们在代码清单12-9中的 I/O 项目中为 `Config::build` 所做的那样。但是在这种情况下，我们决定尝试创建没有任何线程的线程池应该是一个不可恢复的错误。如果你感到雄心勃勃，尝试编写一个名为 `build` 的函数，其签名如下，以与 `new` 函数进行比较：

```rust,ignore
pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {
```

#### 创建存储线程的空间

现在我们有了一个方法来知道我们有有效数量的线程要存储在池中，我们可以在返回结构体之前创建这些线程并将它们存储在 `ThreadPool` 结构体中。但是我们如何“存储”线程？让我们再看看 `thread::spawn` 签名：

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

`spawn` 函数返回一个 `JoinHandle<T>`，其中 `T` 是闭包返回的类型。让我们也尝试使用 `JoinHandle`，看看会发生什么。在我们的情况下，我们传递给线程池的闭包将处理连接并且不返回任何内容，所以 `T` 将是单元类型 `()`。

代码清单21-14中的代码会编译，但尚未创建任何线程。我们更改了 `ThreadPool` 的定义以保存 `thread::JoinHandle<()>` 实例的向量，用 `size` 的容量初始化向量，设置一个 `for` 循环来运行一些代码以创建线程，并返回包含它们的 `ThreadPool` 实例。

<Listing number="21-14" file-name="src/lib.rs" caption="为 `ThreadPool` 创建一个向量以保存线程">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-14/src/lib.rs:here}}
```

</Listing>

我们已将 `std::thread` 引入库 crate 的作用域，因为我们在 `ThreadPool` 中使用的向量中的项目类型是 `thread::JoinHandle`。

一旦收到有效大小，我们的 `ThreadPool` 会创建一个新向量，可以保存 `size` 个项目。`with_capacity` 函数执行与 `Vec::new` 相同的任务，但有一个重要区别：它预先分配向量中的空间。因为我们知道需要在向量中存储 `size` 个元素，所以预先进行此分配比使用 `Vec::new` 稍微更高效，后者在插入元素时会调整自身大小。

当你再次运行 `cargo check` 时，它应该会成功。

<!-- Old headings. Do not remove or links may break. -->
<a id ="a-worker-struct-responsible-for-sending-code-from-the-threadpool-to-a-thread"></a>

#### 从 `ThreadPool` 向线程发送代码

我们在代码清单21-14中的 `for` 循环中留下了关于创建线程的注释。在这里，我们将看看我们如何实际创建线程。标准库提供 `thread::spawn` 作为创建线程的方式，`thread::spawn` 期望获取线程应该在创建后立即运行的代码。但是，在我们的情况下，我们希望创建线程并让它们_等待_我们稍后发送的代码。标准库的线程实现不包括任何这样做的方法；我们必须手动实现它。

我们将通过在 `ThreadPool` 和将管理此新行为的线程之间引入新的数据结构来实现此行为。我们将此数据结构称为 _Worker_，这是池实现中的常用术语。`Worker` 拾取需要运行的代码并在其线程中运行该代码。

想想餐厅厨房里工作的人：工人等待顾客的订单进来，然后他们负责接受这些订单并完成它们。

我们将存储 `Worker` 结构体的实例，而不是在线程池中存储 `JoinHandle<()>` 实例的向量。每个 `Worker` 将存储一个 `JoinHandle<()>` 实例。然后，我们将在 `Worker` 上实现一个方法，该方法将接受要运行的代码闭包并将其发送到已运行的线程执行。我们还将为每个 `Worker` 提供一个 `id`，以便在记录或调试时我们可以区分池中 `Worker` 的不同实例。

当我们创建 `ThreadPool` 时，将发生以下新过程。在我们以这种方式设置 `Worker` 之后，我们将实现将闭包发送到线程的代码：

1. 定义一个 `Worker` 结构体，它保存 `id` 和 `JoinHandle<()>`。
2. 更改 `ThreadPool` 以保存 `Worker` 实例的向量。
3. 定义一个 `Worker::new` 函数，它接受 `id` 编号并返回一个保存 `id` 和用空闭包生成的线程的 `Worker` 实例。
4. 在 `ThreadPool::new` 中，使用 `for` 循环计数器生成 `id`，用该 `id` 创建新的 `Worker`，并将 `Worker` 存储在向量中。

如果你准备好接受挑战，在查看代码清单21-15中的代码之前，尝试自己实现这些更改。

准备好了吗？这里是代码清单21-15，其中包含进行上述修改的一种方法。

<Listing number="21-15" file-name="src/lib.rs" caption="修改 `ThreadPool` 以保存 `Worker` 实例，而不是直接保存线程">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-15/src/lib.rs:here}}
```

</Listing>

我们将 `ThreadPool` 上的字段名称从 `threads` 更改为 `workers`，因为它现在保存 `Worker` 实例而不是 `JoinHandle<()>` 实例。我们使用 `for` 循环中的计数器作为 `Worker::new` 的参数，并将每个新的 `Worker` 存储在名为 `workers` 的向量中。

外部代码（如我们在 _src/main.rs_ 中的服务器）不需要知道关于在 `ThreadPool` 内使用 `Worker` 结构体的实现细节，因此我们使 `Worker` 结构体及其 `new` 函数为私有。`Worker::new` 函数使用我们给它的 `id` 并存储通过使用空闭包生成新线程创建的 `JoinHandle<()>` 实例。

> 注意：如果操作系统无法创建线程，因为没有足够的系统资源，`thread::spawn` 会 panic。这将导致我们的整个服务器 panic，即使某些线程的创建可能成功。为了简单起见，这种行为是可以的，但在生产线程池实现中，你可能希望使用 [`std::thread::Builder`][builder]<!-- ignore -->及其 [`spawn`][builder-spawn]<!-- ignore -->方法，该方法返回 `Result`。

此代码将编译并将我们指定为 `ThreadPool::new` 参数的 `Worker` 实例数量存储起来。但是我们_仍然_没有处理我们在 `execute` 中得到的闭包。让我们接下来看看如何做到这一点。

#### 通过通道向线程发送请求

我们将解决的下一个问题是，传递给 `thread::spawn` 的闭包绝对什么都不做。目前，我们在 `execute` 方法中得到要执行的闭包。但是我们需要在创建 `ThreadPool` 期间创建每个 `Worker` 时给 `thread::spawn` 一个要运行的闭包。

我们希望我们刚刚创建的 `Worker` 结构体从 `ThreadPool` 中保存的队列中获取要运行的代码，并将该代码发送到其线程运行。

我们在第16章中学到的通道——在线程之间通信的简单方法——非常适合这个用例。我们将使用通道作为作业队列，`execute` 将从 `ThreadPool` 向 `Worker` 实例发送作业，这些实例将作业发送到其线程。这是计划：

1. `ThreadPool` 将创建一个通道并保留发送者。
2. 每个 `Worker` 将保留接收者。
3. 我们将创建一个新的 `Job` 结构体，它将保存我们想要通过通道发送的闭包。
4. `execute` 方法将通过发送者发送它想要执行的作业。
5. 在其线程中，`Worker` 将循环其接收者并执行它收到的任何作业的闭包。

让我们首先在 `ThreadPool::new` 中创建一个通道，并在 `ThreadPool` 实例中保留发送者，如代码清单21-16所示。`Job` 结构体现在不保存任何内容，但将是我们通过通道发送的项目类型。

<Listing number="21-16" file-name="src/lib.rs" caption="修改 `ThreadPool` 以存储传输 `Job` 实例的通道的发送者">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-16/src/lib.rs:here}}
```

</Listing>

在 `ThreadPool::new` 中，我们创建新的通道并让池保留发送者。这将成功编译。

让我们尝试将通道的接收者传递给每个 `Worker`，因为线程池创建通道。我们知道我们想要在 `Worker` 实例生成的线程中使用接收者，所以我们在闭包中引用 `receiver` 参数。代码清单21-17中的代码还不会完全编译。

<Listing number="21-17" file-name="src/lib.rs" caption="将接收者传递给每个 `Worker`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-17/src/lib.rs:here}}
```

</Listing>

我们进行了一些小而直接的更改：我们将接收者传递给 `Worker::new`，然后我们在闭包内使用它。

当我们尝试检查此代码时，我们得到此错误：

```console
{{#include ../listings/ch21-web-server/listing-21-17/output.txt}}
```

代码试图将 `receiver` 传递给多个 `Worker` 实例。这不会工作，正如你从第16章中回忆的那样：Rust 提供的通道实现是多_生产者_、单_消费者_。这意味着我们不能只是克隆通道的消费端来修复此代码。我们也不想多次向多个消费者发送消息；我们想要一个消息列表，其中多个 `Worker` 实例使得每条消息都被处理一次。

此外，从通道队列中取下一个作业涉及改变 `receiver`，所以线程需要一种安全的方式来共享和修改 `receiver`；否则，我们可能会遇到竞争条件（如第16章所涵盖的）。

回想一下第16章中讨论的线程安全智能指针：为了在多个线程之间共享所有权并允许线程改变值，我们需要使用 `Arc<Mutex<T>>`。`Arc` 类型将让多个 `Worker` 实例拥有接收者，`Mutex` 将确保一次只有一个 `Worker` 从接收者获取作业。代码清单21-18显示了我们需要进行的更改。

<Listing number="21-18" file-name="src/lib.rs" caption="使用 `Arc` 和 `Mutex` 在 `Worker` 实例之间共享接收者">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-18/src/lib.rs:here}}
```

</Listing>

在 `ThreadPool::new` 中，我们将接收者放在 `Arc` 和 `Mutex` 中。对于每个新的 `Worker`，我们克隆 `Arc` 以增加引用计数，以便 `Worker` 实例可以共享接收者的所有权。

通过这些更改，代码编译了！我们快到了！

#### 实现 `execute` 方法

让我们最终在 `ThreadPool` 上实现 `execute` 方法。我们还将把 `Job` 从结构体更改为类型别名，用于保存 `execute` 收到的闭包类型的 trait 对象。如第20章中[“类型同义词和类型别名”][type-aliases]<!-- ignore -->部分所讨论的，类型别名允许我们缩短长类型以便于使用。查看代码清单21-19。

<Listing number="21-19" file-name="src/lib.rs" caption="为保存每个闭包的 `Box` 创建 `Job` 类型别名，然后通过通道发送作业">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-19/src/lib.rs:here}}
```

</Listing>

在创建使用我们在 `execute` 中得到的闭包的新 `Job` 实例后，我们通过通道的发送端发送该作业。我们在 `send` 上调用 `unwrap` 以处理发送失败的情况。例如，如果我们停止所有线程执行，这可能会发生，这意味着接收端已停止接收新消息。目前，我们无法停止线程执行：只要池存在，我们的线程就会继续执行。我们使用 `unwrap` 的原因是我们知道失败情况不会发生，但编译器不知道这一点。

但我们还没有完全完成！在 `Worker` 中，我们传递给 `thread::spawn` 的闭包仍然只_引用_通道的接收端。相反，我们需要闭包永远循环，向通道的接收端请求作业，并在收到作业时运行它。让我们对 `Worker::new` 进行代码清单21-20中显示的更改。

<Listing number="21-20" file-name="src/lib.rs" caption="在 `Worker` 实例的线程中接收和执行作业">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-20/src/lib.rs:here}}
```

</Listing>

在这里，我们首先在 `receiver` 上调用 `lock` 以获取互斥锁，然后我们调用 `unwrap` 以在任何错误时 panic。如果互斥锁处于_中毒_状态，获取锁可能会失败，如果其他线程在持有锁而不是释放锁时 panic，就会发生这种情况。在这种情况下，调用 `unwrap` 使此线程 panic 是正确的操作。随意将此 `unwrap` 更改为具有对你来说有意义的错误消息的 `expect`。

如果我们获得互斥锁，我们调用 `recv` 从通道接收 `Job`。最后的 `unwrap` 也会移过任何错误，如果持有发送者的线程已关闭，这可能会发生，类似于如果接收者关闭，`send` 方法返回 `Err` 的方式。

对 `recv` 的调用会阻塞，所以如果还没有作业，当前线程将等待直到作业可用。`Mutex<T>` 确保一次只有一个 `Worker` 线程尝试请求作业。

我们的线程池现在处于工作状态！给它一个 `cargo run` 并发出一些请求：

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-20
cargo run
make some requests to 127.0.0.1:7878
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
warning: field `workers` is never read
 --> src/lib.rs:7:5
  |
6 | pub struct ThreadPool {
  |            ---------- field in this struct
7 |     workers: Vec<Worker>,
  |     ^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default

warning: fields `id` and `thread` are never read
  --> src/lib.rs:48:5
   |
47 | struct Worker {
   |        ------ fields in this struct
48 |     id: usize,
   |     ^^
49 |     thread: thread::JoinHandle<()>,
   |     ^^^^^^
  |
warning: `hello` (lib) generated 2 warnings
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 4.91s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
```

成功！我们现在有一个异步执行连接的线程池。创建的线程永远不会超过四个，所以如果服务器收到大量请求，我们的系统不会过载。如果我们对 _/sleep_ 发出请求，服务器将能够通过让另一个线程运行它们来为其他请求提供服务。

> 注意：如果你同时在多个浏览器窗口中打开 _/sleep_，它们可能会以五秒间隔一次加载一个。某些 Web 浏览器由于缓存原因会顺序执行同一请求的多个实例。此限制不是由我们的 Web 服务器引起的。

这是暂停并考虑如果我们在代码清单21-18、21-19和21-20中使用 future 而不是闭包来完成工作，代码会有什么不同的好时机。哪些类型会改变？方法签名会有什么不同（如果有的话）？代码的哪些部分会保持不变？

在学习第17章和第19章中的 `while let` 循环后，你可能想知道为什么我们没有将 `Worker` 线程代码写成代码清单21-21中显示的那样。

<Listing number="21-21" file-name="src/lib.rs" caption="使用 `while let` 的 `Worker::new` 的替代实现">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-21/src/lib.rs:here}}
```

</Listing>

此代码编译并运行，但不会导致所需的线程行为：慢速请求仍会导致其他请求等待处理。原因有些微妙：`Mutex` 结构体没有公共 `unlock` 方法，因为锁的所有权基于 `lock` 方法返回的 `LockResult<MutexGuard<T>>` 内 `MutexGuard<T>` 的生命周期。在编译时，借用检查器然后可以强制执行规则，即除非我们持有锁，否则无法访问由 `Mutex` 保护的资源。但是，如果我们不注意 `MutexGuard<T>` 的生命周期，此实现也可能导致锁被持有的时间比预期的长。

代码清单21-20中使用 `let job = receiver.lock().unwrap().recv().unwrap();` 的代码有效，因为使用 `let`，等号右侧表达式中使用的任何临时值在 `let` 语句结束时立即被丢弃。但是，`while let`（以及 `if let` 和 `match`）不会丢弃临时值，直到关联块结束。在代码清单21-21中，锁在调用 `job()` 的持续时间内保持持有，这意味着其他 `Worker` 实例无法接收作业。

[type-aliases]: ch20-03-advanced-types.html#type-synonyms-and-type-aliases
[integer-types]: ch03-02-data-types.html#integer-types
[moving-out-of-closures]: ch13-01-closures.html#moving-captured-values-out-of-closures
[builder]: ../std/thread/struct.Builder.html
[builder-spawn]: ../std/thread/struct.Builder.html#method.spawn
