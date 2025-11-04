<!-- Old headings. Do not remove or links may break. -->

<a id="using-trait-objects-that-allow-for-values-of-different-types"></a>

## 使用 Trait 对象来抽象共享行为

在第8章中，我们提到向量的一个限制是它们只能存储一种类型的元素。我们在代码清单8-9中创建了一个变通方案，其中我们定义了一个 `SpreadsheetCell` 枚举，该枚举具有保存整数、浮点数和文本的变体。这意味着我们可以在每个单元格中存储不同类型的数据，并且仍然有一个表示单元格行的向量。当我们的可互换项目是我们在编译代码时知道的固定类型集时，这是一个完美的解决方案。

但是，有时我们希望库用户能够扩展在特定情况下有效的类型集。为了展示我们如何实现这一点，我们将创建一个示例图形用户界面（GUI）工具，它迭代项目列表，在每个项目上调用 `draw` 方法将其绘制到屏幕上——这是 GUI 工具的常用技术。我们将创建一个名为 `gui` 的库 crate，其中包含 GUI 库的结构。这个 crate 可能包括一些供人们使用的类型，例如 `Button` 或 `TextField`。此外，`gui` 用户会希望创建自己的可以绘制的类型：例如，一个程序员可能添加一个 `Image`，另一个可能添加一个 `SelectBox`。

在编写库时，我们无法知道和定义其他程序员可能想要创建的所有类型。但我们确实知道 `gui` 需要跟踪许多不同类型的值，并且它需要在每个这些不同类型值上调用 `draw` 方法。它不需要确切知道当我们调用 `draw` 方法时会发生什么，只需要值将具有该方法供我们调用。

在具有继承的语言中，我们可能会定义一个名为 `Component` 的类，该类具有一个名为 `draw` 的方法。其他类，如 `Button`、`Image` 和 `SelectBox`，将从 `Component` 继承，从而继承 `draw` 方法。它们可以各自覆盖 `draw` 方法以定义其自定义行为，但框架可以将所有类型视为 `Component` 实例并在它们上调用 `draw`。但是因为 Rust 没有继承，我们需要另一种方式来构建 `gui` 库以允许用户创建与库兼容的新类型。

### 定义 Trait 以表示共同行为

为了实现我们希望 `gui` 具有的行为，我们将定义一个名为 `Draw` 的 trait，它将有一个名为 `draw` 的方法。然后，我们可以定义一个接受 trait 对象的向量。_Trait 对象_指向实现我们指定 trait 的类型的实例以及用于在运行时在该类型上查找 trait 方法的表。我们通过指定某种指针（例如引用或 `Box<T>` 智能指针），然后 `dyn` 关键字，然后指定相关 trait 来创建 trait 对象。（我们将在第20章的[“动态大小类型和 `Sized` Trait”][dynamically-sized]<!-- ignore -->中讨论 trait 对象必须使用指针的原因。）我们可以使用 trait 对象代替泛型或具体类型。无论我们在哪里使用 trait 对象，Rust 的类型系统都会在编译时确保在该上下文中使用的任何值都将实现 trait 对象的 trait。因此，我们不需要在编译时知道所有可能的类型。

我们提到过，在 Rust 中，我们避免将结构体和枚举称为“对象”以将它们与其他语言的对象区分开来。在结构体或枚举中，结构体字段中的数据和在 `impl` 块中的行为是分开的，而在其他语言中，组合成一种概念的数据和行为通常被标记为对象。Trait 对象与其他语言中的对象不同，因为我们不能向 trait 对象添加数据。Trait 对象不如其他语言中的对象一般有用：它们的特定目的是允许跨共同行为进行抽象。

代码清单18-3显示了如何定义一个名为 `Draw` 的 trait，它有一个名为 `draw` 的方法。

<Listing number="18-3" file-name="src/lib.rs" caption="`Draw` trait 的定义">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-03/src/lib.rs}}
```

</Listing>

这个语法应该从我们在第10章中关于如何定义 traits 的讨论中看起来很熟悉。接下来是一些新语法：代码清单18-4定义了一个名为 `Screen` 的结构体，它保存一个名为 `components` 的向量。这个向量的类型是 `Box<dyn Draw>`，这是一个 trait 对象；它是实现 `Draw` trait 的 `Box` 内部任何类型的替身。

<Listing number="18-4" file-name="src/lib.rs" caption="`Screen` 结构体的定义，带有一个 `components` 字段，保存实现 `Draw` trait 的 trait 对象向量">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-04/src/lib.rs:here}}
```

</Listing>

在 `Screen` 结构体上，我们将定义一个名为 `run` 的方法，它将在其每个 `components` 上调用 `draw` 方法，如代码清单18-5所示。

<Listing number="18-5" file-name="src/lib.rs" caption="`Screen` 上的 `run` 方法，在每个组件上调用 `draw` 方法">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-05/src/lib.rs:here}}
```

</Listing>

这与定义使用带有 trait 约束的泛型类型参数的结构体不同。泛型类型参数一次只能替换为一种具体类型，而 trait 对象允许在运行时使用多种具体类型来填充 trait 对象。例如，我们可以使用泛型类型和 trait 约束定义 `Screen` 结构体，如代码清单18-6所示。

<Listing number="18-6" file-name="src/lib.rs" caption="`Screen` 结构体及其 `run` 方法的替代实现，使用泛型和 trait 约束">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-06/src/lib.rs:here}}
```

</Listing>

这限制了我们的 `Screen` 实例只能有一个组件列表，所有组件都是 `Button` 类型或所有组件都是 `TextField` 类型。如果你只会有同质集合，使用泛型和 trait 约束更可取，因为定义将在编译时单态化以使用具体类型。

另一方面，使用 trait 对象的方法，一个 `Screen` 实例可以持有一个 `Vec<T>`，它包含 `Box<Button>` 以及 `Box<TextField>`。让我们看看这是如何工作的，然后我们将讨论运行时性能影响。

### 实现 Trait

现在我们将添加一些实现 `Draw` trait 的类型。我们将提供 `Button` 类型。同样，实际实现 GUI 库超出了本书的范围，所以 `draw` 方法在其主体中不会有任何有用的实现。为了想象实现可能是什么样子，`Button` 结构体可能有 `width`、`height` 和 `label` 字段，如代码清单18-7所示。

<Listing number="18-7" file-name="src/lib.rs" caption="实现 `Draw` trait 的 `Button` 结构体">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-07/src/lib.rs:here}}
```

</Listing>

`Button` 上的 `width`、`height` 和 `label` 字段将与其他组件上的字段不同；例如，`TextField` 类型可能具有这些相同的字段加上 `placeholder` 字段。我们想要在屏幕上绘制的每种类型都将实现 `Draw` trait，但会在 `draw` 方法中使用不同的代码来定义如何绘制该特定类型，正如 `Button` 在这里所做的那样（如前所述，没有实际的 GUI 代码）。例如，`Button` 类型可能有一个额外的 `impl` 块，包含与用户单击按钮时发生的情况相关的方法。这些类型的方法不适用于 `TextField` 等类型。

如果使用我们库的人决定实现一个具有 `width`、`height` 和 `options` 字段的 `SelectBox` 结构体，他们也会在 `SelectBox` 类型上实现 `Draw` trait，如代码清单18-8所示。

<Listing number="18-8" file-name="src/main.rs" caption="另一个使用 `gui` 并在 `SelectBox` 结构体上实现 `Draw` trait 的 crate">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-08/src/main.rs:here}}
```

</Listing>

我们库的用户现在可以编写他们的 `main` 函数来创建 `Screen` 实例。对于 `Screen` 实例，他们可以通过将每个放在 `Box<T>` 中来添加 `SelectBox` 和 `Button` 以成为 trait 对象。然后他们可以在 `Screen` 实例上调用 `run` 方法，它将在每个组件上调用 `draw`。代码清单18-9显示了此实现。

<Listing number="18-9" file-name="src/main.rs" caption="使用 trait 对象存储实现相同 trait 的不同类型的值">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-09/src/main.rs:here}}
```

</Listing>

当我们编写库时，我们不知道有人可能会添加 `SelectBox` 类型，但我们的 `Screen` 实现能够对新类型进行操作并绘制它，因为 `SelectBox` 实现了 `Draw` trait，这意味着它实现了 `draw` 方法。

这个概念——只关心值响应的消息而不是值的具体类型——类似于动态类型语言中的_鸭子类型_概念：如果它走起来像鸭子，叫起来像鸭子，那么它一定是鸭子！在代码清单18-5中 `Screen` 上的 `run` 实现中，`run` 不需要知道每个组件的具体类型是什么。它不检查组件是否是 `Button` 或 `SelectBox` 的实例，它只是在组件上调用 `draw` 方法。通过将 `Box<dyn Draw>` 指定为 `components` 向量中值的类型，我们已经定义了 `Screen` 需要我们可以调用 `draw` 方法的值。

使用 trait 对象和 Rust 的类型系统编写类似于使用鸭子类型的代码的优势在于，我们永远不必在运行时检查值是否实现了特定方法，也不必担心如果值没有实现方法但我们仍然调用它时会出错。如果值没有实现 trait 对象需要的 traits，Rust 不会编译我们的代码。

例如，代码清单18-10显示了如果我们尝试创建一个以 `String` 作为组件的 `Screen` 会发生什么。

<Listing number="18-10" file-name="src/main.rs" caption="尝试使用未实现 trait 对象的 trait 的类型">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-10/src/main.rs}}
```

</Listing>

我们会得到这个错误，因为 `String` 没有实现 `Draw` trait：

```console
{{#include ../listings/ch18-oop/listing-18-10/output.txt}}
```

这个错误让我们知道我们正在向 `Screen` 传递我们不想传递的内容，因此应该传递不同的类型，或者我们应该在 `String` 上实现 `Draw`，以便 `Screen` 能够在其上调用 `draw`。

<!-- Old headings. Do not remove or links may break. -->

<a id="trait-objects-perform-dynamic-dispatch"></a>

### 执行动态分发

回想第10章中[“使用泛型的代码性能”][performance-of-code-using-generics]<!-- ignore -->中我们关于编译器对泛型执行的单态化过程的讨论：编译器为我们用来代替泛型类型参数的每个具体类型生成函数和方法的非泛型实现。从单态化产生的代码正在执行_静态分发_，这是编译器在编译时知道你正在调用哪个方法的时候。这与_动态分发_相反，这是编译器在编译时无法判断你正在调用哪个方法的时候。在动态分发情况下，编译器发出代码，该代码在运行时将知道要调用哪个方法。

当我们使用 trait 对象时，Rust 必须使用动态分发。编译器不知道可能与使用 trait 对象的代码一起使用的所有类型，因此它不知道要调用在哪个类型上实现的哪个方法。相反，在运行时，Rust 使用 trait 对象内的指针来知道要调用哪个方法。这种查找会产生运行时成本，这在静态分发中不会发生。动态分发还阻止编译器选择内联方法的代码，这反过来又阻止了一些优化，并且 Rust 有一些关于你可以和不能使用动态分发的规则，称为_dyn 兼容性_。这些规则超出了本讨论的范围，但你可以在[参考文档][dyn-compatibility]<!-- ignore -->中阅读更多关于它们的信息。但是，我们在代码清单18-5中编写的代码确实获得了额外的灵活性，并且能够在代码清单18-9中支持，所以这是一个需要考虑的权衡。

[performance-of-code-using-generics]: ch10-01-syntax.html#performance-of-code-using-generics
[dynamically-sized]: ch20-03-advanced-types.html#dynamically-sized-types-and-the-sized-trait
[dyn-compatibility]: https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility
