## 高级函数和闭包

本节探讨与函数和闭包相关的一些高级特性，包括函数指针和返回闭包。

### 函数指针

我们已经讨论过如何将闭包传递给函数；你也可以将常规函数传递给函数！当你想要传递你已经定义的函数而不是定义新闭包时，此技术很有用。函数强制转换为类型 `fn`（小写 _f_），不要与 `Fn` 闭包 trait 混淆。`fn` 类型称为_函数指针_。使用函数指针传递函数将允许你将函数用作其他函数的参数。

指定参数是函数指针的语法类似于闭包的语法，如代码清单20-28所示，我们定义了一个函数 `add_one`，它将 1 添加到其参数。函数 `do_twice` 接受两个参数：指向接受 `i32` 参数并返回 `i32` 的任何函数的函数指针，以及一个 `i32` 值。`do_twice` 函数调用函数 `f` 两次，向其传递 `arg` 值，然后将两个函数调用结果相加。`main` 函数使用参数 `add_one` 和 `5` 调用 `do_twice`。

<Listing number="20-28" file-name="src/main.rs" caption="使用 `fn` 类型接受函数指针作为参数">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-28/src/main.rs}}
```

</Listing>

此代码打印 `The answer is: 12`。我们指定 `do_twice` 中的参数 `f` 是一个接受一个类型为 `i32` 的参数并返回 `i32` 的 `fn`。然后我们可以在 `do_twice` 的主体中调用 `f`。在 `main` 中，我们可以将函数名 `add_one` 作为第一个参数传递给 `do_twice`。

与闭包不同，`fn` 是类型而不是 trait，所以我们直接将 `fn` 指定为参数类型，而不是用 `Fn` traits 之一作为 trait 约束声明泛型类型参数。

函数指针实现所有三个闭包 traits（`Fn`、`FnMut` 和 `FnOnce`），这意味着你总是可以将函数指针作为期望闭包的函数的参数传递。最好使用泛型类型和闭包 trait 之一编写函数，以便你的函数可以接受函数或闭包。

也就是说，你只想接受 `fn` 而不接受闭包的一个例子是在与没有闭包的外部代码交互时：C 函数可以接受函数作为参数，但 C 没有闭包。

作为你可以使用内联定义的闭包或命名函数的示例，让我们看看标准库中 `Iterator` trait 提供的 `map` 方法的使用。要使用 `map` 方法将数字向量转换为字符串向量，我们可以使用闭包，如代码清单20-29所示。

<Listing number="20-29" caption="使用闭包和 `map` 方法将数字转换为字符串">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-29/src/main.rs:here}}
```

</Listing>

或者，我们可以将命名函数作为 `map` 的参数，而不是闭包。代码清单20-30显示了这将是什么样子。

<Listing number="20-30" caption="使用 `String::to_string` 函数和 `map` 方法将数字转换为字符串">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-30/src/main.rs:here}}
```

</Listing>

请注意，我们必须使用我们在[“高级 Traits”][advanced-traits]<!-- ignore -->部分讨论的完全限定语法，因为有多个名为 `to_string` 的函数可用。

在这里，我们使用 `ToString` trait 中定义的 `to_string` 函数，标准库已为任何实现 `Display` 的类型实现了它。

回想一下第6章的[“枚举值”][enum-values]<!-- ignore -->部分，我们定义的每个枚举变体的名称也成为一个初始化函数。我们可以将这些初始化函数用作实现闭包 traits 的函数指针，这意味着我们可以将初始化函数指定为接受闭包的方法的参数，如代码清单20-31所示。

<Listing number="20-31" caption="使用枚举初始化函数和 `map` 方法从数字创建 `Status` 实例">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-31/src/main.rs:here}}
```

</Listing>

在这里，我们通过使用 `Status::Value` 的初始化函数，使用 `map` 调用的范围内的每个 `u32` 值创建 `Status::Value` 实例。有些人更喜欢这种风格，有些人更喜欢使用闭包。它们编译为相同的代码，所以使用对你来说更清楚的任何一种风格。

### 返回闭包

闭包由 traits 表示，这意味着你不能直接返回闭包。在大多数你可能想要返回 trait 的情况下，你可以改为使用实现 trait 的具体类型作为函数的返回值。但是，你通常不能对闭包这样做，因为它们没有可返回的具体类型；例如，如果闭包从其作用域捕获任何值，你不允许使用函数指针 `fn` 作为返回类型。

相反，你通常会使用我们在第10章学到的 `impl Trait` 语法。你可以使用 `Fn`、`FnOnce` 和 `FnMut` 返回任何函数类型。例如，代码清单20-32中的代码将编译得很好。

<Listing number="20-32" caption="使用 `impl Trait` 语法从函数返回闭包">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-32/src/lib.rs}}
```

</Listing>

但是，正如我们在第13章的[“推断和注释闭包类型”][closure-types]<!-- ignore -->部分中提到的，每个闭包也是它自己独特的类型。如果你需要处理具有相同签名但不同实现的多个函数，你需要为它们使用 trait 对象。考虑如果你编写如代码清单20-33所示的代码会发生什么。

<Listing file-name="src/main.rs" number="20-33" caption="创建一个由返回 `impl Fn` 类型的函数定义的闭包的 `Vec<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-33/src/main.rs}}
```

</Listing>

这里我们有两个函数，`returns_closure` 和 `returns_initialized_closure`，它们都返回 `impl Fn(i32) -> i32`。请注意，它们返回的闭包是不同的，即使它们实现相同的类型。如果我们尝试编译此代码，Rust 会告诉我们它不会工作：

```text
{{#include ../listings/ch20-advanced-features/listing-20-33/output.txt}}
```

错误消息告诉我们，每当我们返回 `impl Trait` 时，Rust 会创建一个唯一的_不透明类型_，这是一种我们无法看到 Rust 为我们构造的细节的类型，我们也不能猜测 Rust 将生成以自己编写的类型。因此，即使这些函数返回实现相同 trait `Fn(i32) -> i32` 的闭包，Rust 为每个生成的不透明类型也是不同的。（这类似于 Rust 为不同的 async 块产生不同的具体类型，即使它们具有相同的输出类型，正如我们在第17章的[“`Pin` 类型和 `Unpin` Trait”][future-types]<!-- ignore -->中看到的。）我们已经看到了这个问题的解决方案几次：我们可以使用 trait 对象，如代码清单20-34所示。

<Listing number="20-34" caption="创建一个由返回 `Box<dyn Fn>` 的函数定义的闭包的 `Vec<T>`，以便它们具有相同的类型">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-34/src/main.rs:here}}
```

</Listing>

此代码将编译得很好。有关 trait 对象的更多信息，请参阅第18章的[“使用 Trait 对象抽象共享行为”][trait-objects]<!-- ignore -->部分。

接下来，让我们看看宏！

[advanced-traits]: ch20-02-advanced-traits.html#advanced-traits
[enum-values]: ch06-01-defining-an-enum.html#enum-values
[closure-types]: ch13-01-closures.html#closure-type-inference-and-annotation
[future-types]: ch17-03-more-futures.html
[trait-objects]: ch18-02-trait-objects.html
