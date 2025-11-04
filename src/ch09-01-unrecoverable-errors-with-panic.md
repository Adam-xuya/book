## 使用 `panic!` 处理不可恢复错误

有时你的代码中会发生不好的事情，而你对此无能为力。在这些情况下，Rust 有 `panic!` 宏。实际上有两种方式会导致 panic：通过采取导致我们的代码 panic 的操作（例如访问超出数组末尾）或通过显式调用 `panic!` 宏。在这两种情况下，我们都会导致程序 panic。默认情况下，这些 panic 将打印失败消息、展开、清理堆栈并退出。通过环境变量，你也可以让 Rust 在 panic 发生时显示调用堆栈，以便更容易追踪 panic 的来源。

> ### 展开堆栈或在响应 Panic 时中止
>
> 默认情况下，当 panic 发生时，程序开始 _展开_，这意味着 Rust 沿着堆栈向上回溯并清理它遇到的每个函数的数据。但是，回溯和清理是大量的工作。因此，Rust 允许你选择立即 _中止_ 的替代方案，这会在不清理的情况下结束程序。
>
> 程序正在使用的内存将需要由操作系统清理。如果你的项目中需要使生成的二进制文件尽可能小，你可以通过在 _Cargo.toml_ 文件的适当 `[profile]` 部分添加 `panic = 'abort'` 来从展开切换到在 panic 时中止。例如，如果你想在发布模式下在 panic 时中止，添加这个：
>
> ```toml
> [profile.release]
> panic = 'abort'
> ```

让我们尝试在一个简单程序中调用 `panic!`：

<Listing file-name="src/main.rs">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-01-panic/src/main.rs}}
```

</Listing>

当你运行程序时，你会看到类似这样的内容：

```console
{{#include ../listings/ch09-error-handling/no-listing-01-panic/output.txt}}
```

对 `panic!` 的调用导致最后两行中包含的错误消息。第一行显示我们的 panic 消息以及源代码中发生 panic 的位置：_src/main.rs:2:5_ 表示它是我们 _src/main.rs_ 文件的第二行、第五个字符。

在这种情况下，指示的行是我们代码的一部分，如果我们转到该行，我们会看到 `panic!` 宏调用。在其他情况下，`panic!` 调用可能在我们的代码调用的代码中，错误消息报告的文件名和行号将是其他人调用 `panic!` 宏的代码，而不是最终导致 `panic!` 调用的我们代码的行。

<!-- Old headings. Do not remove or links may break. -->

<a id="using-a-panic-backtrace"></a>

我们可以使用 `panic!` 调用来自的函数的回溯来找出导致问题的代码部分。要了解如何使用 `panic!` 回溯，让我们看另一个例子，看看当 `panic!` 调用来自库时的情况，因为我们的代码中的 bug 而不是我们的代码直接调用宏。代码清单 9-1 有一些代码尝试访问向量中超出有效索引范围的索引。

<Listing number="9-1" file-name="src/main.rs" caption="尝试访问超出向量末尾的元素，这将导致调用 `panic!`">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-01/src/main.rs}}
```

</Listing>

这里，我们尝试访问向量的第 100 个元素（索引为 99，因为索引从零开始），但向量只有三个元素。在这种情况下，Rust 会 panic。使用 `[]` 应该返回一个元素，但如果你传递无效索引，Rust 无法返回正确的元素。

在 C 中，尝试读取超出数据结构末尾的内容是未定义行为。你可能会得到内存中与该数据结构中的元素对应的位置上的任何内容，即使内存不属于该结构。这称为 _缓冲区过度读取_，如果攻击者能够以读取他们不应该被允许的数据的方式操纵索引，这可能导致安全漏洞。

为了保护你的程序免受此类漏洞的影响，如果你尝试读取不存在的索引处的元素，Rust 将停止执行并拒绝继续。让我们试试看：

```console
{{#include ../listings/ch09-error-handling/listing-09-01/output.txt}}
```

此错误指向我们 _main.rs_ 的第 4 行，我们尝试访问 `v` 中索引 99 的向量。

`note:` 行告诉我们，我们可以设置 `RUST_BACKTRACE` 环境变量来获取导致错误的确切原因的回溯。_回溯_ 是已调用以到达此点的所有函数的列表。Rust 中的回溯工作方式与其他语言相同：读取回溯的关键是从顶部开始阅读，直到看到你编写的文件。那是问题起源的地方。该位置之上的行是你的代码调用的代码；该位置之下的行是调用你的代码的代码。这些前后行可能包括核心 Rust 代码、标准库代码或你正在使用的 crate。让我们尝试通过将 `RUST_BACKTRACE` 环境变量设置为除 `0` 之外的任何值来获取回溯。代码清单 9-2 显示了与你将看到的类似的输出。

<!-- manual-regeneration
cd listings/ch09-error-handling/listing-09-01
RUST_BACKTRACE=1 cargo run
copy the backtrace output below
check the backtrace number mentioned in the text below the listing
-->

<Listing number="9-2" caption="设置环境变量 `RUST_BACKTRACE` 时显示的 `panic!` 调用生成的回溯">

```console
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at src/main.rs:4:6:
index out of bounds: the len is 3 but the index is 99
stack backtrace:
   0: rust_begin_unwind
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/std/src/panicking.rs:692:5
   1: core::panicking::panic_fmt
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:75:14
   2: core::panicking::panic_bounds_check
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:273:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:274:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:16:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/alloc/src/vec/mod.rs:3361:9
   6: panic::main
             at ./src/main.rs:4:6
   7: core::ops::function::FnOnce::call_once
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/ops/function.rs:250:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

</Listing>

这是很多输出！你看到的确切输出可能因操作系统和 Rust 版本而异。为了获得包含此信息的回溯，必须启用调试符号。在使用 `cargo build` 或 `cargo run` 而不使用 `--release` 标志时，默认启用调试符号，就像我们这里一样。

在代码清单 9-2 的输出中，回溯的第 6 行指向我们项目中导致问题的行：_src/main.rs_ 的第 4 行。如果我们不希望程序 panic，我们应该从提到我们编写的文件的第一行指向的位置开始调查。在代码清单 9-1 中，我们故意编写了会导致 panic 的代码，修复 panic 的方法是不要请求超出向量索引范围的元素。当你的代码将来 panic 时，你需要弄清楚代码正在使用什么值采取什么操作导致 panic，以及代码应该做什么。

我们将在本章后面的["是否 `panic!`"][to-panic-or-not-to-panic]<!-- ignore -->部分回到 `panic!` 以及我们应该何时使用 `panic!` 来处理错误情况。接下来，我们将看看如何使用 `Result` 从错误中恢复。

[to-panic-or-not-to-panic]: ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic
