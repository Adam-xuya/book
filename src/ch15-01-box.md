## 使用 `Box<T>` 指向堆上的数据

最直接的智能指针是 box，其类型写为 `Box<T>`。_Boxes_ 允许你将数据存储在堆上而不是栈上。保留在栈上的是指向堆数据的指针。请参阅第 4 章以回顾栈和堆之间的区别。

Boxes 没有性能开销，除了将数据存储在堆上而不是栈上。但它们也没有许多额外的功能。你最常在以下情况下使用它们：

- 当你有一个在编译时无法知道大小的类型，并且你想在需要确切大小的上下文中使用该类型的值时
- 当你有大量数据，并且你想转移所有权但确保在这样做时不会复制数据时
- 当你想拥有一个值，并且你只关心它是实现特定 trait 的类型而不是特定类型时

我们将在["使用 Boxes 启用递归类型"](#enabling-recursive-types-with-boxes)<!-- ignore -->中演示第一种情况。在第二种情况下，转移大量数据的所有权可能需要很长时间，因为数据在栈上复制。为了在这种情况下提高性能，我们可以将大量数据存储在堆上的 box 中。然后，只有少量指针数据在栈上复制，而它引用的数据保留在堆上的一个位置。第三种情况称为 _trait 对象_，第 18 章的["使用 Trait 对象抽象共享行为"][trait-objects]<!-- ignore -->专门讨论该主题。所以，你在这里学到的内容将在该部分再次应用！

<!-- Old headings. Do not remove or links may break. -->

<a id="using-boxt-to-store-data-on-the-heap"></a>

### 在堆上存储数据

在我们讨论 `Box<T>` 的堆存储用例之前，我们将涵盖语法以及如何与存储在 `Box<T>` 内的值交互。

代码清单 15-1 显示了如何使用 box 在堆上存储 `i32` 值。

<Listing number="15-1" file-name="src/main.rs" caption="使用 box 在堆上存储 `i32` 值">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-01/src/main.rs}}
```

</Listing>

我们将变量 `b` 定义为指向值 `5` 的 `Box` 的值，该值分配在堆上。此程序将打印 `b = 5`；在这种情况下，我们可以访问 box 中的数据，类似于如果此数据在栈上我们会如何访问。就像任何拥有的值一样，当 box 超出作用域时，如 `b` 在 `main` 结束时所做的，它将被释放。释放发生在 box（存储在栈上）和它指向的数据（存储在堆上）上。

将单个值放在堆上不是很有用，所以你不会经常以这种方式单独使用 boxes。在大多数情况下，在栈上拥有像单个 `i32` 这样的值（默认情况下存储在那里）更合适。让我们看看一个允许我们定义类型的案例，如果我们没有 boxes，我们将不被允许定义这些类型。

### 使用 Boxes 启用递归类型

_递归类型_ 的值可以具有另一个相同类型的值作为其自身的一部分。递归类型会带来问题，因为 Rust 需要在编译时知道类型占用多少空间。但是，递归类型值的嵌套理论上可以无限继续，所以 Rust 无法知道值需要多少空间。因为 boxes 有已知大小，我们可以通过在递归类型定义中插入 box 来启用递归类型。

作为递归类型的示例，让我们探索 cons 列表。这是函数式编程语言中常见的数据类型。我们将定义的 cons 列表类型除了递归之外都很简单；因此，我们在示例中使用的概念在你遇到涉及递归类型的更复杂情况时都会有用。

<!-- Old headings. Do not remove or links may break. -->

<a id="more-information-about-the-cons-list"></a>

#### 理解 Cons 列表

_cons 列表_ 是一种来自 Lisp 编程语言及其方言的数据结构，由嵌套对组成，是链表的 Lisp 版本。它的名字来自 Lisp 中的 `cons` 函数（_construct function_ 的缩写），它从其两个参数构造一个新对。通过在由值和另一个对组成的对上调用 `cons`，我们可以构造由递归对组成的 cons 列表。

例如，这是一个包含列表 `1, 2, 3` 的 cons 列表的伪代码表示，每个对在括号中：

```text
(1, (2, (3, Nil)))
```

cons 列表中的每个项包含两个元素：当前项的值和下一项的值。列表中的最后一项仅包含一个名为 `Nil` 的值，没有下一项。cons 列表通过递归调用 `cons` 函数产生。表示递归基本情况的标准名称是 `Nil`。注意，这与第 6 章中讨论的"null"或"nil"概念不同，后者是无效或缺失的值。

cons 列表不是 Rust 中常用的数据结构。在 Rust 中，当你有一个项列表时，大多数时候 `Vec<T>` 是更好的选择。其他更复杂的递归数据类型 _确实_ 在各种情况下有用，但通过在本章中从 cons 列表开始，我们可以探索 boxes 如何让我们定义递归数据类型而不会太分散注意力。

代码清单 15-2 包含 cons 列表的枚举定义。注意，此代码尚无法编译，因为 `List` 类型没有已知大小，我们将演示这一点。

<Listing number="15-2" file-name="src/main.rs" caption="第一次尝试定义枚举以表示 `i32` 值的 cons 列表数据结构">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-02/src/main.rs:here}}
```

</Listing>

> 注意：我们正在实现一个仅保存 `i32` 值的 cons 列表，用于此示例的目的。我们本可以使用泛型来实现它，正如我们在第 10 章中讨论的，以定义一个可以存储任何类型值的 cons 列表类型。

使用 `List` 类型存储列表 `1, 2, 3` 将看起来像代码清单 15-3 中的代码。

<Listing number="15-3" file-name="src/main.rs" caption="使用 `List` 枚举存储列表 `1, 2, 3`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-03/src/main.rs:here}}
```

</Listing>

第一个 `Cons` 值保存 `1` 和另一个 `List` 值。此 `List` 值是另一个 `Cons` 值，它保存 `2` 和另一个 `List` 值。此 `List` 值是另一个 `Cons` 值，它保存 `3` 和一个 `List` 值，最终是 `Nil`，这是表示列表结束的非递归变体。

如果我们尝试编译代码清单 15-3 中的代码，我们会得到代码清单 15-4 中显示的错误。

<Listing number="15-4" caption="尝试定义递归枚举时得到的错误">

```console
{{#include ../listings/ch15-smart-pointers/listing-15-03/output.txt}}
```

</Listing>

错误显示此类型"具有无限大小"。原因是我们用递归变体定义了 `List`：它直接保存自身的另一个值。因此，Rust 无法计算出需要多少空间来存储 `List` 值。让我们分解为什么我们会得到此错误。首先，我们将查看 Rust 如何决定需要多少空间来存储非递归类型的值。

#### 计算非递归类型的大小

回想我们在第 6 章讨论枚举定义时在代码清单 6-2 中定义的 `Message` 枚举：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

为了确定要为 `Message` 值分配多少空间，Rust 遍历每个变体以查看哪个变体需要最多空间。Rust 看到 `Message::Quit` 不需要任何空间，`Message::Move` 需要足够空间来存储两个 `i32` 值，等等。因为只会使用一个变体，`Message` 值所需的最大空间是存储其最大变体所需的空间。

这与 Rust 尝试确定递归类型（如代码清单 15-2 中的 `List` 枚举）需要多少空间时发生的情况形成对比。编译器首先查看 `Cons` 变体，它保存类型 `i32` 的值和类型 `List` 的值。因此，`Cons` 需要等于 `i32` 的大小加上 `List` 大小的空间量。为了计算出 `List` 类型需要多少内存，编译器查看变体，从 `Cons` 变体开始。`Cons` 变体保存类型 `i32` 的值和类型 `List` 的值，此过程无限继续，如 Figure 15-1 所示。

<img alt="An infinite Cons list: a rectangle labeled 'Cons' split into two smaller rectangles. The first smaller rectangle holds the label 'i32', and the second smaller rectangle holds the label 'Cons' and a smaller version of the outer 'Cons' rectangle. The 'Cons' rectangles continue to hold smaller and smaller versions of themselves until the smallest comfortably sized rectangle holds an infinity symbol, indicating that this repetition goes on forever." src="img/trpl15-01.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 15-1: 由无限 `Cons` 变体组成的无限 `List`</span>

<!-- Old headings. Do not remove or links may break. -->

<a id="using-boxt-to-get-a-recursive-type-with-a-known-size"></a>

#### 获得具有已知大小的递归类型

因为 Rust 无法计算出要为递归定义的类型分配多少空间，编译器给出一个带有此有用建议的错误：

<!-- manual-regeneration
after doing automatic regeneration, look at listings/ch15-smart-pointers/listing-15-03/output.txt and copy the relevant line
-->

```text
help: insert some indirection (e.g., a `Box`, `Rc`, or `&`) to break the cycle
  |
2 |     Cons(i32, Box<List>),
  |               ++++    +
```

在此建议中，_间接性_ 意味着我们不应该直接存储值，而应该更改数据结构以通过存储指向值的指针来间接存储值。

因为 `Box<T>` 是一个指针，Rust 总是知道 `Box<T>` 需要多少空间：指针的大小不会根据它指向的数据量而改变。这意味着我们可以在 `Cons` 变体内放置 `Box<T>` 而不是直接放置另一个 `List` 值。`Box<T>` 将指向下一个 `List` 值，该值将在堆上而不是在 `Cons` 变体内。从概念上讲，我们仍然有一个列表，由保存其他列表的列表创建，但此实现现在更像将项彼此相邻放置而不是彼此内部。

我们可以将代码清单 15-2 中的 `List` 枚举定义和代码清单 15-3 中的 `List` 用法更改为代码清单 15-5 中的代码，这将编译。

<Listing number="15-5" file-name="src/main.rs" caption="使用 `Box<T>` 以具有已知大小的 `List` 定义">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-05/src/main.rs}}
```

</Listing>

`Cons` 变体需要 `i32` 的大小加上存储 box 指针数据的空间。`Nil` 变体不存储任何值，所以它需要比 `Cons` 变体更少的栈空间。我们现在知道任何 `List` 值将占用 `i32` 的大小加上 box 指针数据的大小。通过使用 box，我们打破了无限递归链，所以编译器可以计算出需要多少空间来存储 `List` 值。Figure 15-2 显示了 `Cons` 变体现在的外观。

<img alt="A rectangle labeled 'Cons' split into two smaller rectangles. The first smaller rectangle holds the label 'i32', and the second smaller rectangle holds the label 'Box' with one inner rectangle that contains the label 'usize', representing the finite size of the box's pointer." src="img/trpl15-02.svg" class="center" />

<span class="caption">Figure 15-2: 不是无限大小的 `List`，因为 `Cons` 保存 `Box`</span>

Boxes 仅提供间接性和堆分配；它们没有任何其他特殊功能，就像我们将看到的其他智能指针类型一样。它们也没有这些特殊功能带来的性能开销，所以它们可以在像 cons 列表这样的情况中有用，其中间接性是我们需要的唯一功能。我们将在第 18 章中查看 boxes 的更多用例。

`Box<T>` 类型是一个智能指针，因为它实现了 `Deref` trait，这允许 `Box<T>` 值被像引用一样对待。当 `Box<T>` 值超出作用域时，box 指向的堆数据也会被清理，因为 `Drop` trait 的实现。这两个 trait 对我们将在本章其余部分讨论的其他智能指针类型提供的功能更加重要。让我们更详细地探索这两个 trait。

[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
