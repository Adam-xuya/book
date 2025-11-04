## `RefCell<T>` 和内部可变性模式

_内部可变性_ 是 Rust 中的设计模式，允许你在存在对该数据的不可变引用时改变数据；通常，此操作被借用规则禁止。为了改变数据，模式在数据结构内部使用 `unsafe` 代码来弯曲 Rust 通常管理变异和借用的规则。Unsafe 代码向编译器指示我们正在手动检查规则，而不是依赖编译器为我们检查它们；我们将在第 20 章更详细地讨论 unsafe 代码。

我们只能在我们能够确保运行时将遵循借用规则时使用使用内部可变性模式的类型，即使编译器无法保证这一点。然后，所涉及的 `unsafe` 代码被包装在安全 API 中，外部类型仍然是不可变的。

让我们通过查看遵循内部可变性模式的 `RefCell<T>` 类型来探索此概念。

<!-- Old headings. Do not remove or links may break. -->

<a id="enforcing-borrowing-rules-at-runtime-with-refcellt"></a>

### 在运行时强制执行借用规则

与 `Rc<T>` 不同，`RefCell<T>` 类型表示对其保存的数据的单一所有权。那么，什么使 `RefCell<T>` 与 `Box<T>` 这样的类型不同？回想你在第 4 章学到的借用规则：

- 在任何给定时间，你可以有 _一个_ 可变引用或任意数量的不可变引用（但不能同时拥有两者）。
- 引用必须始终有效。

对于引用和 `Box<T>`，借用规则的不变量在编译时强制执行。对于 `RefCell<T>`，这些不变量在 _运行时_ 强制执行。对于引用，如果你违反这些规则，你将得到编译器错误。对于 `RefCell<T>`，如果你违反这些规则，你的程序将 panic 并退出。

在编译时检查借用规则的优势是错误将在开发过程中更早被捕获，并且对运行时性能没有影响，因为所有分析都是预先完成的。由于这些原因，在编译时检查借用规则在大多数情况下是最好的选择，这就是为什么这是 Rust 的默认值。

在运行时而不是编译时检查借用规则的优势是，然后允许某些内存安全场景，它们会被编译时检查禁止。静态分析，如 Rust 编译器，本质上是保守的。代码的某些属性通过分析代码无法检测：最著名的例子是停机问题，这超出了本书的范围，但这是一个有趣的研究主题。

因为某些分析是不可能的，如果 Rust 编译器不能确定代码符合所有权规则，它可能会拒绝正确的程序；以这种方式，它是保守的。如果 Rust 接受了错误的程序，用户将无法信任 Rust 做出的保证。但是，如果 Rust 拒绝了正确的程序，程序员会感到不便，但不会发生灾难性的事情。`RefCell<T>` 类型在你确定代码遵循借用规则但编译器无法理解和保证这一点时很有用。

类似于 `Rc<T>`，`RefCell<T>` 仅用于单线程场景，如果你尝试在多线程上下文中使用它，将给出编译时错误。我们将在第 16 章讨论如何在多线程程序中获得 `RefCell<T>` 的功能。

以下是选择 `Box<T>`、`Rc<T>` 或 `RefCell<T>` 的原因总结：

- `Rc<T>` 实现同一数据的多个所有者；`Box<T>` 和 `RefCell<T>` 有单一所有者。
- `Box<T>` 允许在编译时检查的不可变或可变借用；`Rc<T>` 只允许在编译时检查的不可变借用；`RefCell<T>` 允许在运行时检查的不可变或可变借用。
- 因为 `RefCell<T>` 允许在运行时检查的可变借用，即使 `RefCell<T>` 是不可变的，你也可以改变 `RefCell<T>` 内部的值。

改变不可变值内部的值是内部可变性模式。让我们看看内部可变性有用的情况，并检查它是如何可能的。

<!-- Old headings. Do not remove or links may break. -->

<a id="interior-mutability-a-mutable-borrow-to-an-immutable-value"></a>

### 使用内部可变性

借用规则的一个结果是，当你有一个不可变值时，你不能可变地借用它。例如，此代码不会编译：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/src/main.rs}}
```

如果你尝试编译此代码，你将得到以下错误：

```console
{{#include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/output.txt}}
```

但是，在某些情况下，值在其方法中改变自身但对外部代码显得不可变是有用的。值方法外部的代码将无法改变值。使用 `RefCell<T>` 是获得内部可变性能力的一种方式，但 `RefCell<T>` 不能完全绕过借用规则：编译器中的借用检查器允许此内部可变性，并且借用规则在运行时而不是编译时检查。如果你违反规则，你将得到 `panic!` 而不是编译器错误。

让我们通过一个实际示例，其中我们可以使用 `RefCell<T>` 来改变不可变值，并看看为什么这很有用。

<!-- Old headings. Do not remove or links may break. -->

<a id="a-use-case-for-interior-mutability-mock-objects"></a>

#### 使用 Mock 对象进行测试

有时在测试期间，程序员会使用一个类型代替另一个类型，以便观察特定行为并断言它已正确实现。此占位符类型称为 _测试替身_。想想电影制作中的特技替身，一个人介入并替代演员来做一个特别棘手的场景。测试替身在我们运行测试时代表其他类型。_Mock 对象_ 是特定类型的测试替身，它们记录测试期间发生的情况，以便你可以断言发生了正确的操作。

Rust 没有与其他语言具有对象相同意义上的对象，Rust 也没有像其他一些语言那样内置到标准库中的 mock 对象功能。但是，你绝对可以创建一个结构体，它将服务于与 mock 对象相同的目的。

这是我们将测试的场景：我们将创建一个库，它跟踪一个值相对于最大值，并根据当前值接近最大值的程度发送消息。例如，此库可用于跟踪用户允许进行的 API 调用数量的配额。

我们的库将只提供跟踪值接近最大值的程度以及应该在什么时间发送什么消息的功能。使用我们库的应用程序将需要提供发送消息的机制：应用程序可以直接向用户显示消息、发送电子邮件、发送文本消息或执行其他操作。库不需要知道该细节。它只需要实现我们将提供的 trait 的东西，称为 `Messenger`。代码清单 15-20 显示了库代码。

<Listing number="15-20" file-name="src/lib.rs" caption="一个库，用于跟踪值接近最大值的程度，并在值处于某些级别时发出警告">

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-20/src/lib.rs}}
```

</Listing>

此代码的一个重要部分是 `Messenger` trait 有一个名为 `send` 的方法，它接受对 `self` 的不可变引用和消息的文本。此 trait 是我们的 mock 对象需要实现的接口，以便 mock 可以以与真实对象相同的方式使用。另一个重要部分是，我们想要测试 `LimitTracker` 上的 `set_value` 方法的行为。我们可以更改我们为 `value` 参数传递的内容，但 `set_value` 不会返回任何东西供我们进行断言。我们希望能够说，如果我们使用实现 `Messenger` trait 的东西和 `max` 的特定值创建 `LimitTracker`，当我们为 `value` 传递不同的数字时，messenger 被告知发送适当的消息。

我们需要一个 mock 对象，它在我们调用 `send` 时不是发送电子邮件或文本消息，而是只跟踪它被告知要发送的消息。我们可以创建 mock 对象的新实例，创建使用 mock 对象的 `LimitTracker`，在 `LimitTracker` 上调用 `set_value` 方法，然后检查 mock 对象具有我们期望的消息。代码清单 15-21 显示尝试实现 mock 对象来做到这一点，但借用检查器不允许它。

<Listing number="15-21" file-name="src/lib.rs" caption="尝试实现 `MockMessenger`，但借用检查器不允许">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-21/src/lib.rs:here}}
```

</Listing>

此测试代码定义了一个 `MockMessenger` 结构体，它有一个 `sent_messages` 字段，带有 `String` 值的 `Vec` 来跟踪它被告知要发送的消息。我们还定义了一个关联函数 `new`，以便于创建以空消息列表开始的新 `MockMessenger` 值。然后，我们为 `MockMessenger` 实现 `Messenger` trait，以便我们可以将 `MockMessenger` 提供给 `LimitTracker`。在 `send` 方法的定义中，我们将作为参数传递的消息存储在 `MockMessenger` 的 `sent_messages` 列表中。

在测试中，我们测试当 `LimitTracker` 被告知将 `value` 设置为超过 `max` 值的 75% 时会发生什么。首先，我们创建一个新的 `MockMessenger`，它将以空消息列表开始。然后，我们创建一个新的 `LimitTracker`，并给它新 `MockMessenger` 的引用和 `max` 值 `100`。我们在 `LimitTracker` 上使用值 `80` 调用 `set_value` 方法，这超过 100 的 75%。然后，我们断言 `MockMessenger` 跟踪的消息列表现在应该在其中有一条消息。

但是，此测试有一个问题，如下所示：

```console
{{#include ../listings/ch15-smart-pointers/listing-15-21/output.txt}}
```

我们不能修改 `MockMessenger` 来跟踪消息，因为 `send` 方法接受对 `self` 的不可变引用。我们也不能采用错误文本中的建议，在 `impl` 方法和 trait 定义中都使用 `&mut self`。我们不想仅仅为了测试而更改 `Messenger` trait。相反，我们需要找到一种方法使我们的测试代码与现有设计正确工作。

这是内部可变性可以帮助的情况！我们将在 `RefCell<T>` 内存储 `sent_messages`，然后 `send` 方法将能够修改 `sent_messages` 以存储我们看到的消息。代码清单 15-22 显示了它的样子。

<Listing number="15-22" file-name="src/lib.rs" caption="使用 `RefCell<T>` 在外部值被认为是不可变时改变内部值">

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-22/src/lib.rs:here}}
```

</Listing>

`sent_messages` 字段现在是类型 `RefCell<Vec<String>>` 而不是 `Vec<String>`。在 `new` 函数中，我们围绕空向量创建一个新的 `RefCell<Vec<String>>` 实例。

对于 `send` 方法的实现，第一个参数仍然是对 `self` 的不可变借用，这与 trait 定义匹配。我们在 `self.sent_messages` 中的 `RefCell<Vec<String>>` 上调用 `borrow_mut`，以获取对 `RefCell<Vec<String>>` 内部值的可变引用，这是向量。然后，我们可以对向量的可变引用调用 `push`，以跟踪测试期间发送的消息。

我们必须进行的最后一个更改是在断言中：要查看内部向量中有多少项，我们在 `RefCell<Vec<String>>` 上调用 `borrow`，以获取对向量的不可变引用。

现在你已经看到了如何使用 `RefCell<T>`，让我们深入了解它是如何工作的！

<!-- Old headings. Do not remove or links may break. -->

<a id="keeping-track-of-borrows-at-runtime-with-refcellt"></a>

#### 在运行时跟踪借用

创建不可变和可变引用时，我们分别使用 `&` 和 `&mut` 语法。对于 `RefCell<T>`，我们使用 `borrow` 和 `borrow_mut` 方法，它们属于 `RefCell<T>` 的安全 API 的一部分。`borrow` 方法返回智能指针类型 `Ref<T>`，`borrow_mut` 返回智能指针类型 `RefMut<T>`。两种类型都实现 `Deref`，所以我们可以像常规引用一样对待它们。

`RefCell<T>` 跟踪当前有多少 `Ref<T>` 和 `RefMut<T>` 智能指针处于活动状态。每次我们调用 `borrow`，`RefCell<T>` 增加其活动不可变借用的计数。当 `Ref<T>` 值超出作用域时，不可变借用的计数减少 1。就像编译时借用规则一样，`RefCell<T>` 让我们在任何时间点都有许多不可变借用或一个可变借用。

如果我们尝试违反这些规则，而不是像使用引用那样得到编译器错误，`RefCell<T>` 的实现将在运行时 panic。代码清单 15-23 显示了对代码清单 15-22 中 `send` 实现的修改。我们故意尝试为同一作用域创建两个活动可变借用，以说明 `RefCell<T>` 阻止我们在运行时这样做。

<Listing number="15-23" file-name="src/lib.rs" caption="在同一作用域中创建两个可变引用，以查看 `RefCell<T>` 将 panic">

```rust,ignore,panics
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-23/src/lib.rs:here}}
```

</Listing>

我们为从 `borrow_mut` 返回的 `RefMut<T>` 智能指针创建变量 `one_borrow`。然后，我们在变量 `two_borrow` 中以相同方式创建另一个可变借用。这使得同一作用域中有两个可变引用，这是不允许的。当我们为库运行测试时，代码清单 15-23 中的代码将在没有任何错误的情况下编译，但测试将失败：

```console
{{#include ../listings/ch15-smart-pointers/listing-15-23/output.txt}}
```

注意，代码 panic 并显示消息 `already borrowed: BorrowMutError`。这是 `RefCell<T>` 在运行时处理违反借用规则的方式。

选择在运行时而不是编译时捕获借用错误，正如我们在这里所做的，意味着你可能在开发过程的后期发现代码中的错误：可能直到你的代码部署到生产环境。此外，你的代码将因为跟踪运行时而不是编译时的借用而遭受小的运行时性能损失。但是，使用 `RefCell<T>` 使得编写可以修改自身以跟踪它已看到的消息的 mock 对象成为可能，而你在只允许不可变值的上下文中使用它。尽管有这些权衡，你可以使用 `RefCell<T>` 来获得比常规引用提供的更多功能。

<!-- Old headings. Do not remove or links may break. -->

<a id="having-multiple-owners-of-mutable-data-by-combining-rc-t-and-ref-cell-t"></a>
<a id="allowing-multiple-owners-of-mutable-data-with-rct-and-refcellt"></a>

### 允许可变数据的多个所有者

使用 `RefCell<T>` 的常见方式是与 `Rc<T>` 结合使用。回想一下，`Rc<T>` 让你拥有某些数据的多个所有者，但它只提供对该数据的不可变访问。如果你有一个持有 `RefCell<T>` 的 `Rc<T>`，你可以获得一个可以具有多个所有者 _并且_ 你可以改变的值！

例如，回想代码清单 15-18 中的 cons 列表示例，我们使用 `Rc<T>` 允许多个列表共享另一个列表的所有权。因为 `Rc<T>` 只保存不可变值，一旦我们创建了它们，我们就不能更改列表中的任何值。让我们添加 `RefCell<T>` 以改变列表中的值。代码清单 15-24 显示，通过在 `Cons` 定义中使用 `RefCell<T>`，我们可以修改存储在所有列表中的值。

<Listing number="15-24" file-name="src/main.rs" caption="使用 `Rc<RefCell<i32>>` 创建可以改变的 `List`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-24/src/main.rs}}
```

</Listing>

我们创建一个值是 `Rc<RefCell<i32>>` 实例的值，并将其存储在名为 `value` 的变量中，以便我们稍后可以直接访问它。然后，我们在 `a` 中创建一个 `List`，其中包含持有 `value` 的 `Cons` 变体。我们需要克隆 `value`，以便 `a` 和 `value` 都拥有内部 `5` 值的所有权，而不是将所有权从 `value` 转移到 `a` 或让 `a` 从 `value` 借用。

我们将列表 `a` 包装在 `Rc<T>` 中，以便当我们创建列表 `b` 和 `c` 时，它们都可以引用 `a`，这就是我们在代码清单 15-18 中所做的。

在我们创建 `a`、`b` 和 `c` 中的列表后，我们想要在 `value` 中的值上加 10。我们通过在 `value` 上调用 `borrow_mut` 来做到这一点，它使用我们在第 5 章的["`->` 操作符在哪里？"][wheres-the---operator]<!-- ignore -->中讨论的自动解引用功能，将 `Rc<T>` 解引用到内部 `RefCell<T>` 值。`borrow_mut` 方法返回 `RefMut<T>` 智能指针，我们在其上使用解引用操作符并更改内部值。

当我们打印 `a`、`b` 和 `c` 时，我们可以看到它们都有修改后的值 `15` 而不是 `5`：

```console
{{#include ../listings/ch15-smart-pointers/listing-15-24/output.txt}}
```

此技术非常整洁！通过使用 `RefCell<T>`，我们有一个外表不可变的 `List` 值。但我们可以使用 `RefCell<T>` 上的方法，这些方法提供对其内部可变性的访问，以便我们可以在需要时修改数据。借用规则的运行时检查保护我们免受数据竞争，有时值得在我们的数据结构中为了这种灵活性而牺牲一点速度。注意，`RefCell<T>` 不适用于多线程代码！`Mutex<T>` 是 `RefCell<T>` 的线程安全版本，我们将在第 16 章讨论 `Mutex<T>`。

[wheres-the---operator]: ch05-03-method-syntax.html#wheres-the---operator
