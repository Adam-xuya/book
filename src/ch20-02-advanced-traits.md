## 高级 Traits

我们首先在第10章的[“使用 Traits 定义共享行为”][traits]<!-- ignore -->部分中介绍了 traits，但我们没有讨论更高级的细节。现在你对 Rust 有了更多了解，我们可以深入了解细节。

<!-- Old headings. Do not remove or links may break. -->

<a id="specifying-placeholder-types-in-trait-definitions-with-associated-types"></a>
<a id="associated-types"></a>

### 使用关联类型定义 Traits

_关联类型_将类型占位符与 trait 连接起来，这样 trait 方法定义可以在其签名中使用这些占位符类型。trait 的实现者将指定要使用的具体类型，而不是占位符类型，用于特定实现。这样，我们可以定义一个使用某些类型的 trait，而不需要确切知道这些类型是什么，直到 trait 被实现。

我们已经将本章中的大多数高级特性描述为很少需要。关联类型介于中间：它们比本书其余部分解释的特性使用得更少，但比本章讨论的许多其他特性使用得更频繁。

具有关联类型的 trait 的一个示例是标准库提供的 `Iterator` trait。关联类型名为 `Item`，代表实现 `Iterator` trait 的类型正在迭代的值的类型。`Iterator` trait 的定义如代码清单20-13所示。

<Listing number="20-13" caption="具有关联类型 `Item` 的 `Iterator` trait 的定义">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-13/src/lib.rs}}
```

</Listing>

类型 `Item` 是一个占位符，`next` 方法的定义显示它将返回类型 `Option<Self::Item>` 的值。`Iterator` trait 的实现者将指定 `Item` 的具体类型，`next` 方法将返回包含该具体类型值的 `Option`。

关联类型可能看起来与泛型类似，因为后者允许我们定义函数而不指定它可以处理哪些类型。为了检查这两个概念之间的差异，我们将查看在名为 `Counter` 的类型上实现 `Iterator` trait 的实现，该实现指定 `Item` 类型为 `u32`：

<Listing file-name="src/lib.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-22-iterator-on-counter/src/lib.rs:ch19}}
```

</Listing>

这种语法似乎与泛型相当。那么，为什么不只用泛型定义 `Iterator` trait，如代码清单20-14所示？

<Listing number="20-14" caption="使用泛型的 `Iterator` trait 的假设定义">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-14/src/lib.rs}}
```

</Listing>

区别在于，当使用泛型时，如代码清单20-14所示，我们必须在每个实现中注释类型；因为我们也可以为 `Counter` 实现 `Iterator<String>` 或任何其他类型，我们可以为 `Counter` 有多个 `Iterator` 实现。换句话说，当 trait 具有泛型参数时，它可以为类型多次实现，每次都更改泛型类型参数的具体类型。当我们在 `Counter` 上使用 `next` 方法时，我们必须提供类型注释以指示我们要使用哪个 `Iterator` 实现。

使用关联类型时，我们不需要注释类型，因为我们不能多次为类型实现 trait。在代码清单20-13中使用关联类型的定义中，我们只能选择一次 `Item` 的类型，因为只能有一个 `impl Iterator for Counter`。我们不必在每次调用 `Counter` 上的 `next` 时指定我们想要 `u32` 值的迭代器。

关联类型也成为 trait 契约的一部分：trait 的实现者必须提供类型来替代关联类型占位符。关联类型通常有一个描述类型将如何使用的名称，在 API 文档中记录关联类型是一个好习惯。

<!-- Old headings. Do not remove or links may break. -->

<a id="default-generic-type-parameters-and-operator-overloading"></a>

### 使用默认泛型参数和运算符重载

当我们使用泛型类型参数时，我们可以为泛型类型指定默认具体类型。如果默认类型有效，这消除了 trait 实现者指定具体类型的需要。你在使用 `<PlaceholderType=ConcreteType>` 语法声明泛型类型时指定默认类型。

这种技术有用的一个很好的例子是_运算符重载_，在这种情况下，你在特定情况下自定义运算符（如 `+`）的行为。

Rust 不允许你创建自己的运算符或重载任意运算符。但是，你可以通过实现与运算符关联的 traits 来重载 `std::ops` 中列出的操作和相应的 traits。例如，在代码清单20-15中，我们重载 `+` 运算符以将两个 `Point` 实例相加。我们通过在 `Point` 结构体上实现 `Add` trait 来做到这一点。

<Listing number="20-15" file-name="src/main.rs" caption="实现 `Add` trait 以重载 `Point` 实例的 `+` 运算符">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-15/src/main.rs}}
```

</Listing>

`add` 方法将两个 `Point` 实例的 `x` 值和两个 `Point` 实例的 `y` 值相加以创建新的 `Point`。`Add` trait 有一个名为 `Output` 的关联类型，它确定从 `add` 方法返回的类型。

此代码中的默认泛型类型在 `Add` trait 内。这是它的定义：

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

此代码应该看起来很熟悉：一个具有一个方法和一个关联类型的 trait。新部分是 `Rhs=Self`：此语法称为_默认类型参数_。`Rhs` 泛型类型参数（“右手边”的缩写）定义 `add` 方法中 `rhs` 参数的类型。如果我们在实现 `Add` trait 时不指定 `Rhs` 的具体类型，`Rhs` 的类型将默认为 `Self`，这将是我们在其上实现 `Add` 的类型。

当我们为 `Point` 实现 `Add` 时，我们使用 `Rhs` 的默认值，因为我们想要将两个 `Point` 实例相加。让我们看一个实现 `Add` trait 的示例，我们希望自定义 `Rhs` 类型而不是使用默认值。

我们有两个结构体，`Millimeters` 和 `Meters`，以不同单位保存值。这种将现有类型包装在另一个结构体中的薄包装称为_新类型模式_，我们在[“使用新类型模式实现外部 Traits”][newtype]<!-- ignore -->部分中更详细地描述。我们希望将毫米值添加到米值，并让 `Add` 的实现正确进行转换。我们可以为 `Millimeters` 实现 `Add`，将 `Meters` 作为 `Rhs`，如代码清单20-16所示。

<Listing number="20-16" file-name="src/lib.rs" caption="在 `Millimeters` 上实现 `Add` trait 以添加 `Millimeters` 和 `Meters`">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-16/src/lib.rs}}
```

</Listing>

要添加 `Millimeters` 和 `Meters`，我们指定 `impl Add<Meters>` 来设置 `Rhs` 类型参数的值，而不是使用 `Self` 的默认值。

你将以两种主要方式使用默认类型参数：

1. 扩展类型而不破坏现有代码
2. 允许在大多数用户不需要的特定情况下进行自定义

标准库的 `Add` trait 是第二个目的的示例：通常，你会添加两个相似类型，但 `Add` trait 提供了超越该能力进行自定义的能力。在 `Add` trait 定义中使用默认类型参数意味着你不需要在大多数时候指定额外参数。换句话说，不需要一些实现样板，使 trait 更容易使用。

第一个目的与第二个目的类似，但相反：如果你想向现有 trait 添加类型参数，你可以给它一个默认值，以允许扩展 trait 的功能而不破坏现有实现代码。

<!-- Old headings. Do not remove or links may break. -->

<a id="fully-qualified-syntax-for-disambiguation-calling-methods-with-the-same-name"></a>
<a id="disambiguating-between-methods-with-the-same-name"></a>

### 消除同名方法之间的歧义

Rust 中没有任何东西阻止 trait 具有与另一个 trait 的方法同名的方法，Rust 也不阻止你在一个类型上实现两个 traits。也可以在类型上直接实现一个与来自 traits 的方法同名的方法。

当调用同名方法时，你需要告诉 Rust 你想要使用哪一个。考虑代码清单20-17中的代码，我们定义了两个 traits，`Pilot` 和 `Wizard`，它们都有一个名为 `fly` 的方法。然后我们在类型 `Human` 上实现这两个 traits，该类型已经有一个名为 `fly` 的方法实现。每个 `fly` 方法都做一些不同的事情。

<Listing number="20-17" file-name="src/main.rs" caption="定义两个具有 `fly` 方法的 traits，并在 `Human` 类型上实现它们，并在 `Human` 上直接实现 `fly` 方法。">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-17/src/main.rs:here}}
```

</Listing>

当我们在 `Human` 的实例上调用 `fly` 时，编译器默认调用直接在类型上实现的方法，如代码清单20-18所示。

<Listing number="20-18" file-name="src/main.rs" caption="在 `Human` 的实例上调用 `fly`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-18/src/main.rs:here}}
```

</Listing>

运行此代码将打印 `*waving arms furiously*`，显示 Rust 调用了直接在 `Human` 上实现的 `fly` 方法。

要调用来自 `Pilot` trait 或 `Wizard` trait 的 `fly` 方法，我们需要使用更明确的语法来指定我们指的是哪个 `fly` 方法。代码清单20-19演示了此语法。

<Listing number="20-19" file-name="src/main.rs" caption="指定我们想要调用的 trait 的 `fly` 方法">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-19/src/main.rs:here}}
```

</Listing>

在方法名称之前指定 trait 名称向 Rust 澄清了我们想要调用哪个 `fly` 实现。我们也可以编写 `Human::fly(&person)`，这等价于我们在代码清单20-19中使用的 `person.fly()`，但如果我们不需要消除歧义，这会有点长。

运行此代码会打印以下内容：

```console
{{#include ../listings/ch20-advanced-features/listing-20-19/output.txt}}
```

因为 `fly` 方法接受 `self` 参数，如果我们有两个_类型_都实现一个_trait_，Rust 可以基于 `self` 的类型确定要使用哪个 trait 实现。

但是，不是方法的关联函数没有 `self` 参数。当有多个类型或 traits 定义具有相同函数名的非方法函数时，Rust 并不总是知道你的意思，除非你使用完全限定语法。例如，在代码清单20-20中，我们为希望将所有小狗命名为 Spot 的动物收容所创建一个 trait。我们创建一个具有关联非方法函数 `baby_name` 的 `Animal` trait。`Animal` trait 为结构体 `Dog` 实现，我们还在其上直接提供关联非方法函数 `baby_name`。

<Listing number="20-20" file-name="src/main.rs" caption="具有关联函数的 trait 和具有同名关联函数的类型，该类型也实现该 trait">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-20/src/main.rs}}
```

</Listing>

我们在 `Dog` 上定义的 `baby_name` 关联函数中实现将所有小狗命名为 Spot 的代码。`Dog` 类型也实现 trait `Animal`，它描述了所有动物具有的特征。小狗被称为 puppy，这在 `Dog` 上的 `Animal` trait 实现中与 `Animal` trait 关联的 `baby_name` 函数中表达。

在 `main` 中，我们调用 `Dog::baby_name` 函数，它直接调用在 `Dog` 上定义的关联函数。此代码打印以下内容：

```console
{{#include ../listings/ch20-advanced-features/listing-20-20/output.txt}}
```

此输出不是我们想要的。我们想要调用 `Animal` trait 的一部分的 `baby_name` 函数，我们在 `Dog` 上实现了它，以便代码打印 `A baby dog is called a puppy`。我们在代码清单20-19中使用的指定 trait 名称的技术在这里没有帮助；如果我们将 `main` 更改为代码清单20-21中的代码，我们将得到编译错误。

<Listing number="20-21" file-name="src/main.rs" caption="尝试从 `Animal` trait 调用 `baby_name` 函数，但 Rust 不知道要使用哪个实现">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-21/src/main.rs:here}}
```

</Listing>

因为 `Animal::baby_name` 没有 `self` 参数，并且可能有其他类型实现 `Animal` trait，Rust 无法确定我们想要哪个 `Animal::baby_name` 实现。我们将收到此编译器错误：

```console
{{#include ../listings/ch20-advanced-features/listing-20-21/output.txt}}
```

为了消除歧义并告诉 Rust 我们想要使用 `Dog` 的 `Animal` 实现，而不是某些其他类型的 `Animal` 实现，我们需要使用完全限定语法。代码清单20-22演示了如何使用完全限定语法。

<Listing number="20-22" file-name="src/main.rs" caption="使用完全限定语法指定我们想要调用在 `Dog` 上实现的 `Animal` trait 的 `baby_name` 函数">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-22/src/main.rs:here}}
```

</Listing>

我们在尖括号内为 Rust 提供类型注释，这表明我们想要从 `Animal` trait 调用 `baby_name` 方法，通过在 `Dog` 上实现，通过说我们希望将 `Dog` 类型视为此函数调用的 `Animal`。此代码现在将打印我们想要的内容：

```console
{{#include ../listings/ch20-advanced-features/listing-20-22/output.txt}}
```

一般来说，完全限定语法定义如下：

```rust,ignore
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

对于不是方法的关联函数，不会有 `receiver`：只有其他参数的列表。你可以在调用函数或方法的任何地方使用完全限定语法。但是，允许你省略 Rust 可以从程序中的其他信息推断出的此语法的任何部分。你只需要在存在多个使用相同名称的实现且 Rust 需要帮助来识别你想要调用哪个实现的情况下使用此更详细的语法。

<!-- Old headings. Do not remove or links may break. -->

<a id="using-supertraits-to-require-one-traits-functionality-within-another-trait"></a>

### 使用超 Traits

有时你可能编写一个依赖于另一个 trait 的 trait 定义：对于实现第一个 trait 的类型，你希望要求该类型也实现第二个 trait。你这样做是为了让你的 trait 定义可以利用第二个 trait 的关联项。你的 trait 定义所依赖的 trait 称为你的 trait 的_超 trait_。

例如，假设我们想要创建一个具有 `outline_print` 方法的 `OutlinePrint` trait，该方法将打印格式化后的给定值，使其用星号框起来。也就是说，给定一个实现标准库 trait `Display` 以产生 `(x, y)` 的 `Point` 结构体，当我们在 `x` 为 `1`，`y` 为 `3` 的 `Point` 实例上调用 `outline_print` 时，它应该打印以下内容：

```text
**********
*        *
* (1, 3) *
*        *
**********
```

在 `outline_print` 方法的实现中，我们希望使用 `Display` trait 的功能。因此，我们需要指定 `OutlinePrint` trait 仅适用于也实现 `Display` 并提供 `OutlinePrint` 所需功能的类型。我们可以在 trait 定义中通过指定 `OutlinePrint: Display` 来做到这一点。这种技术类似于向 trait 添加 trait 约束。代码清单20-23显示了 `OutlinePrint` trait 的实现。

<Listing number="20-23" file-name="src/main.rs" caption="实现需要 `Display` 功能的 `OutlinePrint` trait">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-23/src/main.rs:here}}
```

</Listing>

因为我们指定了 `OutlinePrint` 需要 `Display` trait，我们可以使用 `to_string` 函数，该函数自动为任何实现 `Display` 的类型实现。如果我们尝试在不添加冒号和指定 trait 名称后的 `Display` trait 的情况下使用 `to_string`，我们会收到错误，说在当前作用域中没有为类型 `&Self` 找到名为 `to_string` 的方法。

让我们看看当我们尝试在不实现 `Display` 的类型（如 `Point` 结构体）上实现 `OutlinePrint` 时会发生什么：

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/src/main.rs:here}}
```

</Listing>

我们收到错误，说 `Display` 是必需的但未实现：

```console
{{#include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/output.txt}}
```

要修复此问题，我们在 `Point` 上实现 `Display` 并满足 `OutlinePrint` 要求的约束，如下所示：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-03-impl-display-for-point/src/main.rs:here}}
```

</Listing>

然后，在 `Point` 上实现 `OutlinePrint` trait 将成功编译，我们可以在 `Point` 实例上调用 `outline_print` 以在星号轮廓中显示它。

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-newtype-pattern-to-implement-external-traits-on-external-types"></a>
<a id="using-the-newtype-pattern-to-implement-external-traits"></a>

### 使用新类型模式实现外部 Traits

在第10章的[“在类型上实现 Trait”][implementing-a-trait-on-a-type]<!-- ignore -->部分中，我们提到了孤儿规则，该规则规定只有当 trait 或类型，或两者都是我们 crate 的本地时，我们才被允许在类型上实现 trait。可以使用新类型模式来绕过此限制，这涉及在元组结构体中创建新类型。（我们在第5章的[“使用元组结构体创建不同类型”][tuple-structs]<!-- ignore -->部分中介绍了元组结构体。）元组结构体将有一个字段，并且是我们想要实现 trait 的类型的薄包装。然后，包装类型是我们 crate 的本地，我们可以在包装器上实现 trait。_Newtype_是一个源自 Haskell 编程语言的术语。使用此模式没有运行时性能损失，包装类型在编译时被省略。

例如，假设我们想在 `Vec<T>` 上实现 `Display`，孤儿规则阻止我们直接这样做，因为 `Display` trait 和 `Vec<T>` 类型都在我们的 crate 外部定义。我们可以创建一个 `Wrapper` 结构体，它保存 `Vec<T>` 的实例；然后，我们可以在 `Wrapper` 上实现 `Display` 并使用 `Vec<T>` 值，如代码清单20-24所示。

<Listing number="20-24" file-name="src/main.rs" caption="围绕 `Vec<String>` 创建 `Wrapper` 类型以实现 `Display`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-24/src/main.rs}}
```

</Listing>

`Display` 的实现使用 `self.0` 访问内部 `Vec<T>`，因为 `Wrapper` 是元组结构体，`Vec<T>` 是元组中索引 0 处的项。然后，我们可以在 `Wrapper` 上使用 `Display` trait 的功能。

使用此技术的缺点是 `Wrapper` 是新类型，所以它没有它持有的值的方法。我们必须直接在 `Wrapper` 上实现 `Vec<T>` 的所有方法，以便方法委托给 `self.0`，这将允许我们像对待 `Vec<T>` 一样对待 `Wrapper`。如果我们希望新类型具有内部类型具有的每个方法，在 `Wrapper` 上实现 `Deref` trait 以返回内部类型将是一个解决方案（我们在第15章的[“将智能指针视为常规引用”][smart-pointer-deref]<!-- ignore -->部分中讨论了实现 `Deref` trait）。如果我们不希望 `Wrapper` 类型具有内部类型的所有方法——例如，限制 `Wrapper` 类型的行为——我们必须手动实现我们确实想要的方法。

即使不涉及 traits，这种新类型模式也很有用。让我们切换焦点，看看与 Rust 类型系统交互的一些高级方法。

[newtype]: ch20-02-advanced-traits.html#implementing-external-traits-with-the-newtype-pattern
[implementing-a-trait-on-a-type]: ch10-02-traits.html#implementing-a-trait-on-a-type
[traits]: ch10-02-traits.html
[smart-pointer-deref]: ch15-02-deref.html#treating-smart-pointers-like-regular-references
[tuple-structs]: ch05-01-defining-structs.html#creating-different-types-with-tuple-structs
