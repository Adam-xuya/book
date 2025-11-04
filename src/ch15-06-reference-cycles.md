## 引用循环可能泄漏内存

Rust 的内存安全保证使意外创建永远不会被清理的内存（称为 _内存泄漏_）变得困难，但并非不可能。完全防止内存泄漏不是 Rust 的保证之一，这意味着内存泄漏在 Rust 中是内存安全的。我们可以看到 Rust 通过使用 `Rc<T>` 和 `RefCell<T>` 允许内存泄漏：可能创建项在循环中相互引用的引用。这会产生内存泄漏，因为循环中每个项的引用计数永远不会达到 0，并且值永远不会被丢弃。

### 创建引用循环

让我们看看引用循环可能如何发生以及如何防止它，从代码清单 15-25 中 `List` 枚举的定义和 `tail` 方法开始。

<Listing number="15-25" file-name="src/main.rs" caption="保存 `RefCell<T>` 的 cons 列表定义，以便我们可以修改 `Cons` 变体引用的内容">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-25/src/main.rs:here}}
```

</Listing>

我们正在使用代码清单 15-5 中 `List` 定义的另一个变体。`Cons` 变体中的第二个元素现在是 `RefCell<Rc<List>>`，这意味着我们想要修改 `Cons` 变体指向的 `List` 值，而不是像我们在代码清单 15-24 中所做的那样修改 `i32` 值的能力。我们还添加一个 `tail` 方法，以便如果我们有 `Cons` 变体，我们可以方便地访问第二项。

在代码清单 15-26 中，我们添加一个使用代码清单 15-25 中定义的 `main` 函数。此代码在 `a` 中创建一个列表，在 `b` 中创建一个指向 `a` 中列表的列表。然后，它修改 `a` 中的列表以指向 `b`，创建引用循环。沿途有 `println!` 语句，以显示在此过程的各个点引用计数是什么。

<Listing number="15-26" file-name="src/main.rs" caption="创建两个相互指向的 `List` 值的引用循环">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-26/src/main.rs:here}}
```

</Listing>

我们在变量 `a` 中创建一个 `Rc<List>` 实例，保存一个初始列表为 `5, Nil` 的 `List` 值。然后，我们在变量 `b` 中创建一个 `Rc<List>` 实例，保存另一个包含值 `10` 并指向 `a` 中列表的 `List` 值。

我们修改 `a`，使其指向 `b` 而不是 `Nil`，创建循环。我们通过使用 `tail` 方法获取 `a` 中 `RefCell<Rc<List>>` 的引用来做到这一点，我们将其放在变量 `link` 中。然后，我们在 `RefCell<Rc<List>>` 上使用 `borrow_mut` 方法，将内部值从保存 `Nil` 值的 `Rc<List>` 更改为 `b` 中的 `Rc<List>`。

当我们运行此代码时，暂时保持最后一个 `println!` 注释掉，我们将得到此输出：

```console
{{#include ../listings/ch15-smart-pointers/listing-15-26/output.txt}}
```

当我们更改 `a` 中的列表以指向 `b` 后，`a` 和 `b` 中 `Rc<List>` 实例的引用计数都是 2。在 `main` 结束时，Rust 丢弃变量 `b`，这会将 `b` `Rc<List>` 实例的引用计数从 2 减少到 1。`Rc<List>` 在堆上的内存此时不会被丢弃，因为它的引用计数是 1，不是 0。然后，Rust 丢弃 `a`，这也会将 `a` `Rc<List>` 实例的引用计数从 2 减少到 1。此实例的内存也不能被丢弃，因为另一个 `Rc<List>` 实例仍引用它。分配给列表的内存将永远保持未收集。为了可视化此引用循环，我们创建了 Figure 15-4 中的图表。

<img alt="A rectangle labeled 'a' that points to a rectangle containing the integer 5. A rectangle labeled 'b' that points to a rectangle containing the integer 10. The rectangle containing 5 points to the rectangle containing 10, and the rectangle containing 10 points back to the rectangle containing 5, creating a cycle." src="img/trpl15-04.svg" class="center" />

<span class="caption">Figure 15-4: 列表 `a` 和 `b` 相互指向的引用循环</span>

如果你取消注释最后一个 `println!` 并运行程序，Rust 将尝试打印此循环，其中 `a` 指向 `b` 指向 `a` 等等，直到它溢出栈。

与真实世界的程序相比，在此示例中创建引用循环的后果并不是非常严重：在我们创建引用循环之后，程序立即结束。但是，如果更复杂的程序在循环中分配大量内存并长时间持有它，程序将使用比它需要的更多内存，并可能压倒系统，导致它耗尽可用内存。

创建引用循环并不容易完成，但也不是不可能的。如果你有包含 `Rc<T>` 值的 `RefCell<T>` 值或具有内部可变性和引用计数的类型的类似嵌套组合，你必须确保不创建循环；你不能依赖 Rust 来捕获它们。创建引用循环将是你程序中的逻辑错误，你应该使用自动化测试、代码审查和其他软件开发实践来最小化。

避免引用循环的另一个解决方案是重新组织数据结构，以便某些引用表达所有权而某些引用不表达所有权。因此，你可以有由一些所有权关系和一些非所有权关系组成的循环，只有所有权关系影响值是否可以被丢弃。在代码清单 15-25 中，我们总是希望 `Cons` 变体拥有它们的列表，所以重新组织数据结构是不可能的。让我们看看一个使用由父节点和子节点组成的图的示例，以查看非所有权关系何时是防止引用循环的适当方式。

<!-- Old headings. Do not remove or links may break. -->

<a id="preventing-reference-cycles-turning-an-rct-into-a-weakt"></a>

### 使用 `Weak<T>` 防止引用循环

到目前为止，我们已经演示了调用 `Rc::clone` 会增加 `Rc<T>` 实例的 `strong_count`，并且只有当 `Rc<T>` 实例的 `strong_count` 为 0 时才会清理它。你也可以通过调用 `Rc::downgrade` 并传递对 `Rc<T>` 的引用来创建对 `Rc<T>` 实例内值的弱引用。*强引用* 是你如何共享 `Rc<T>` 实例的所有权。*弱引用* 不表达所有权关系，它们的计数不影响 `Rc<T>` 实例何时被清理。它们不会导致引用循环，因为涉及一些弱引用的任何循环将在涉及的值的强引用计数为 0 时被打破。

当你调用 `Rc::downgrade` 时，你得到类型 `Weak<T>` 的智能指针。调用 `Rc::downgrade` 不会将 `Rc<T>` 实例中的 `strong_count` 增加 1，而是将 `weak_count` 增加 1。`Rc<T>` 类型使用 `weak_count` 来跟踪存在多少 `Weak<T>` 引用，类似于 `strong_count`。区别在于 `weak_count` 不需要为 0 才能清理 `Rc<T>` 实例。

因为 `Weak<T>` 引用的值可能已被丢弃，要对 `Weak<T>` 指向的值做任何事情，你必须确保值仍然存在。通过在 `Weak<T>` 实例上调用 `upgrade` 方法来做到这一点，该方法将返回 `Option<Rc<T>>`。如果 `Rc<T>` 值尚未被丢弃，你将得到 `Some` 结果；如果 `Rc<T>` 值已被丢弃，你将得到 `None` 结果。因为 `upgrade` 返回 `Option<Rc<T>>`，Rust 将确保处理 `Some` 情况和 `None` 情况，并且不会有无效指针。

作为示例，而不是使用项只知道下一项的列表，我们将创建一个树，其项知道它们的子项 _和_ 它们的父项。

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-a-tree-data-structure-a-node-with-child-nodes"></a>

#### 创建树数据结构

首先，我们将构建一个节点知道其子节点的树。我们将创建一个名为 `Node` 的结构体，它保存自己的 `i32` 值以及对子 `Node` 值的引用：

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:here}}
```

我们想要 `Node` 拥有其子节点，并且我们想要与变量共享该所有权，以便我们可以直接访问树中的每个 `Node`。为此，我们将 `Vec<T>` 项定义为类型 `Rc<Node>` 的值。我们也想要修改哪些节点是另一个节点的子节点，所以我们在 `children` 中有一个围绕 `Vec<Rc<Node>>` 的 `RefCell<T>`。

接下来，我们将使用我们的结构体定义并创建一个名为 `leaf` 的 `Node` 实例，值为 `3` 且没有子节点，以及另一个名为 `branch` 的实例，值为 `5` 且 `leaf` 作为其子节点之一，如代码清单 15-27 所示。

<Listing number="15-27" file-name="src/main.rs" caption="创建没有子节点的 `leaf` 节点和将 `leaf` 作为其子节点之一的 `branch` 节点">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:there}}
```

</Listing>

我们克隆 `leaf` 中的 `Rc<Node>` 并将其存储在 `branch` 中，这意味着 `leaf` 中的 `Node` 现在有两个所有者：`leaf` 和 `branch`。我们可以通过 `branch.children` 从 `branch` 到达 `leaf`，但没有办法从 `leaf` 到达 `branch`。原因是 `leaf` 没有对 `branch` 的引用，也不知道它们是相关的。我们希望 `leaf` 知道 `branch` 是它的父节点。我们接下来会这样做。

#### 添加从子节点到其父节点的引用

为了使子节点知道其父节点，我们需要向 `Node` 结构体定义添加 `parent` 字段。麻烦在于决定 `parent` 的类型应该是什么。我们知道它不能包含 `Rc<T>`，因为这将创建引用循环，其中 `leaf.parent` 指向 `branch` 且 `branch.children` 指向 `leaf`，这将导致它们的 `strong_count` 值永远不会为 0。

以另一种方式思考关系，父节点应该拥有其子节点：如果父节点被丢弃，其子节点也应该被丢弃。但是，子节点不应该拥有其父节点：如果我们丢弃子节点，父节点应该仍然存在。这是弱引用的情况！

所以，而不是 `Rc<T>`，我们将使 `parent` 的类型使用 `Weak<T>`，具体是 `RefCell<Weak<Node>>`。现在我们的 `Node` 结构体定义如下：

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:here}}
```

节点将能够引用其父节点但不拥有其父节点。在代码清单 15-28 中，我们更新 `main` 以使用此新定义，以便 `leaf` 节点将有一种方式引用其父节点 `branch`。

<Listing number="15-28" file-name="src/main.rs" caption="具有对其父节点 `branch` 的弱引用的 `leaf` 节点">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:there}}
```

</Listing>

创建 `leaf` 节点看起来类似于代码清单 15-27，除了 `parent` 字段：`leaf` 开始时没有父节点，所以我们创建一个新的、空的 `Weak<Node>` 引用实例。

此时，当我们尝试通过使用 `upgrade` 方法获取 `leaf` 父节点的引用时，我们得到 `None` 值。我们在第一个 `println!` 语句的输出中看到这一点：

```text
leaf parent = None
```

当我们创建 `branch` 节点时，它也会在 `parent` 字段中有一个新的 `Weak<Node>` 引用，因为 `branch` 没有父节点。我们仍然将 `leaf` 作为 `branch` 的子节点之一。一旦我们在 `branch` 中有 `Node` 实例，我们就可以修改 `leaf` 以给它一个对其父节点的 `Weak<Node>` 引用。我们在 `leaf` 的 `parent` 字段中的 `RefCell<Weak<Node>>` 上使用 `borrow_mut` 方法，然后我们使用 `Rc::downgrade` 函数从 `branch` 中的 `Rc<Node>` 创建对 `branch` 的 `Weak<Node>` 引用。

当我们再次打印 `leaf` 的父节点时，这次我们将得到一个保存 `branch` 的 `Some` 变体：现在 `leaf` 可以访问其父节点！当我们打印 `leaf` 时，我们也避免了最终在栈溢出中结束的循环，就像我们在代码清单 15-26 中那样；`Weak<Node>` 引用被打印为 `(Weak)`：

```text
leaf parent = Some(Node { value: 5, parent: RefCell { value: (Weak) },
children: RefCell { value: [Node { value: 3, parent: RefCell { value: (Weak) },
children: RefCell { value: [] } }] } })
```

缺乏无限输出表明此代码没有创建引用循环。我们也可以通过查看从调用 `Rc::strong_count` 和 `Rc::weak_count` 获得的值来判断这一点。

#### 可视化 `strong_count` 和 `weak_count` 的变化

让我们通过创建新的内部作用域并将 `branch` 的创建移动到该作用域来查看 `Rc<Node>` 实例的 `strong_count` 和 `weak_count` 值如何变化。通过这样做，我们可以看到当 `branch` 被创建然后在其超出作用域时被丢弃时会发生什么。修改如代码清单 15-29 所示。

<Listing number="15-29" file-name="src/main.rs" caption="在内部作用域中创建 `branch` 并检查强引用和弱引用计数">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-29/src/main.rs:here}}
```

</Listing>

创建 `leaf` 后，其 `Rc<Node>` 的强计数为 1，弱计数为 0。在内部作用域中，我们创建 `branch` 并将其与 `leaf` 关联，此时当我们打印计数时，`branch` 中的 `Rc<Node>` 的强计数为 1，弱计数为 1（对于 `leaf.parent` 指向 `branch` 的 `Weak<Node>`）。当我们在 `leaf` 中打印计数时，我们将看到它的强计数将为 2，因为 `branch` 现在有存储在 `branch.children` 中的 `leaf` 的 `Rc<Node>` 的克隆，但仍将具有弱计数 0。

当内部作用域结束时，`branch` 超出作用域，`Rc<Node>` 的强计数减少到 0，所以它的 `Node` 被丢弃。来自 `leaf.parent` 的弱计数 1 对 `Node` 是否被丢弃没有影响，所以我们不会得到任何内存泄漏！

如果我们尝试在作用域结束后访问 `leaf` 的父节点，我们将再次得到 `None`。在程序结束时，`leaf` 中的 `Rc<Node>` 的强计数为 1，弱计数为 0，因为变量 `leaf` 现在又是对 `Rc<Node>` 的唯一引用。

所有管理计数和值丢弃的逻辑都内置在 `Rc<T>` 和 `Weak<T>` 以及它们对 `Drop` trait 的实现中。通过在 `Node` 的定义中指定从子节点到其父节点的关系应该是 `Weak<T>` 引用，你能够使父节点指向子节点，反之亦然，而不会创建引用循环和内存泄漏。

## 总结

本章涵盖了如何使用智能指针做出与 Rust 使用常规引用默认做出的不同保证和权衡。`Box<T>` 类型具有已知大小并指向在堆上分配的数据。`Rc<T>` 类型跟踪对堆上数据的引用数量，以便数据可以有多个所有者。`RefCell<T>` 类型及其内部可变性为我们提供了一个类型，当我们需要不可变类型但需要更改该类型的内部值时可以使用；它还在运行时而不是编译时强制执行借用规则。

还讨论了 `Deref` 和 `Drop` trait，它们启用了智能指针的许多功能。我们探索了可能导致内存泄漏的引用循环以及如何使用 `Weak<T>` 防止它们。

如果本章引起了你的兴趣，并且你想实现自己的智能指针，请查看["The Rustonomicon"][nomicon] 以获取更多有用信息。

接下来，我们将讨论 Rust 中的并发。你甚至会学到一些新的智能指针。

[nomicon]: ../nomicon/index.html
