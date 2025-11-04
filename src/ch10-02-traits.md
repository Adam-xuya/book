<!-- Old headings. Do not remove or links may break. -->

<a id="traits-defining-shared-behavior"></a>

## 使用 Trait 定义共享行为

_Trait_ 定义特定类型具有并可以与其他类型共享的功能。我们可以使用 trait 以抽象方式定义共享行为。我们可以使用 _trait 约束_ 来指定泛型类型可以是具有某些行为的任何类型。

> 注意：Trait 类似于其他语言中通常称为 _接口_ 的功能，尽管有一些差异。

### 定义 Trait

类型的行为由我们可以在该类型上调用的方法组成。如果我们可以在所有这些类型上调用相同的方法，则不同类型共享相同的行为。Trait 定义是一种将方法签名组合在一起以定义完成某些目的所需的一组行为的方法。

例如，假设我们有多个结构体，它们保存各种类型和数量的文本：一个 `NewsArticle` 结构体，它保存在特定位置归档的新闻故事，以及一个 `SocialPost`，它最多可以有 280 个字符以及指示它是新帖子、转发还是对另一个帖子的回复的元数据。

我们想要创建一个名为 `aggregator` 的媒体聚合器库 crate，它可以显示可能存储在 `NewsArticle` 或 `SocialPost` 实例中的数据摘要。为此，我们需要每个类型的摘要，我们将通过在实例上调用 `summarize` 方法来请求该摘要。代码清单 10-12 显示了一个公共 `Summary` trait 的定义，它表达了这种行为。

<Listing number="10-12" file-name="src/lib.rs" caption="一个 `Summary` trait，由 `summarize` 方法提供的行为组成">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-12/src/lib.rs}}
```

</Listing>

这里，我们使用 `trait` 关键字然后是该 trait 的名称来声明一个 trait，在这种情况下是 `Summary`。我们还将 trait 声明为 `pub`，以便依赖此 crate 的 crate 也可以使用此 trait，我们将在几个示例中看到。在花括号内，我们声明描述实现此 trait 的类型的行为的方法签名，在这种情况下是 `fn summarize(&self) -> String`。

在方法签名之后，我们不在花括号内提供实现，而是使用分号。实现此 trait 的每个类型必须为方法的主体提供自己的自定义行为。编译器将强制任何具有 `Summary` trait 的类型都使用此签名定义 `summarize` 方法。

Trait 可以在其主体中有多个方法：方法签名每行列出一个，每行以分号结尾。

### 在类型上实现 Trait

既然我们已经定义了 `Summary` trait 的方法的所需签名，我们可以在媒体聚合器中的类型上实现它。代码清单 10-13 显示了在 `NewsArticle` 结构体上实现 `Summary` trait，它使用标题、作者和位置来创建 `summarize` 的返回值。对于 `SocialPost` 结构体，我们将 `summarize` 定义为用户名后跟帖子的整个文本，假设帖子内容已经限制为 280 个字符。

<Listing number="10-13" file-name="src/lib.rs" caption="在 `NewsArticle` 和 `SocialPost` 类型上实现 `Summary` trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-13/src/lib.rs:here}}
```

</Listing>

在类型上实现 trait 类似于实现常规方法。区别在于在 `impl` 之后，我们放置我们想要实现的 trait 名称，然后使用 `for` 关键字，然后指定我们想要为其实现 trait 的类型名称。在 `impl` 块内，我们放置 trait 定义定义的方法签名。我们不在每个签名后添加分号，而是使用花括号并用我们希望 trait 的方法对于特定类型具有的特定行为填充方法主体。

既然库已经在 `NewsArticle` 和 `SocialPost` 上实现了 `Summary` trait，crate 的用户可以在 `NewsArticle` 和 `SocialPost` 的实例上以与我们调用常规方法相同的方式调用 trait 方法。唯一的区别是用户必须将 trait 引入作用域以及类型。以下是如何使用我们的 `aggregator` 库 crate 的二进制 crate 的示例：

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-01-calling-trait-method/src/main.rs}}
```

此代码打印 `1 new post: horse_ebooks: of course, as you probably already know, people`。

依赖 `aggregator` crate 的其他 crate 也可以将 `Summary` trait 引入作用域，以便在他们自己的类型上实现 `Summary`。需要注意的一个限制是，只有当 trait 或类型，或两者都是我们 crate 本地的时，我们才能在类型上实现 trait。例如，我们可以在像 `SocialPost` 这样的自定义类型上实现标准库 trait，如 `Display`，作为我们 `aggregator` crate 功能的一部分，因为类型 `SocialPost` 是我们 `aggregator` crate 本地的。我们也可以在我们的 `aggregator` crate 中的 `Vec<T>` 上实现 `Summary`，因为 trait `Summary` 是我们 `aggregator` crate 本地的。

但是我们不能在外部类型上实现外部 trait。例如，我们不能在我们 `aggregator` crate 内的 `Vec<T>` 上实现 `Display` trait，因为 `Display` 和 `Vec<T>` 都在标准库中定义，不是我们 `aggregator` crate 本地的。此限制是称为 _一致性_ 的属性的一部分，更具体地说是 _孤儿规则_，之所以这样命名是因为父类型不存在。此规则确保其他人的代码无法破坏你的代码，反之亦然。如果没有此规则，两个 crate 可以为同一类型实现相同的 trait，而 Rust 不知道使用哪个实现。

<!-- Old headings. Do not remove or links may break. -->

<a id="default-implementations"></a>

### 使用默认实现

有时为 trait 中的某些或所有方法具有默认行为而不是要求每个类型上的所有方法都有实现是有用的。然后，当我们在特定类型上实现 trait 时，我们可以保留或覆盖每个方法的默认行为。

在代码清单 10-14 中，我们为 `Summary` trait 的 `summarize` 方法指定默认字符串，而不是仅定义方法签名，就像我们在代码清单 10-12 中所做的那样。

<Listing number="10-14" file-name="src/lib.rs" caption="定义一个带有 `summarize` 方法默认实现的 `Summary` trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-14/src/lib.rs:here}}
```

</Listing>

要使用默认实现来汇总 `NewsArticle` 的实例，我们使用 `impl Summary for NewsArticle {}` 指定一个空的 `impl` 块。

即使我们不再直接在 `NewsArticle` 上定义 `summarize` 方法，我们也提供了默认实现并指定 `NewsArticle` 实现 `Summary` trait。因此，我们仍然可以在 `NewsArticle` 的实例上调用 `summarize` 方法，如下所示：

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-02-calling-default-impl/src/main.rs:here}}
```

此代码打印 `New article available! (Read more...)`。

创建默认实现不需要我们更改代码清单 10-13 中 `SocialPost` 上 `Summary` 实现的任何内容。原因是覆盖默认实现的语法与实现没有默认实现的 trait 方法的语法相同。

默认实现可以调用同一 trait 中的其他方法，即使这些其他方法没有默认实现。通过这种方式，trait 可以提供大量有用的功能，只需要实现者指定其中的一小部分。例如，我们可以定义 `Summary` trait 具有一个 `summarize_author` 方法，其实现是必需的，然后定义一个具有默认实现的 `summarize` 方法，该方法调用 `summarize_author` 方法：

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:here}}
```

要使用此版本的 `Summary`，我们只需要在类型上实现 trait 时定义 `summarize_author`：

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:impl}}
```

在我们定义 `summarize_author` 之后，我们可以在 `SocialPost` 结构体的实例上调用 `summarize`，`summarize` 的默认实现将调用我们提供的 `summarize_author` 定义。因为我们已经实现了 `summarize_author`，`Summary` trait 为我们提供了 `summarize` 方法的行为，而无需我们编写任何更多代码。这就是它的样子：

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/main.rs:here}}
```

此代码打印 `1 new post: (Read more from @horse_ebooks...)`。

注意，不可能从该方法的覆盖实现中调用默认实现。

<!-- Old headings. Do not remove or links may break. -->

<a id="traits-as-parameters"></a>

### 使用 Trait 作为参数

既然你知道如何定义和实现 trait，我们可以探索如何使用 trait 来定义接受许多不同类型的函数。我们将使用我们在代码清单 10-13 中在 `NewsArticle` 和 `SocialPost` 类型上实现的 `Summary` trait 来定义一个 `notify` 函数，该函数在其 `item` 参数上调用 `summarize` 方法，该参数是某种实现 `Summary` trait 的类型。为此，我们使用 `impl Trait` 语法，如下所示：

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-04-traits-as-parameters/src/lib.rs:here}}
```

我们不是为 `item` 参数指定具体类型，而是指定 `impl` 关键字和 trait 名称。此参数接受实现指定 trait 的任何类型。在 `notify` 的主体中，我们可以在 `item` 上调用来自 `Summary` trait 的任何方法，例如 `summarize`。我们可以调用 `notify` 并传入 `NewsArticle` 或 `SocialPost` 的任何实例。使用任何其他类型（如 `String` 或 `i32`）调用函数的代码将无法编译，因为那些类型不实现 `Summary`。

<!-- Old headings. Do not remove or links may break. -->

<a id="fixing-the-largest-function-with-trait-bounds"></a>

#### Trait 约束语法

`impl Trait` 语法适用于简单的情况，但实际上是称为 _trait 约束_ 的较长形式的语法糖；它看起来像这样：

```rust,ignore
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

这种较长形式等效于上一节中的示例，但更冗长。我们将 trait 约束与泛型类型参数的声明一起放在冒号之后和尖括号内。

`impl Trait` 语法很方便，在简单情况下使代码更简洁，而完整的 trait 约束语法可以在其他情况下表达更复杂的复杂性。例如，我们可以有两个实现 `Summary` 的参数。使用 `impl Trait` 语法这样做如下所示：

```rust,ignore
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
```

如果我们希望此函数允许 `item1` 和 `item2` 具有不同的类型（只要两种类型都实现 `Summary`），使用 `impl Trait` 是合适的。但是，如果我们想强制两个参数具有相同的类型，我们必须使用 trait 约束，如下所示：

```rust,ignore
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

指定为 `item1` 和 `item2` 参数类型的泛型类型 `T` 限制了函数，使得作为 `item1` 和 `item2` 的参数传递的值的具体类型必须相同。

<!-- Old headings. Do not remove or links may break. -->

<a id="specifying-multiple-trait-bounds-with-the--syntax"></a>

#### 使用 `+` 语法的多个 Trait 约束

我们还可以指定多个 trait 约束。假设我们想要 `notify` 在 `item` 上使用显示格式以及 `summarize`：我们在 `notify` 定义中指定 `item` 必须同时实现 `Display` 和 `Summary`。我们可以使用 `+` 语法这样做：

```rust,ignore
pub fn notify(item: &(impl Summary + Display)) {
```

`+` 语法也适用于泛型类型上的 trait 约束：

```rust,ignore
pub fn notify<T: Summary + Display>(item: &T) {
```

指定了两个 trait 约束后，`notify` 的主体可以调用 `summarize` 并使用 `{}` 格式化 `item`。

#### 使用 `where` 子句的更清晰的 Trait 约束

使用太多 trait 约束有其缺点。每个泛型都有自己的 trait 约束，所以具有多个泛型类型参数的函数可以在函数名称和其参数列表之间包含大量 trait 约束信息，使函数签名难以阅读。因此，Rust 在函数签名之后使用 `where` 子句内指定 trait 约束的替代语法。所以，而不是写这个：

```rust,ignore
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```

我们可以使用 `where` 子句，如下所示：

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-07-where-clause/src/lib.rs:here}}
```

此函数的签名不那么杂乱：函数名称、参数列表和返回类型紧密在一起，类似于没有大量 trait 约束的函数。

### 返回实现 Trait 的类型

我们还可以在返回位置使用 `impl Trait` 语法来返回实现 trait 的某种类型的值，如下所示：

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-05-returning-impl-trait/src/lib.rs:here}}
```

通过使用 `impl Summary` 作为返回类型，我们指定 `returns_summarizable` 函数返回实现 `Summary` trait 的某种类型，而不命名具体类型。在这种情况下，`returns_summarizable` 返回一个 `SocialPost`，但调用此函数的代码不需要知道这一点。

仅通过它实现的 trait 指定返回类型的能力在闭包和迭代器的上下文中特别有用，我们在第 13 章中介绍。闭包和迭代器创建只有编译器知道的类型或非常长的类型。`impl Trait` 语法让你可以简洁地指定函数返回实现 `Iterator` trait 的某种类型，而无需写出非常长的类型。

但是，如果你只返回单个类型，则只能使用 `impl Trait`。例如，这段代码返回 `NewsArticle` 或 `SocialPost`，返回类型指定为 `impl Summary`，不会工作：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-06-impl-trait-returns-one-type/src/lib.rs:here}}
```

由于编译器如何实现 `impl Trait` 语法的限制，不允许返回 `NewsArticle` 或 `SocialPost`。我们将在第 18 章的["使用 Trait 对象抽象共享行为"][trait-objects]<!-- ignore -->部分介绍如何编写具有此行为的函数。

### 使用 Trait 约束有条件地实现方法

通过使用带有使用泛型类型参数的 `impl` 块的 trait 约束，我们可以有条件地为实现指定 trait 的类型实现方法。例如，代码清单 10-15 中的类型 `Pair<T>` 总是实现 `new` 函数以返回 `Pair<T>` 的新实例（回想第 5 章的["方法语法"][methods]<!-- ignore -->部分，`Self` 是 `impl` 块类型的类型别名，在这种情况下是 `Pair<T>`）。但是在下一个 `impl` 块中，只有当其内部类型 `T` 实现启用比较的 `PartialOrd` trait _和_ 启用打印的 `Display` trait 时，`Pair<T>` 才实现 `cmp_display` 方法。

<Listing number="10-15" file-name="src/lib.rs" caption="根据 trait 约束有条件地在泛型类型上实现方法">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-15/src/lib.rs}}
```

</Listing>

我们也可以有条件地为实现另一个 trait 的任何类型实现 trait。在满足 trait 约束的任何类型上的 trait 实现称为 _覆盖实现_，并在 Rust 标准库中广泛使用。例如，标准库在任何实现 `Display` trait 的类型上实现 `ToString` trait。标准库中的 `impl` 块看起来类似于此代码：

```rust,ignore
impl<T: Display> ToString for T {
    // --snip--
}
```

因为标准库具有此覆盖实现，我们可以在任何实现 `Display` trait 的类型上调用由 `ToString` trait 定义的 `to_string` 方法。例如，我们可以将整数转换为它们对应的 `String` 值，如下所示，因为整数实现 `Display`：

```rust
let s = 3.to_string();
```

覆盖实现出现在 trait 文档的"实现者"部分。

Trait 和 trait 约束让我们编写使用泛型类型参数来减少重复的代码，但也向编译器指定我们希望泛型类型具有特定行为。然后编译器可以使用 trait 约束信息来检查与我们代码一起使用的所有具体类型是否提供正确的行为。在动态类型语言中，如果我们在未定义该方法的类型上调用方法，我们会在运行时收到错误。但是 Rust 将这些错误移到编译时，以便我们在代码甚至能够运行之前被迫修复问题。此外，我们不必编写在运行时检查行为的代码，因为我们已经在编译时检查过了。这样做可以提高性能，而不必放弃泛型的灵活性。

[trait-objects]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[methods]: ch05-03-method-syntax.html#method-syntax
