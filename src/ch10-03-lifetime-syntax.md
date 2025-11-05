## 使用生命周期验证引用

生命周期是我们已经使用的另一种泛型。与其确保类型具有我们想要的行为，生命周期确保引用在我们需要它们有效的整个时间内都有效。

我们在第 4 章的["引用和借用"][references-and-borrowing]<!-- ignore -->部分没有讨论的一个细节是 Rust 中的每个引用都有一个生命周期，这是该引用有效的范围。大多数时候，生命周期是隐式的和推断的，就像大多数时候类型是推断的一样。我们只需要在可能有多种类型时注释类型。以类似的方式，当引用的生命周期可能以几种不同方式相关时，我们必须注释生命周期。Rust 要求我们使用泛型生命周期参数注释这些关系，以确保在运行时使用的实际引用肯定有效。

注释生命周期甚至不是大多数其他编程语言都有的概念，所以这会感觉不熟悉。虽然我们不会在本章中全面介绍生命周期，但我们将讨论你可能遇到生命周期语法的常见方式，以便你可以熟悉这个概念。

<!-- Old headings. Do not remove or links may break. -->

<a id="preventing-dangling-references-with-lifetimes"></a>

### 悬垂引用

生命周期的主要目标是防止悬垂引用，如果允许它们存在，将导致程序引用它不打算引用的数据。考虑代码清单 10-16 中的程序，它有一个外部作用域和一个内部作用域。

<Listing number="10-16" caption="尝试使用其值已超出作用域的引用">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/src/main.rs}}
```

</Listing>

> 注意：代码清单 10-16、10-17 和 10-23 中的示例声明变量而不给它们初始值，所以变量名存在于外部作用域中。乍一看，这可能看起来与 Rust 没有空值冲突。但是，如果我们在给它值之前尝试使用变量，我们会得到一个编译时错误，这表明 Rust 确实不允许空值。

外部作用域声明一个名为 `r` 的变量，没有初始值，内部作用域声明一个名为 `x` 的变量，初始值为 `5`。在内部作用域内，我们尝试将 `r` 的值设置为对 `x` 的引用。然后，内部作用域结束，我们尝试打印 `r` 中的值。此代码无法编译，因为 `r` 引用的值在我们尝试使用它之前已经超出作用域。这是错误消息：

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/output.txt}}
```

错误消息说变量 `x`"活得不够长"。原因是当内部作用域在第 7 行结束时，`x` 将超出作用域。但是 `r` 仍然对外部作用域有效；因为它的作用域更大，我们说它"活得更长"。如果 Rust 允许此代码工作，`r` 将引用在 `x` 超出作用域时被释放的内存，我们尝试对 `r` 做的任何事情都不会正常工作。那么，Rust 如何确定此代码无效？它使用借用检查器。

### 借用检查器

Rust 编译器有一个 _借用检查器_，它比较作用域以确定所有借用是否有效。代码清单 10-17 显示了与代码清单 10-16 相同的代码，但带有注释显示变量的生命周期。

<Listing number="10-17" caption="`r` 和 `x` 的生命周期注释，分别命名为 `'a` 和 `'b`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-17/src/main.rs}}
```

</Listing>

这里，我们用 `'a` 注释了 `r` 的生命周期，用 `'b` 注释了 `x` 的生命周期。如你所见，内部 `'b` 块比外部 `'a` 生命周期块小得多。在编译时，Rust 比较两个生命周期的大小，并看到 `r` 具有 `'a` 的生命周期，但它引用的内存具有 `'b` 的生命周期。程序被拒绝，因为 `'b` 比 `'a` 短：引用的主题没有引用活得长。

代码清单 10-18 修复了代码，使其没有悬垂引用，并且它可以在没有任何错误的情况下编译。

<Listing number="10-18" caption="一个有效的引用，因为数据的生命周期比引用长">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-18/src/main.rs}}
```

</Listing>

这里，`x` 具有生命周期 `'b`，在这种情况下比 `'a` 大。这意味着 `r` 可以引用 `x`，因为 Rust 知道 `r` 中的引用在 `x` 有效时总是有效的。

既然你知道引用的生命周期在哪里以及 Rust 如何分析生命周期以确保引用始终有效，让我们探索函数参数和返回值中的泛型生命周期。

### 函数中的泛型生命周期

我们将编写一个返回两个字符串切片中较长者的函数。此函数将接受两个字符串切片并返回单个字符串切片。在我们实现 `longest` 函数后，代码清单 10-19 中的代码应该打印 `The longest string is abcd`。

<Listing number="10-19" file-name="src/main.rs" caption="一个 `main` 函数，调用 `longest` 函数以查找两个字符串切片中较长者">

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-19/src/main.rs}}
```

</Listing>

注意，我们希望函数接受字符串切片，它们是引用，而不是字符串，因为我们不希望 `longest` 函数获取其参数的所有权。有关为什么我们在代码清单 10-19 中使用的参数是我们想要的参数的更多讨论，请参阅第 4 章的["字符串切片作为参数"][string-slices-as-parameters]<!-- ignore -->。

如果我们尝试实现 `longest` 函数，如代码清单 10-20 所示，它无法编译。

<Listing number="10-20" file-name="src/main.rs" caption="`longest` 函数的实现，返回两个字符串切片中较长者但尚未编译">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/src/main.rs:here}}
```

</Listing>

相反，我们得到以下关于生命周期的错误：

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/output.txt}}
```

帮助文本揭示返回类型需要在其上有一个泛型生命周期参数，因为 Rust 无法判断返回的引用是指向 `x` 还是 `y`。实际上，我们也不知道，因为此函数主体中的 `if` 块返回对 `x` 的引用，而 `else` 块返回对 `y` 的引用！

当我们定义此函数时，我们不知道将传递给此函数的具体值，所以我们不知道 `if` 情况还是 `else` 情况会执行。我们也不知道将传入的引用的具体生命周期，所以我们无法像在代码清单 10-17 和 10-18 中那样查看作用域来确定我们返回的引用是否始终有效。借用检查器也无法确定这一点，因为它不知道 `x` 和 `y` 的生命周期如何与返回值的生命周期相关。要修复此错误，我们将添加定义引用之间关系的泛型生命周期参数，以便借用检查器可以执行其分析。

### 生命周期注释语法

生命周期注释不会改变任何引用的存活时间。相反，它们描述了多个引用的生命周期之间的关系，而不影响生命周期。就像函数在签名指定泛型类型参数时可以接受任何类型一样，函数可以通过指定泛型生命周期参数来接受具有任何生命周期的引用。

生命周期注释有一个稍微不寻常的语法：生命周期参数的名称必须以撇号（`'`）开头，通常全部小写且非常短，就像泛型类型一样。大多数人使用名称 `'a` 作为第一个生命周期注释。我们将生命周期参数注释放在引用的 `&` 之后，使用空格将注释与引用的类型分开。

以下是一些示例——一个没有生命周期参数的 `i32` 引用，一个具有名为 `'a` 的生命周期参数的 `i32` 引用，以及一个也具有生命周期 `'a` 的 `i32` 可变引用：

```rust,ignore
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime
```

一个生命周期注释本身没有太大意义，因为注释旨在告诉 Rust 多个引用的泛型生命周期参数如何相互关联。让我们检查生命周期注释在 `longest` 函数的上下文中如何相互关联。

<!-- Old headings. Do not remove or links may break. -->

<a id="lifetime-annotations-in-function-signatures"></a>

### 在函数签名中

为了在函数签名中使用生命周期注释，我们需要在函数名称和参数列表之间的尖括号内声明泛型生命周期参数，就像我们对泛型类型参数所做的那样。

我们希望签名表达以下约束：返回的引用将有效，只要两个参数都有效。这是参数和返回值的生命周期之间的关系。我们将生命周期命名为 `'a`，然后将其添加到每个引用，如代码清单 10-21 所示。

<Listing number="10-21" file-name="src/main.rs" caption="`longest` 函数定义，指定签名中的所有引用必须具有相同的生命周期 `'a`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-21/src/main.rs:here}}
```

</Listing>

此代码应该编译并在我们将其与代码清单 10-19 中的 `main` 函数一起使用时产生我们想要的结果。

函数签名现在告诉 Rust，对于某个生命周期 `'a`，函数接受两个参数，这两个参数都是至少存活生命周期 `'a` 的字符串切片。函数签名还告诉 Rust，从函数返回的字符串切片将至少存活生命周期 `'a`。实际上，这意味着 `longest` 函数返回的引用的生命周期与函数参数引用的值的生命周期中较小的一个相同。这些关系是我们希望 Rust 在分析此代码时使用的关系。

记住，当我们在函数签名中指定生命周期参数时，我们并没有更改任何传入或返回的值的生命周期。相反，我们指定借用检查器应该拒绝任何不遵守这些约束的值。注意，`longest` 函数不需要确切知道 `x` 和 `y` 将存活多长时间，只需要某个作用域可以替换为 `'a`，这将满足此签名。

当在函数中注释生命周期时，注释进入函数签名，而不是函数主体。生命周期注释成为函数契约的一部分，就像签名中的类型一样。让函数签名包含生命周期契约意味着 Rust 编译器所做的分析可以更简单。如果函数的注释方式或调用方式有问题，编译器错误可以更精确地指向我们代码的特定部分和约束。相反，如果 Rust 编译器对我们希望生命周期关系的意图做出更多推断，编译器可能只能指向使用我们代码的许多步骤，远离问题的原因。

当我们将具体引用传递给 `longest` 时，替换为 `'a` 的具体生命周期是 `x` 的作用域与 `y` 的作用域重叠的部分。换句话说，泛型生命周期 `'a` 将获得等于 `x` 和 `y` 的生命周期中较小的一个的具体生命周期。因为我们用相同的生命周期参数 `'a` 注释了返回的引用，返回的引用也将对 `x` 和 `y` 的生命周期中较小的一个的长度有效。

让我们看看生命周期注释如何通过传入具有不同具体生命周期的引用来限制 `longest` 函数。代码清单 10-22 是一个简单的例子。

<Listing number="10-22" file-name="src/main.rs" caption="使用 `longest` 函数和具有不同具体生命周期的 `String` 值的引用">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-22/src/main.rs:here}}
```

</Listing>

在这个例子中，`string1` 有效直到外部作用域结束，`string2` 有效直到内部作用域结束，`result` 引用有效直到内部作用域结束的某个东西。运行此代码，你将看到借用检查器批准；它将编译并打印 `The longest string is long string is long`。

接下来，让我们尝试一个例子，显示 `result` 中引用的生命周期必须是两个参数中较小的生命周期。我们将 `result` 变量的声明移到内部作用域之外，但将值赋值给 `result` 变量保留在 `string2` 的作用域内。然后，我们将使用 `result` 的 `println!` 移到内部作用域之外，在内部作用域结束后。代码清单 10-23 中的代码无法编译。

<Listing number="10-23" file-name="src/main.rs" caption="尝试在 `string2` 超出作用域后使用 `result`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/src/main.rs:here}}
```

</Listing>

当我们尝试编译此代码时，我们得到此错误：

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/output.txt}}
```

错误显示，为了使 `result` 对 `println!` 语句有效，`string2` 需要有效直到外部作用域结束。Rust 知道这一点，因为我们使用相同的生命周期参数 `'a` 注释了函数参数和返回值的生命周期。

作为人类，我们可以查看此代码并看到 `string1` 比 `string2` 长，因此，`result` 将包含对 `string1` 的引用。因为 `string1` 尚未超出作用域，对 `string1` 的引用仍将对 `println!` 语句有效。但是，编译器在这种情况下看不到引用是有效的。我们已经告诉 Rust，`longest` 函数返回的引用的生命周期与传入的引用的生命周期中较小的一个相同。因此，借用检查器禁止代码清单 10-23 中的代码，因为它可能具有无效引用。

尝试设计更多实验，改变传递给 `longest` 函数的引用的值和生命周期，以及如何使用返回的引用。在编译之前假设你的实验是否会通过借用检查器；然后，检查一下你是否正确！

<!-- Old headings. Do not remove or links may break. -->

<a id="thinking-in-terms-of-lifetimes"></a>

### 关系

你需要指定生命周期参数的方式取决于函数的功能。例如，如果我们将 `longest` 函数的实现更改为始终返回第一个参数而不是最长的字符串切片，我们不需要在 `y` 参数上指定生命周期。以下代码将编译：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-08-only-one-reference-with-lifetime/src/main.rs:here}}
```

</Listing>

我们为参数 `x` 和返回类型指定了生命周期参数 `'a`，但没有为参数 `y` 指定，因为 `y` 的生命周期与 `x` 或返回值的生命周期没有任何关系。

当从函数返回引用时，返回类型的生命周期参数需要与其中一个参数的生命周期参数匹配。如果返回的引用 _不_ 引用其中一个参数，它必须引用在此函数内创建的值。但是，这将是一个悬垂引用，因为该值将在函数结束时超出作用域。考虑这个尝试实现的 `longest` 函数，它无法编译：

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/src/main.rs:here}}
```

</Listing>

这里，尽管我们为返回类型指定了生命周期参数 `'a`，但此实现将无法编译，因为返回值生命周期与参数的生命周期完全无关。这是我们得到的错误消息：

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/output.txt}}
```

问题是 `result` 在 `longest` 函数结束时超出作用域并被清理。我们还试图从函数返回对 `result` 的引用。我们无法指定会更改悬垂引用的生命周期参数，Rust 不会让我们创建悬垂引用。在这种情况下，最好的修复方法是返回拥有的数据类型而不是引用，以便调用函数负责清理值。

最终，生命周期语法是关于连接函数的各种参数和返回值的生命周期。一旦它们被连接，Rust 就有足够的信息来允许内存安全操作并禁止会创建悬垂指针或以其他方式违反内存安全的操作。

<!-- Old headings. Do not remove or links may break. -->

<a id="lifetime-annotations-in-struct-definitions"></a>

### 在结构体定义中

到目前为止，我们定义的结构体都保存拥有的类型。我们可以定义结构体来保存引用，但在这种情况下，我们需要在结构体定义中的每个引用上添加生命周期注释。代码清单 10-24 有一个名为 `ImportantExcerpt` 的结构体，它保存一个字符串切片。

<Listing number="10-24" file-name="src/main.rs" caption="一个保存引用的结构体，需要生命周期注释">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-24/src/main.rs}}
```

</Listing>

此结构体有单个字段 `part`，它保存一个字符串切片，这是一个引用。与泛型数据类型一样，我们在结构体名称后面的尖括号内声明泛型生命周期参数的名称，以便我们可以在结构体定义的主体中使用生命周期参数。此注释意味着 `ImportantExcerpt` 的实例不能超过它在 `part` 字段中保存的引用。

这里的 `main` 函数创建 `ImportantExcerpt` 结构体的实例，它保存对变量 `novel` 拥有的 `String` 的第一句话的引用。`novel` 中的数据在创建 `ImportantExcerpt` 实例之前就存在。此外，`novel` 直到 `ImportantExcerpt` 超出作用域之后才超出作用域，所以 `ImportantExcerpt` 实例中的引用是有效的。

### 生命周期省略

你已经了解到每个引用都有一个生命周期，并且你需要为使用引用的函数或结构体指定生命周期参数。但是，我们在代码清单 4-9 中有一个函数，在代码清单 10-25 中再次显示，它在没有生命周期注释的情况下编译。

<Listing number="10-25" file-name="src/lib.rs" caption="我们在代码清单 4-9 中定义的函数，即使参数和返回类型是引用，也在没有生命周期注释的情况下编译">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-25/src/main.rs:here}}
```

</Listing>

此函数在没有生命周期注释的情况下编译的原因是历史性的：在 Rust 的早期版本（1.0 之前）中，此代码无法编译，因为每个引用都需要显式生命周期。那时，函数签名会这样写：

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

在编写了大量 Rust 代码后，Rust 团队发现 Rust 程序员在特定情况下一遍又一遍地输入相同的生命周期注释。这些情况是可预测的，并遵循一些确定性模式。开发人员将这些模式编程到编译器的代码中，以便借用检查器可以在这些情况下推断生命周期，而不需要显式注释。

这段 Rust 历史是相关的，因为可能会出现更多确定性模式并将它们添加到编译器。将来，可能需要更少的生命周期注释。

编程到 Rust 引用分析中的模式称为 _生命周期省略规则_。这些不是程序员要遵循的规则；它们是编译器将考虑的一组特定情况，如果你的代码适合这些情况，你不需要显式编写生命周期。

省略规则不提供完全推断。如果在 Rust 应用规则后，关于引用的生命周期仍然存在歧义，编译器不会猜测剩余引用的生命周期应该是什么。编译器不会猜测，而是会给你一个错误，你可以通过添加生命周期注释来解决。

函数或方法参数上的生命周期称为 _输入生命周期_，返回值上的生命周期称为 _输出生命周期_。

编译器使用三个规则来找出没有显式注释时引用的生命周期。第一个规则适用于输入生命周期，第二个和第三个规则适用于输出生命周期。如果编译器到达三个规则的末尾，并且仍有无法确定生命周期的引用，编译器将停止并出现错误。这些规则适用于 `fn` 定义以及 `impl` 块。

第一个规则是编译器为每个作为引用的参数分配一个生命周期参数。换句话说，具有一个参数的函数获得一个生命周期参数：`fn foo<'a>(x: &'a i32)`；具有两个参数的函数获得两个单独的生命周期参数：`fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`；依此类推。

第二个规则是，如果恰好有一个输入生命周期参数，则该生命周期分配给所有输出生命周期参数：`fn foo<'a>(x: &'a i32) -> &'a i32`。

第三个规则是，如果有多个输入生命周期参数，但其中一个是 `&self` 或 `&mut self`，因为这是一个方法，`self` 的生命周期分配给所有输出生命周期参数。这第三个规则使方法更容易读写，因为需要更少的符号。

让我们假装我们是编译器。我们将应用这些规则来找出代码清单 10-25 中 `first_word` 函数签名中引用的生命周期。签名开始时没有任何与引用关联的生命周期：

```rust,ignore
fn first_word(s: &str) -> &str {
```

然后，编译器应用第一个规则，该规则指定每个参数获得自己的生命周期。我们将它称为 `'a`，就像通常一样，所以现在签名是：

```rust,ignore
fn first_word<'a>(s: &'a str) -> &str {
```

第二个规则适用，因为恰好有一个输入生命周期。第二个规则指定一个输入参数的生命周期被分配给输出生命周期，所以签名现在是：

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

现在此函数签名中的所有引用都有生命周期，编译器可以继续其分析，而无需程序员在此函数签名中注释生命周期。

让我们看另一个例子，这次使用 `longest` 函数，当我们在代码清单 10-20 中开始使用它时，它没有生命周期参数：

```rust,ignore
fn longest(x: &str, y: &str) -> &str {
```

让我们应用第一个规则：每个参数获得自己的生命周期。这次我们有两个参数而不是一个，所以我们有两个生命周期：

```rust,ignore
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
```

你可以看到第二个规则不适用，因为有一个以上的输入生命周期。第三个规则也不适用，因为 `longest` 是一个函数而不是方法，所以没有参数是 `self`。在完成所有三个规则后，我们仍然没有找出返回类型的生命周期是什么。这就是为什么我们在尝试编译代码清单 10-20 中的代码时收到错误：编译器完成了生命周期省略规则，但仍然无法找出签名中所有引用的生命周期。

因为第三个规则实际上只适用于方法签名，我们接下来将在该上下文中查看生命周期，看看为什么第三个规则意味着我们不需要经常在方法签名中注释生命周期。

<!-- Old headings. Do not remove or links may break. -->

<a id="lifetime-annotations-in-method-definitions"></a>

### 在方法定义中

当我们在具有生命周期的结构体上实现方法时，我们使用与泛型类型参数相同的语法，如代码清单 10-11 所示。我们声明和使用生命周期参数的位置取决于它们是与结构体字段相关还是与方法参数和返回值相关。

结构体字段的生命周期名称总是需要在 `impl` 关键字之后声明，然后在结构体名称之后使用，因为那些生命周期是结构体类型的一部分。

在 `impl` 块内的方法签名中，引用可能与结构体字段中引用的生命周期相关联，或者它们可能是独立的。此外，生命周期省略规则通常使得方法签名中不需要生命周期注释。让我们看一些使用我们在代码清单 10-24 中定义的名为 `ImportantExcerpt` 的结构体的示例。

首先，我们将使用一个名为 `level` 的方法，其唯一参数是对 `self` 的引用，其返回值是 `i32`，它不是对任何东西的引用：

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:1st}}
```

在 `impl` 之后的生命周期参数声明及其在类型名称之后的使用是必需的，但由于第一个省略规则，我们不需要注释对 `self` 的引用的生命周期。

这是一个应用第三个生命周期省略规则的例子：

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:3rd}}
```

有两个输入生命周期，所以 Rust 应用第一个生命周期省略规则，并给 `&self` 和 `announcement` 各自的生命周期。然后，因为其中一个参数是 `&self`，返回类型获得 `&self` 的生命周期，所有生命周期都已考虑在内。

### 静态生命周期

我们需要讨论的一个特殊生命周期是 `'static`，它表示受影响的引用 _可以_ 在整个程序持续时间内存活。所有字符串字面量都具有 `'static` 生命周期，我们可以如下注释：

```rust
let s: &'static str = "I have a static lifetime.";
```

此字符串的文本直接存储在程序的二进制文件中，始终可用。因此，所有字符串字面量的生命周期是 `'static`。

你可能在错误消息中看到使用 `'static` 生命周期的建议。但在指定 `'static` 作为引用的生命周期之前，请考虑你拥有的引用是否真的在整个程序生命周期内存活，以及你是否希望它这样做。大多数时候，建议 `'static` 生命周期的错误消息是由于尝试创建悬垂引用或可用生命周期不匹配导致的。在这种情况下，解决方案是修复这些问题，而不是指定 `'static` 生命周期。

<!-- Old headings. Do not remove or links may break. -->

<a id="generic-type-parameters-trait-bounds-and-lifetimes-together"></a>

## 泛型类型参数、Trait 约束和生命周期

让我们简要看一下在一个函数中指定泛型类型参数、trait 约束和生命周期的语法！

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-11-generics-traits-and-lifetimes/src/main.rs:here}}
```

这是来自代码清单 10-21 的 `longest` 函数，它返回两个字符串切片中较长者。但现在它有一个额外的参数，名为 `ann`，类型为泛型类型 `T`，可以由实现 `Display` trait 的任何类型填充，如 `where` 子句所指定。这个额外参数将使用 `{}` 打印，这就是为什么需要 `Display` trait 约束。因为生命周期是一种泛型，生命周期参数 `'a` 和泛型类型参数 `T` 的声明进入函数名称后尖括号内的同一列表中。

## 总结

我们在本章中涵盖了很多内容！现在你知道泛型类型参数、trait 和 trait 约束以及泛型生命周期参数，你已经准备好编写无重复的代码，这些代码可以在许多不同情况下工作。泛型类型参数让你可以将代码应用于不同类型。Trait 和 trait 约束确保即使类型是泛型的，它们也会有代码所需的行为。你学习了如何使用生命周期注释来确保这个灵活的代码不会有任何悬垂引用。所有这些分析都在编译时发生，不会影响运行时性能！

信不信由你，我们在本章讨论的主题还有更多要学习的内容：第 18 章讨论 trait 对象，这是使用 trait 的另一种方式。还有涉及生命周期注释的更复杂场景，你只需要在非常高级的场景中需要；对于这些，你应该阅读 [Rust 参考手册][reference]。但接下来，你将学习如何在 Rust 中编写测试，以便你可以确保代码按应有的方式工作。

[references-and-borrowing]: ch04-02-references-and-borrowing.html#references-and-borrowing
[string-slices-as-parameters]: ch04-03-slices.html#string-slices-as-parameters
[reference]: https://doc.rust-lang.org/reference/trait-bounds.html
