<!-- Old headings. Do not remove or links may break. -->

<a id="extensible-concurrency-with-the-sync-and-send-traits"></a>
<a id="extensible-concurrency-with-the-send-and-sync-traits"></a>

## 使用 `Send` 和 `Sync` 扩展并发

有趣的是，我们在本章中讨论的几乎所有并发功能都是标准库的一部分，而不是语言。你处理并发的选项不限于语言或标准库；你可以编写自己的并发功能或使用其他人编写的功能。

但是，嵌入在语言中而不是标准库中的关键并发概念是 `std::marker` trait `Send` 和 `Sync`。

<!-- Old headings. Do not remove or links may break. -->

<a id="allowing-transference-of-ownership-between-threads-with-send"></a>

### 在线程之间转移所有权

`Send` marker trait 表示实现 `Send` 的类型值的所有权可以在线程之间转移。几乎每个 Rust 类型都实现 `Send`，但有一些例外，包括 `Rc<T>`：这不能实现 `Send`，因为如果你克隆 `Rc<T>` 值并尝试将克隆的所有权转移到另一个线程，两个线程可能同时更新引用计数。因此，`Rc<T>` 被实现为在单线程情况中使用，你不想支付线程安全的性能损失。

因此，Rust 的类型系统和 trait 约束确保你永远不会意外地在线程之间不安全地发送 `Rc<T>` 值。当我们在代码清单 16-14 中尝试这样做时，我们得到了错误 `` the trait `Send` is not implemented for `Rc<Mutex<i32>>` ``。当我们切换到确实实现 `Send` 的 `Arc<T>` 时，代码编译了。

完全由 `Send` 类型组成的任何类型也会自动标记为 `Send`。几乎所有原始类型都是 `Send`，除了原始指针，我们将在第 20 章讨论。

<!-- Old headings. Do not remove or links may break. -->

<a id="allowing-access-from-multiple-threads-with-sync"></a>

### 从多个线程访问

`Sync` marker trait 表示实现 `Sync` 的类型从多个线程引用是安全的。换句话说，如果 `&T`（对 `T` 的不可变引用）实现 `Send`，任何类型 `T` 都实现 `Sync`，意味着引用可以安全地发送到另一个线程。类似于 `Send`，原始类型都实现 `Sync`，完全由实现 `Sync` 的类型组成的类型也实现 `Sync`。

智能指针 `Rc<T>` 也不实现 `Sync`，原因与它不实现 `Send` 的原因相同。`RefCell<T>` 类型（我们在第 15 章讨论过）和相关 `Cell<T>` 类型系列不实现 `Sync`。`RefCell<T>` 在运行时执行的借用检查的实现不是线程安全的。智能指针 `Mutex<T>` 实现 `Sync`，可以用于与多个线程共享访问，正如你在["对 `Mutex<T>` 的共享访问"][shared-access]<!-- ignore -->中看到的。

### 手动实现 `Send` 和 `Sync` 是不安全的

因为完全由其他实现 `Send` 和 `Sync` trait 的类型组成的类型也会自动实现 `Send` 和 `Sync`，我们不必手动实现这些 trait。作为 marker trait，它们甚至没有任何要实现的方法。它们只是用于强制执行与并发相关的不变量。

手动实现这些 trait 涉及实现 unsafe Rust 代码。我们将在第 20 章讨论使用 unsafe Rust 代码；现在，重要信息是构建不是由 `Send` 和 `Sync` 部分组成的新并发类型需要仔细思考以维护安全保证。["The Rustonomicon"][nomicon] 有更多关于这些保证以及如何维护它们的信息。

## 总结

这不是你在本书中最后一次看到并发：下一章专注于异步编程，第 21 章中的项目将在比这里讨论的较小示例更真实的情况下使用本章的概念。

如前所述，因为 Rust 处理并发的方式很少是语言的一部分，许多并发解决方案都是作为 crate 实现的。这些比标准库发展得更快，所以一定要在线搜索当前、最先进的 crate 以在多线程情况下使用。

Rust 标准库提供用于消息传递的通道和智能指针类型，如 `Mutex<T>` 和 `Arc<T>`，它们在并发上下文中可以安全使用。类型系统和借用检查器确保使用这些解决方案的代码不会以数据竞争或无效引用结束。一旦你的代码编译，你可以放心，它将在多个线程上愉快地运行，而不会有其他语言中常见的难以追踪的错误。并发编程不再是一个令人恐惧的概念：继续前进，让你的程序并发，无畏地！

[shared-access]: ch16-03-shared-state.html#shared-access-to-mutext
[nomicon]: ../nomicon/index.html
