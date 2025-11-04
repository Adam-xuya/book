## 高级类型

Rust 类型系统有一些我们到目前为止提到但尚未讨论的特性。我们将首先讨论新类型，因为我们将检查它们为什么作为类型很有用。然后，我们将转到类型别名，这是一个类似于新类型但语义略有不同的特性。我们还将讨论 `!` 类型和动态大小类型。

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-newtype-pattern-for-type-safety-and-abstraction"></a>

### 使用新类型模式实现类型安全和抽象

本节假设你已经阅读了前面的部分[“使用新类型模式实现外部 Traits”][newtype]<!-- ignore -->。新类型模式对于超出我们到目前为止讨论的任务也很有用，包括静态强制执行值永远不会混淆并指示值的单位。你在代码清单20-16中看到了使用新类型指示单位的示例：回想一下 `Millimeters` 和 `Meters` 结构体在新类型中包装了 `u32` 值。如果我们编写一个参数类型为 `Millimeters` 的函数，我们将无法编译一个意外尝试使用类型为 `Meters` 或普通 `u32` 的值调用该函数的程序。

我们也可以使用新类型模式来抽象掉类型的某些实现细节：新类型可以公开与私有内部类型的 API 不同的公共 API。

新类型也可以隐藏内部实现。例如，我们可以提供一个 `People` 类型来包装 `HashMap<i32, String>`，它存储与名称关联的人的 ID。使用 `People` 的代码只会与我们提供的公共 API 交互，例如向 `People` 集合添加名称字符串的方法；该代码不需要知道我们在内部为名称分配 `i32` ID。新类型模式是实现封装以隐藏实现细节的轻量级方法，我们在第18章的[“隐藏实现细节的封装”][encapsulation-that-hides-implementation-details]<!-- ignore -->部分中讨论过。

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-type-synonyms-with-type-aliases"></a>

### 类型同义词和类型别名

Rust 提供了声明_类型别名_的能力，以给现有类型另一个名称。为此，我们使用 `type` 关键字。例如，我们可以创建别名 `Kilometers` 为 `i32`，如下所示：

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:here}}
```

现在别名 `Kilometers` 是 `i32` 的_同义词_；与我们
        
        
        

在代码清单20-16中创建的 `Millimeters` 和 `Meters` 类型不同，`Kilometers` 不是单独的、新类型。类型为 `Kilometers` 的值将被视为与类型为 `i32` 的值相同：

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:there}}
```

因为 `Kilometers` 和 `i32` 是相同类型，我们可以添加两种类型的值，并且可以将 `Kilometers` 值传递给接受 `i32` 参数的函数。但是，使用此方法，我们不会获得从之前讨论的新类型模式中获得的类型检查优势。换句话说，如果我们在某处混淆了 `Kilometers` 和 `i32` 值，编译器不会给我们错误。

类型同义词的主要用例是减少重复。例如，我们可能有一个冗长的类型，如下所示：

```rust,ignore
Box<dyn Fn() + Send + 'static>
```

在函数签名和作为类型注释在代码中到处编写此冗长类型可能很繁琐且容易出错。想象一下，有一个充满代码清单20-25中那样的代码的项目。

<Listing number="20-25" caption="在许多地方使用长类型">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-25/src/main.rs:here}}
```

</Listing>

类型别名通过减少重复使此代码更易于管理。在代码清单20-26中，我们为冗长类型引入了一个名为 `Thunk` 的别名，可以用较短的别名 `Thunk` 替换该类型的所有使用。

<Listing number="20-26" caption="引入类型别名 `Thunk` 以减少重复">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-26/src/main.rs:here}}
```

</Listing>

此代码更容易读写！为类型别名选择有意义的名称可以帮助传达你的意图（_thunk_ 是稍后评估的代码的词，所以它是存储的闭包的适当名称）。

类型别名也常用于 `Result<T, E>` 类型以减少重复。考虑标准库中的 `std::io` 模块。I/O 操作经常返回 `Result<T, E>` 来处理操作失败的情况。此库有一个 `std::io::Error` 结构体，表示所有可能的 I/O 错误。`std::io` 中的许多函数将返回 `Result<T, E>`，其中 `E` 是 `std::io::Error`，例如 `Write` trait 中的这些函数：

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-05-write-trait/src/lib.rs}}
```

`Result<..., Error>` 重复了很多次。因此，`std::io` 有这个类型别名声明：

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:here}}
```

因为此声明在 `std::io` 模块中，我们可以使用完全限定别名 `std::io::Result<T>`；也就是说，`E` 填充为 `std::io::Error` 的 `Result<T, E>`。`Write` trait 函数签名最终看起来像这样：

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:there}}
```

类型别名在两个方面有帮助：它使代码更容易编写_并且_它为我们提供了跨所有 `std::io` 的一致接口。因为它是别名，它只是另一个 `Result<T, E>`，这意味着我们可以使用对 `Result<T, E>` 有效的任何方法，以及特殊语法，如 `?` 运算符。

### 永不返回的 Never 类型

Rust 有一个名为 `!` 的特殊类型，在类型理论术语中被称为_空类型_，因为它没有值。我们更喜欢称它为_never 类型_，因为它在函数永远不会返回时代表返回类型。这是一个示例：

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-07-never-type/src/lib.rs:here}}
```

此代码读作“函数 `bar` 返回 never”。返回 never 的函数称为_发散函数_。我们无法创建类型 `!` 的值，所以 `bar` 永远不可能返回。

但是你永远无法创建值的类型有什么用？回想一下代码清单2-5中的代码，这是猜数字游戏的一部分；我们在这里在代码清单20-27中重现了它的一部分。

<Listing number="20-27" caption="以 `continue` 结尾的 `match` 分支">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:ch19}}
```

</Listing>

当时，我们跳过了此代码中的一些细节。在第6章的[“`match` 控制流构造”][the-match-control-flow-construct]<!-- ignore -->部分中，我们讨论了 `match` 分支必须全部返回相同类型。因此，例如，以下代码不起作用：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-08-match-arms-different-types/src/main.rs:here}}
```

此代码中 `guess` 的类型必须是整数_和_字符串，Rust 要求 `guess` 只有一种类型。那么，`continue` 返回什么？在代码清单20-27中，我们如何被允许从一个分支返回 `u32` 并让另一个分支以 `continue` 结尾？

正如你可能已经猜到的，`continue` 有一个 `!` 值。也就是说，当 Rust 计算 `guess` 的类型时，它查看两个匹配分支，前者具有 `u32` 值，后者具有 `!` 值。因为 `!` 永远不能有值，Rust 确定 `guess` 的类型是 `u32`。

描述此行为的正式方式是类型 `!` 的表达式可以强制转换为任何其他类型。我们被允许以 `continue` 结束此 `match` 分支，因为 `continue` 不返回值；相反，它将控制权移回循环的顶部，所以在 `Err` 情况下，我们永远不会为 `guess` 赋值。

never 类型与 `panic!` 宏也很有用。回想一下我们在 `Option<T>` 值上调用的 `unwrap` 函数，以产生值或 panic，其定义如下：

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-09-unwrap-definition/src/lib.rs:here}}
```

在此代码中，发生了与代码清单20-27中的 `match` 相同的事情：Rust 看到 `val` 具有类型 `T`，`panic!` 具有类型 `!`，所以整个 `match` 表达式的结果是 `T`。此代码有效，因为 `panic!` 不产生值；它结束程序。在 `None` 情况下，我们不会从 `unwrap` 返回值，所以此代码是有效的。

具有类型 `!` 的最后一个表达式是循环：

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-10-loop-returns-never/src/main.rs:here}}
```

在这里，循环永远不会结束，所以 `!` 是表达式的值。但是，如果我们包含 `break`，这将不是真的，因为循环在到达 `break` 时会终止。

### 动态大小类型和 `Sized` Trait

Rust 需要了解其类型的某些细节，例如为特定类型的值分配多少空间。这使其类型系统的一个角落在一开始有点令人困惑：_动态大小类型_的概念。有时称为_DST_或_无大小类型_，这些类型让我们编写使用值的代码，这些值的大小我们只能在运行时知道。

让我们深入了解称为 `str` 的动态大小类型的细节，我们在整本书中一直在使用它。没错，不是 `&str`，而是 `str` 本身，是一个 DST。在许多情况下，例如存储用户输入的文本时，我们无法知道字符串的长度，直到运行时。这意味着我们无法创建类型为 `str` 的变量，也无法接受类型为 `str` 的参数。考虑以下代码，它不起作用：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-11-cant-create-str/src/main.rs:here}}
```

Rust 需要知道为特定类型的任何值分配多少内存，并且类型的所有值必须使用相同的内存量。如果 Rust 允许我们编写此代码，这两个 `str` 值将需要占用相同的空间。但它们有不同的长度：`s1` 需要 12 字节的存储空间，`s2` 需要 15 字节。这就是为什么不可能创建持有动态大小类型的变量。

那么，我们做什么？在这种情况下，你已经知道答案：我们将 `s1` 和 `s2` 的类型设为字符串切片（`&str`）而不是 `str`。回想一下第4章的[“字符串切片”][string-slices]<!-- ignore -->部分，切片数据结构只存储切片的起始位置和长度。因此，虽然 `&T` 是存储 `T` 所在内存地址的单个值，但字符串切片是_两个_值：`str` 的地址及其长度。因此，我们可以在编译时知道字符串切片值的大小：它是 `usize` 长度的两倍。也就是说，我们总是知道字符串切片的大小，无论它引用的字符串有多长。一般来说，这是在 Rust 中使用动态大小类型的方式：它们有一个额外的元数据位，存储动态信息的大小。动态大小类型的黄金法则是我们必须始终将动态大小类型的值放在某种指针后面。

我们可以将 `str` 与各种指针组合：例如，`Box<str>` 或 `Rc<str>`。事实上，你之前见过这个，但使用不同的动态大小类型：traits。每个 trait 都是一个动态大小类型，我们可以通过使用 trait 的名称来引用它。在第18章的[“使用 Trait 对象抽象共享行为”][using-trait-objects-to-abstract-over-shared-behavior]<!-- ignore -->部分中，我们提到要使用 traits 作为 trait 对象，我们必须将它们放在指针后面，例如 `&dyn Trait` 或 `Box<dyn Trait>`（`Rc<dyn Trait>` 也可以工作）。

为了处理 DST，Rust 提供了 `Sized` trait 来确定类型的大小是否在编译时已知。此 trait 自动为所有在编译时已知大小的类型实现。此外，Rust 隐式地向每个泛型函数添加 `Sized` 约束。也就是说，像这样的泛型函数定义：

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-12-generic-fn-definition/src/lib.rs}}
```

实际上被视为我们编写了这样：

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-13-generic-implicit-sized-bound/src/lib.rs}}
```

默认情况下，泛型函数只能处理在编译时已知大小的类型。但是，你可以使用以下特殊语法来放宽此限制：

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-14-generic-maybe-sized/src/lib.rs}}
```

对 `?Sized` 的 trait 约束意味着“`T` 可能是或可能不是 `Sized`”，此表示法覆盖了泛型类型必须在编译时已知大小的默认值。具有此含义的 `?Trait` 语法仅适用于 `Sized`，不适用于任何其他 traits。

还要注意，我们将 `t` 参数的类型从 `T` 切换为 `&T`。因为类型可能不是 `Sized`，我们需要在某种指针后面使用它。在这种情况下，我们选择了引用。

接下来，我们将讨论函数和闭包！

[encapsulation-that-hides-implementation-details]: ch18-01-what-is-oo.html#encapsulation-that-hides-implementation-details
[string-slices]: ch04-03-slices.html#string-slices
[the-match-control-flow-construct]: ch06-02-match.html#the-match-control-flow-construct
[using-trait-objects-to-abstract-over-shared-behavior]: ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
[newtype]: ch20-02-advanced-traits.html#implementing-external-traits-with-the-newtype-pattern
