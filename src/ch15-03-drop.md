## 使用 `Drop` Trait 运行清理代码

对智能指针模式重要的第二个 trait 是 `Drop`，它允许你自定义值即将超出作用域时发生的情况。你可以为任何类型提供 `Drop` trait 的实现，并且该代码可用于释放资源，如文件或网络连接。

我们在智能指针的上下文中介绍 `Drop`，因为 `Drop` trait 的功能在实现智能指针时几乎总是被使用。例如，当 `Box<T>` 被丢弃时，它将释放 box 指向的堆上的空间。

在某些语言中，对于某些类型，程序员必须在每次完成使用这些类型的实例时调用代码来释放内存或资源。示例包括文件句柄、套接字和锁。如果程序员忘记，系统可能会过载并崩溃。在 Rust 中，你可以指定每当值超出作用域时运行特定代码，编译器将自动插入此代码。因此，你不需要小心地将清理代码放在程序中特定类型实例完成使用的每个地方——你仍然不会泄漏资源！

你通过实现 `Drop` trait 来指定值超出作用域时运行的代码。`Drop` trait 要求你实现一个名为 `drop` 的方法，该方法接受对 `self` 的可变引用。为了查看 Rust 何时调用 `drop`，让我们现在用 `println!` 语句实现 `drop`。

代码清单 15-14 显示了一个 `CustomSmartPointer` 结构体，其唯一的自定义功能是它在实例超出作用域时将打印 `Dropping CustomSmartPointer!`，以显示 Rust 何时运行 `drop` 方法。

<Listing number="15-14" file-name="src/main.rs" caption="实现 `Drop` trait 的 `CustomSmartPointer` 结构体，我们将在这里放置清理代码">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-14/src/main.rs}}
```

</Listing>

`Drop` trait 包含在 prelude 中，所以我们不需要将其引入作用域。我们在 `CustomSmartPointer` 上实现 `Drop` trait，并为调用 `println!` 的 `drop` 方法提供实现。`drop` 方法的主体是你想要在类型实例超出作用域时运行的任何逻辑的位置。我们在这里打印一些文本，以直观地演示 Rust 何时调用 `drop`。

在 `main` 中，我们创建两个 `CustomSmartPointer` 实例，然后打印 `CustomSmartPointers created`。在 `main` 结束时，我们的 `CustomSmartPointer` 实例将超出作用域，Rust 将调用我们放在 `drop` 方法中的代码，打印我们的最终消息。注意，我们不需要显式调用 `drop` 方法。

当我们运行此程序时，我们将看到以下输出：

```console
{{#include ../listings/ch15-smart-pointers/listing-15-14/output.txt}}
```

Rust 在我们的实例超出作用域时自动为我们调用 `drop`，调用我们指定的代码。变量以与其创建相反的顺序被丢弃，所以 `d` 在 `c` 之前被丢弃。此示例的目的是为你提供 `drop` 方法如何工作的视觉指南；通常你会指定类型需要运行的清理代码而不是打印消息。

<!-- Old headings. Do not remove or links may break. -->

<a id="dropping-a-value-early-with-std-mem-drop"></a>

不幸的是，禁用自动 `drop` 功能并不简单。禁用 `drop` 通常不是必需的；`Drop` trait 的全部要点是它是自动处理的。但是，偶尔你可能想要提前清理值。一个例子是使用管理锁的智能指针：你可能想要强制释放锁的 `drop` 方法，以便同一作用域中的其他代码可以获取锁。Rust 不允许你手动调用 `Drop` trait 的 `drop` 方法；相反，如果你想在值的作用域结束之前强制丢弃值，你必须调用标准库提供的 `std::mem::drop` 函数。

通过修改代码清单 15-14 中的 `main` 函数来尝试手动调用 `Drop` trait 的 `drop` 方法将不起作用，如代码清单 15-15 所示。

<Listing number="15-15" file-name="src/main.rs" caption="尝试手动调用 `Drop` trait 的 `drop` 方法以提前清理">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-15/src/main.rs:here}}
```

</Listing>

当我们尝试编译此代码时，我们将得到此错误：

```console
{{#include ../listings/ch15-smart-pointers/listing-15-15/output.txt}}
```

此错误消息说明不允许我们显式调用 `drop`。错误消息使用术语 _析构函数_，这是清理实例的函数的一般编程术语。_析构函数_ 类似于 _构造函数_，它创建实例。Rust 中的 `drop` 函数是一个特定的析构函数。

Rust 不允许我们显式调用 `drop`，因为 Rust 仍然会在 `main` 结束时自动在值上调用 `drop`。这将导致双重释放错误，因为 Rust 将尝试两次清理同一个值。

当值超出作用域时，我们不能禁用 `drop` 的自动插入，也不能显式调用 `drop` 方法。所以，如果我们需要强制提前清理值，我们使用 `std::mem::drop` 函数。

`std::mem::drop` 函数与 `Drop` trait 中的 `drop` 方法不同。我们通过传递我们想要强制丢弃的值作为参数来调用它。该函数在 prelude 中，所以我们可以修改代码清单 15-15 中的 `main` 以调用 `drop` 函数，如代码清单 15-16 所示。

<Listing number="15-16" file-name="src/main.rs" caption="调用 `std::mem::drop` 以在值超出作用域之前显式丢弃值">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-16/src/main.rs:here}}
```

</Listing>

运行此代码将打印以下内容：

```console
{{#include ../listings/ch15-smart-pointers/listing-15-16/output.txt}}
```

文本 ``Dropping CustomSmartPointer with data `some data`!`` 在 `CustomSmartPointer created` 和 `CustomSmartPointer dropped before the end of main` 文本之间打印，显示在该点调用 `drop` 方法代码以丢弃 `c`。

你可以以多种方式使用 `Drop` trait 实现中指定的代码来使清理方便和安全：例如，你可以使用它来创建自己的内存分配器！使用 `Drop` trait 和 Rust 的所有权系统，你不必记住清理，因为 Rust 会自动完成。

你也不必担心由于意外清理仍在使用中的值而导致的问题：确保引用始终有效的所有权系统也确保 `drop` 只在值不再使用时调用一次。

现在我们已经检查了 `Box<T>` 和智能指针的一些特性，让我们看看标准库中定义的一些其他智能指针。
