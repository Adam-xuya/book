## 不安全 Rust

到目前为止我们讨论的所有代码都在编译时强制执行 Rust 的内存安全保证。但是，Rust 内部隐藏着第二种语言，它不强制执行这些内存安全保证：它被称为_不安全 Rust_，与常规 Rust 一样工作，但为我们提供了额外的超能力。

不安全 Rust 存在是因为，从本质上讲，静态分析是保守的。当编译器尝试确定代码是否维护保证时，拒绝一些有效程序比接受一些无效程序更好。虽然代码_可能_没问题，但如果 Rust 编译器没有足够的信息来确信，它将拒绝代码。在这些情况下，你可以使用不安全代码来告诉编译器，“相信我，我知道我在做什么。”但是，请注意，你使用不安全 Rust 的风险自负：如果你不正确使用不安全代码，可能会由于内存不安全而出现问题，例如空指针解引用。

Rust 有不安全替身的另一个原因是底层计算机硬件本质上是不安全的。如果 Rust 不允许你执行不安全操作，你将无法执行某些任务。Rust 需要允许你进行低级系统编程，例如直接与操作系统交互，甚至编写自己的操作系统。使用低级系统编程是该语言的目标之一。让我们探索我们可以用不安全 Rust 做什么以及如何做。

<!-- Old headings. Do not remove or links may break. -->

<a id="unsafe-superpowers"></a>

### 执行不安全超能力

要切换到不安全 Rust，请使用 `unsafe` 关键字，然后开始一个包含不安全代码的新块。你可以在不安全 Rust 中执行五个操作，这些操作在安全 Rust 中无法执行，我们称之为_不安全超能力_。这些超能力包括：

1. 解引用原始指针。
1. 调用不安全函数或方法。
1. 访问或修改可变静态变量。
1. 实现不安全 trait。
1. 访问 `union` 的字段。

重要的是要理解 `unsafe` 不会关闭借用检查器或禁用 Rust 的任何其他安全检查：如果你在不安全代码中使用引用，它仍将被检查。`unsafe` 关键字只允许你访问这五个功能，然后编译器不会检查它们的内存安全性。在不安全块内，你仍然会得到一定程度的安全性。

此外，`unsafe` 并不意味着块内的代码必然危险或肯定会有内存安全问题：意图是作为程序员，你将确保 `unsafe` 块内的代码将以有效的方式访问内存。

人是会犯错误的，错误会发生，但通过要求这五个不安全操作位于用 `unsafe` 注释的块内，你将知道与内存安全相关的任何错误必须在 `unsafe` 块内。保持 `unsafe` 块较小；以后当你调查内存错误时，你会感激的。

为了尽可能隔离不安全代码，最好将此类代码封装在安全抽象中并提供安全 API，我们将在本章稍后检查不安全函数和方法时讨论这一点。标准库的部分实现为已审核的不安全代码的安全抽象。将不安全代码包装在安全抽象中可以防止 `unsafe` 的使用泄漏到你或你的用户可能想要使用用 `unsafe` 代码实现的功能的所有地方，因为使用安全抽象是安全的。

让我们依次查看这五个不安全超能力。我们还将查看一些为不安全代码提供安全接口的抽象。

### 解引用原始指针

在第4章的[“悬垂引用”][dangling-references]<!-- ignore -->部分中，我们提到编译器确保引用始终有效。不安全 Rust 有两种新类型，称为_原始指针_，它们类似于引用。与引用一样，原始指针可以是不可变的或可变的，分别写为 `*const T` 和 `*mut T`。星号不是解引用运算符；它是类型名称的一部分。在原始指针的上下文中，_不可变_意味着指针在解引用后不能直接赋值。

与引用和智能指针不同，原始指针：

- 允许忽略借用规则，对同一位置具有不可变和可变指针或多个可变指针
- 不保证指向有效内存
- 允许为空
- 不实现任何自动清理

通过选择退出让 Rust 强制执行这些保证，你可以放弃有保证的安全性，以换取更高的性能或与 Rust 保证不适用的另一种语言或硬件交互的能力。

代码清单20-1显示了如何创建不可变和可变原始指针。

<Listing number="20-1" caption="使用原始借用运算符创建原始指针">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-01/src/main.rs:here}}
```

</Listing>

请注意，我们在此代码中不包含 `unsafe` 关键字。我们可以在安全代码中创建原始指针；我们只是不能在不安全块外解引用原始指针，正如你稍后看到的。

我们通过使用原始借用运算符创建了原始指针：`&raw const num` 创建一个 `*const i32` 不可变原始指针，`&raw mut num` 创建一个 `*mut i32` 可变原始指针。因为我们直接从局部变量创建它们，我们知道这些特定的原始指针是有效的，但我们不能对任何原始指针做出这样的假设。

为了演示这一点，接下来我们将创建一个原始指针，我们无法确定其有效性，使用关键字 `as` 将值强制转换，而不是使用原始借用运算符。代码清单20-2显示了如何创建指向内存中任意位置的原始指针。尝试使用任意内存是未定义的：该地址可能有数据，也可能没有，编译器可能优化代码，使其没有内存访问，或者程序可能因段错误而终止。通常，没有好的理由编写这样的代码，特别是在你可以使用原始借用运算符的情况下，但这是可能的。

<Listing number="20-2" caption="创建指向任意内存地址的原始指针">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-02/src/main.rs:here}}
```

</Listing>

回想一下，我们可以在安全代码中创建原始指针，但我们不能解引用原始指针并读取指向的数据。在代码清单20-3中，我们在原始指针上使用解引用运算符 `*`，这需要 `unsafe` 块。

<Listing number="20-3" caption="在 `unsafe` 块内解引用原始指针">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-03/src/main.rs:here}}
```

</Listing>

创建指针没有害处；只有当我们尝试访问它指向的值时，我们才可能最终处理无效值。

还要注意，在代码清单20-1和20-3中，我们创建了 `*const i32` 和 `*mut i32` 原始指针，它们都指向同一内存位置，即存储 `num` 的位置。如果我们改为尝试创建对 `num` 的不可变和可变引用，代码将无法编译，因为 Rust 的所有权规则不允许在有任何不可变引用的同时存在可变引用。使用原始指针，我们可以创建指向同一位置的可变指针和不可变指针，并通过可变指针更改数据，这可能会创建数据竞争。小心！

考虑到所有这些危险，为什么你还要使用原始指针？一个主要用例是在与 C 代码交互时，正如你将在下一节中看到的。另一种情况是在构建借用检查器不理解的安全抽象时。我们将介绍不安全函数，然后查看使用不安全代码的安全抽象的示例。

### 调用不安全函数或方法

你可以在不安全块中执行的第二种操作是调用不安全函数。不安全函数和方法看起来与常规函数和方法完全相同，但它们在定义的其余部分之前有一个额外的 `unsafe`。此上下文中的 `unsafe` 关键字表示函数在我们调用此函数时需要维护的要求，因为 Rust 无法保证我们已经满足这些要求。通过在不安全块内调用不安全函数，我们表示我们已经阅读了此函数的文档，并负责维护函数的契约。

这是一个名为 `dangerous` 的不安全函数，其主体中不执行任何操作：

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-01-unsafe-fn/src/main.rs:here}}
```

我们必须在单独的不安全块内调用 `dangerous` 函数。如果我们尝试在没有 `unsafe` 块的情况下调用 `dangerous`，我们将收到错误：

```console
{{#include ../listings/ch20-advanced-features/output-only-01-missing-unsafe/output.txt}}
```

有了 `unsafe` 块，我们向 Rust 断言我们已经阅读了函数的文档，我们理解如何正确使用它，并且我们已经验证我们正在履行函数的契约。

要在不安全函数的主体中执行不安全操作，你仍然需要使用 `unsafe` 块，就像在常规函数中一样，如果你忘记，编译器会警告你。这帮助我们保持 `unsafe` 块尽可能小，因为可能不需要在整个函数主体中进行不安全操作。

#### 在不安全代码上创建安全抽象

仅仅因为函数包含不安全代码并不意味着我们需要将整个函数标记为不安全。实际上，将不安全代码包装在安全函数中是一种常见的抽象。例如，让我们研究标准库中的 `split_at_mut` 函数，它需要一些不安全代码。我们将探索如何实现它。这个安全方法在可变切片上定义：它接受一个切片，并通过在作为参数给出的索引处拆分切片来使其成为两个。代码清单20-4显示了如何使用 `split_at_mut`。

<Listing number="20-4" caption="使用安全的 `split_at_mut` 函数">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-04/src/main.rs:here}}
```

</Listing>

我们不能仅使用安全 Rust 实现此函数。尝试可能看起来像代码清单20-5，它不会编译。为了简单起见，我们将 `split_at_mut` 实现为函数而不是方法，并且仅用于 `i32` 值的切片而不是泛型类型 `T`。

<Listing number="20-5" caption="尝试仅使用安全 Rust 实现 `split_at_mut`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-05/src/main.rs:here}}
```

</Listing>

此函数首先获取切片的总长度。然后，它通过检查索引是否小于或等于长度来断言作为参数给出的索引在切片内。断言意味着如果我们传递一个大于长度的索引来拆分切片，函数将在尝试使用该索引之前 panic。

然后，我们在元组中返回两个可变切片：一个从原始切片的开始到 `mid` 索引，另一个从 `mid` 到切片的结束。

当我们尝试编译代码清单20-5中的代码时，我们将收到错误：

```console
{{#include ../listings/ch20-advanced-features/listing-20-05/output.txt}}
```

Rust 的借用检查器无法理解我们正在借用切片的不同部分；它只知道我们两次从同一切片借用。借用切片的不同部分在根本上是没问题的，因为两个切片不重叠，但 Rust 不够聪明，无法知道这一点。当我们知道代码没问题，但 Rust 不知道时，是时候使用不安全代码了。

代码清单20-6显示了如何使用 `unsafe` 块、原始指针和一些对不安全函数的调用来使 `split_at_mut` 的实现工作。

<Listing number="20-6" caption="在 `split_at_mut` 函数的实现中使用不安全代码">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-06/src/main.rs:here}}
```

</Listing>

回想一下第4章的[“切片类型”][the-slice-type]<!-- ignore -->部分，切片是指向某些数据的指针和切片的长度。我们使用 `len` 方法获取切片的长度，使用 `as_mut_ptr` 方法访问切片的原始指针。在这种情况下，因为我们有一个指向 `i32` 值的可变切片，`as_mut_ptr` 返回类型为 `*mut i32` 的原始指针，我们将其存储在变量 `ptr` 中。

我们保持断言 `mid` 索引在切片内。然后，我们进入不安全代码：`slice::from_raw_parts_mut` 函数接受原始指针和长度，并创建切片。我们使用此函数创建一个从 `ptr` 开始且长度为 `mid` 项的切片。然后，我们在 `ptr` 上调用 `add` 方法，将 `mid` 作为参数，以获得从 `mid` 开始的原始指针，我们使用该指针和 `mid` 之后的剩余项数作为长度创建切片。

`slice::from_raw_parts_mut` 函数是不安全的，因为它接受原始指针，必须信任该指针是有效的。原始指针上的 `add` 方法也是不安全的，因为它必须信任偏移位置也是有效指针。因此，我们必须在调用 `slice::from_raw_parts_mut` 和 `add` 周围放置 `unsafe` 块，以便我们可以调用它们。通过查看代码并添加 `mid` 必须小于或等于 `len` 的断言，我们可以告诉 `unsafe` 块内使用的所有原始指针都将是切片内数据的有效指针。这是 `unsafe` 的可接受和适当的使用。

请注意，我们不需要将结果 `split_at_mut` 函数标记为 `unsafe`，我们可以从安全 Rust 调用此函数。我们创建了一个到不安全代码的安全抽象，实现了使用 `unsafe` 代码以安全方式实现的函数，因为它仅从此函数可以访问的数据创建有效指针。

相比之下，代码清单20-7中对 `slice::from_raw_parts_mut` 的使用在切片被使用时可能会崩溃。此代码接受任意内存位置并创建一个长度为 10,000 项的切片。

<Listing number="20-7" caption="从任意内存位置创建切片">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-07/src/main.rs:here}}
```

</Listing>

我们不拥有此任意位置的内存，并且不能保证此代码创建的切片包含有效的 `i32` 值。尝试将 `values` 用作有效切片会导致未定义行为。

#### 使用 `extern` 函数调用外部代码

有时你的 Rust 代码可能需要与用另一种语言编写的代码交互。为此，Rust 有关键字 `extern`，它有助于创建和使用_外部函数接口（FFI）_，这是编程语言定义函数并启用不同（外部）编程语言调用这些函数的一种方式。

代码清单20-8演示了如何设置与 C 标准库中的 `abs` 函数的集成。在 `extern` 块内声明的函数从 Rust 代码调用通常是不安全的，所以 `extern` 块也必须标记为 `unsafe`。原因是其他语言不强制执行 Rust 的规则和保证，Rust 无法检查它们，因此责任落在程序员身上以确保安全。

<Listing number="20-8" file-name="src/main.rs" caption="声明和调用在另一种语言中定义的 `extern` 函数">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-08/src/main.rs}}
```

</Listing>

在 `unsafe extern "C"` 块内，我们列出我们想要调用的来自另一种语言的外部函数的名称和签名。`"C"` 部分定义了外部函数使用的_应用程序二进制接口（ABI）_：ABI 定义了如何在汇编级别调用函数。`"C"` ABI 是最常见的，遵循 C 编程语言的 ABI。关于 Rust 支持的所有 ABI 的信息可在[Rust 参考文档][ABI]中找到。

在 `unsafe extern` 块内声明的每个项目都隐式不安全。但是，一些 FFI 函数*确实*可以安全调用。例如，C 标准库中的 `abs` 函数没有任何内存安全考虑，我们知道它可以用任何 `i32` 调用。在这种情况下，我们可以使用 `safe` 关键字来表示这个特定函数是安全的，即使它在 `unsafe extern` 块中。一旦我们进行了更改，调用它就不再需要 `unsafe` 块，如代码清单20-9所示。

<Listing number="20-9" file-name="src/main.rs" caption="在 `unsafe extern` 块内明确将函数标记为 `safe` 并安全调用它">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-09/src/main.rs}}
```

</Listing>

将函数标记为 `safe` 并不会使其本质上安全！相反，这就像你向 Rust 做出的承诺，它是安全的。你仍然有责任确保这个承诺得到遵守！

#### 从其他语言调用 Rust 函数

我们也可以使用 `extern` 创建允许其他语言调用 Rust 函数的接口。我们不是创建整个 `extern` 块，而是添加 `extern` 关键字并为相关函数在 `fn` 关键字之前指定要使用的 ABI。我们还需要添加 `#[unsafe(no_mangle)]` 注释，告诉 Rust 编译器不要破坏此函数的名称。_名称修饰_是编译器将我们给函数的名称更改为包含更多信息以供编译过程的其他部分使用但可读性较差的不同名称的时候。每个编程语言编译器修饰名称的方式略有不同，因此对于其他语言可以命名的 Rust 函数，我们必须禁用 Rust 编译器的名称修饰。这是不安全的，因为如果没有内置的修饰，跨库可能会有名称冲突，所以确保我们选择的名称在没有修饰的情况下导出是安全的，这是我们的责任。

在以下示例中，我们使 `call_from_c` 函数可以从 C 代码访问，在它编译为共享库并从 C 链接后：

```
#[unsafe(no_mangle)]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
```

这种 `extern` 用法只需要在属性中使用 `unsafe`，而不是在 `extern` 块上。

### 访问或修改可变静态变量

在本书中，我们还没有讨论过全局变量，Rust 确实支持它们，但可能会与 Rust 的所有权规则产生问题。如果两个线程正在访问同一个可变全局变量，它可能导致数据竞争。

在 Rust 中，全局变量称为_静态_变量。代码清单20-10显示了以字符串切片作为值的静态变量的声明和使用示例。

<Listing number="20-10" file-name="src/main.rs" caption="定义和使用不可变静态变量">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-10/src/main.rs}}
```

</Listing>

静态变量类似于常量，我们在第3章的[“声明常量”][constants]<!-- ignore -->部分中讨论过。静态变量的名称按约定为 `SCREAMING_SNAKE_CASE`。静态变量只能存储具有 `'static` 生命周期的引用，这意味着 Rust 编译器可以找出生命周期，我们不需要显式注释它。访问不可变静态变量是安全的。

常量与不可变静态变量之间的细微差别是静态变量中的值在内存中具有固定地址。使用该值将始终访问相同的数据。另一方面，常量允许在每次使用时复制其数据。另一个区别是静态变量可以是可变的。访问和修改可变静态变量是_不安全的_。代码清单20-11显示了如何声明、访问和修改名为 `COUNTER` 的可变静态变量。

<Listing number="20-11" file-name="src/main.rs" caption="读取或写入可变静态变量是不安全的。">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-11/src/main.rs}}
```

</Listing>

与常规变量一样，我们使用 `mut` 关键字指定可变性。任何从 `COUNTER` 读取或写入的代码都必须位于 `unsafe` 块内。代码清单20-11中的代码编译并打印 `COUNTER: 3`，正如我们所期望的，因为它是单线程的。让多个线程访问 `COUNTER` 可能会导致数据竞争，所以这是未定义行为。因此，我们需要将整个函数标记为 `unsafe` 并记录安全限制，以便任何调用函数的人都知道他们可以安全地做什么和不允许做什么。

每当我们编写不安全函数时，习惯上编写以 `SAFETY` 开头的注释，解释调用者需要做什么才能安全调用函数。同样，每当我们执行不安全操作时，习惯上编写以 `SAFETY` 开头的注释，解释如何维护安全规则。

此外，编译器将默认拒绝任何尝试通过编译器 lint 创建对可变静态变量的引用。你必须通过添加 `#[allow(static_mut_refs)]` 注释明确选择退出该 lint 的保护，或者通过使用原始借用运算符之一创建的原始指针访问可变静态变量。这包括引用被隐式创建的情况，例如在此代码清单中的 `println!` 中使用时。要求通过原始指针创建对静态可变变量的引用有助于使使用它们的安全要求更加明显。

对于全局可访问的可变数据，很难确保没有数据竞争，这就是为什么 Rust 认为可变静态变量是不安全的。在可能的情况下，最好使用我们在第16章中讨论的并发技术和线程安全智能指针，以便编译器检查来自不同线程的数据访问是否安全完成。

### 实现不安全 Trait

我们可以使用 `unsafe` 来实现不安全 trait。当 trait 的至少一个方法有一些编译器无法验证的不变量时，trait 是不安全的。我们通过在 `trait` 之前添加 `unsafe` 关键字来声明 trait 是 `unsafe` 的，并将 trait 的实现也标记为 `unsafe`，如代码清单20-12所示。

<Listing number="20-12" caption="定义和实现不安全 trait">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-12/src/main.rs:here}}
```

</Listing>

通过使用 `unsafe impl`，我们承诺我们将维护编译器无法验证的不变量。

例如，回想一下我们在第16章的[“使用 `Send` 和 `Sync` 扩展并发”][send-and-sync]<!-- ignore -->部分中讨论的 `Send` 和 `Sync` 标记 traits：如果我们的类型完全由实现 `Send` 和 `Sync` 的其他类型组成，编译器会自动实现这些 traits。如果我们实现一个包含不实现 `Send` 或 `Sync` 的类型的类型，例如原始指针，并且我们希望将该类型标记为 `Send` 或 `Sync`，我们必须使用 `unsafe`。Rust 无法验证我们的类型维护了它可以安全地跨线程发送或从多个线程访问的保证；因此，我们需要手动进行这些检查，并用 `unsafe` 表示这一点。

### 访问 Union 的字段

仅与 `unsafe` 一起工作的最后一个操作是访问 union 的字段。*Union*类似于 `struct`，但在特定实例中一次只使用一个声明的字段。Unions 主要用于与 C 代码中的 unions 交互。访问 union 字段是不安全的，因为 Rust 无法保证当前存储在 union 实例中的数据的类型。你可以在[Rust 参考文档][unions]中了解更多关于 unions 的信息。

### 使用 Miri 检查不安全代码

编写不安全代码时，你可能想要检查你编写的内容实际上是安全和正确的。最好的方法之一是使用 Miri，这是 Rust 官方工具，用于检测未定义行为。虽然借用检查器是一个在编译时工作的_静态_工具，但 Miri 是一个在运行时工作的_动态_工具。它通过运行你的程序或其测试套件来检查你的代码，并检测你何时违反它理解的关于 Rust 应该如何工作的规则。

使用 Miri 需要 Rust 的 nightly 版本（我们在[附录 G：Rust 的制作方式和“Nightly Rust”][nightly]<!-- ignore -->中更详细地讨论）。你可以通过输入 `rustup +nightly component add miri` 来安装 Rust 的 nightly 版本和 Miri 工具。这不会更改项目使用的 Rust 版本；它只是将工具添加到你的系统，以便你可以在需要时使用它。你可以通过在项目上输入 `cargo +nightly miri run` 或 `cargo +nightly miri test` 来运行 Miri。

关于这有多有用的示例，考虑当我们针对代码清单20-7运行它时会发生什么。

```console
{{#include ../listings/ch20-advanced-features/listing-20-07/output.txt}}
```

Miri 正确地警告我们正在将整数强制转换为指针，这可能是一个问题，但 Miri 无法确定是否存在问题，因为它不知道指针是如何起源的。然后，Miri 在代码清单20-7具有未定义行为的地方返回错误，因为我们有一个悬垂指针。感谢 Miri，我们现在知道存在未定义行为的风险，我们可以考虑如何使代码安全。在某些情况下，Miri 甚至可以就如何修复错误提出建议。

Miri 不会捕获你在编写不安全代码时可能出错的所有内容。Miri 是一个动态分析工具，所以它只捕获实际运行的代码的问题。这意味着你需要将它与良好的测试技术结合使用，以增加对你编写的不安全代码的信心。Miri 也不涵盖代码可能不健全的每种可能方式。

换句话说：如果 Miri _确实_捕获到问题，你知道有一个错误，但仅仅因为 Miri _没有_捕获错误并不意味着没有问题。不过，它可以捕获很多。尝试在本章的其他不安全代码示例上运行它，看看它说了什么！

你可以在[其 GitHub 仓库][miri]中了解更多关于 Miri 的信息。

<!-- Old headings. Do not remove or links may break. -->

<a id="when-to-use-unsafe-code"></a>

### 正确使用不安全代码

使用 `unsafe` 来使用刚才讨论的五个超能力之一并不是错误的，甚至不会被反对，但让 `unsafe` 代码正确是更棘手的，因为编译器无法帮助维护内存安全。当你有理由使用 `unsafe` 代码时，你可以这样做，并且显式的 `unsafe` 注释使在问题发生时更容易追踪问题的根源。每当你编写不安全代码时，你可以使用 Miri 来帮助你更有信心你编写的代码维护了 Rust 的规则。

要更深入地探索如何有效地使用不安全 Rust，请阅读 Rust 的 `unsafe` 官方指南，[The Rustonomicon][nomicon]。

[dangling-references]: ch04-02-references-and-borrowing.html#dangling-references
[ABI]: https://doc.rust-lang.org/reference/items/external-blocks.html#abi
[constants]: ch03-01-variables-and-mutability.html#declaring-constants
[send-and-sync]: ch16-04-extensible-concurrency-sync-and-send.html
[the-slice-type]: ch04-03-slices.html#the-slice-type
[unions]: https://doc.rust-lang.org/reference/items/unions.html
[miri]: https://github.com/rust-lang/miri
[editions]: appendix-05-editions.html
[nightly]: appendix-07-nightly-rust.html
[nomicon]: https://doc.rust-lang.org/nomicon/
