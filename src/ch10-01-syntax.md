## 泛型数据类型

我们使用泛型为函数签名或结构体等项创建定义，然后我们可以将这些定义与许多不同的具体数据类型一起使用。让我们首先看看如何使用泛型定义函数、结构体、枚举和方法。然后，我们将讨论泛型如何影响代码性能。

### 在函数定义中

当定义使用泛型的函数时，我们将泛型放在函数签名中，通常我们会在那里指定参数和返回值的数据类型。这样做使我们的代码更灵活，并为函数的调用者提供更多功能，同时防止代码重复。

继续我们的 `largest` 函数，代码清单 10-4 显示了两个函数，它们都在切片中查找最大值。然后，我们将它们合并为一个使用泛型的函数。

<Listing number="10-4" file-name="src/main.rs" caption="两个函数，仅在名称和签名中的类型上不同">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-04/src/main.rs:here}}
```

</Listing>

`largest_i32` 函数是我们在代码清单 10-3 中提取的函数，它在切片中查找最大的 `i32`。`largest_char` 函数在切片中查找最大的 `char`。函数体具有相同的代码，所以让我们通过在单个函数中引入泛型类型参数来消除重复。

为了在新函数中参数化类型，我们需要命名类型参数，就像我们为函数的值参数命名一样。你可以使用任何标识符作为类型参数名称。但我们将使用 `T`，因为按照约定，Rust 中的类型参数名称很短，通常只是一个字母，而 Rust 的类型命名约定是 UpperCamelCase。`T` 是 _type_ 的缩写，是大多数 Rust 程序员的默认选择。

当我们在函数主体中使用参数时，我们必须在签名中声明参数名称，以便编译器知道该名称的含义。类似地，当我们在函数签名中使用类型参数名称时，我们必须在使用它之前声明类型参数名称。为了定义泛型 `largest` 函数，我们将类型名称声明放在尖括号 `<>` 内，位于函数名称和参数列表之间，如下所示：

```rust,ignore
fn largest<T>(list: &[T]) -> &T {
```

我们将此定义读作"函数 `largest` 在某个类型 `T` 上是泛型的。"此函数有一个名为 `list` 的参数，它是类型 `T` 的值的切片。`largest` 函数将返回对相同类型 `T` 的值的引用。

代码清单 10-5 显示了使用泛型数据类型在其签名中的合并 `largest` 函数定义。该清单还显示了如何使用 `i32` 值切片或 `char` 值切片调用该函数。注意，此代码尚无法编译。

<Listing number="10-5" file-name="src/main.rs" caption="使用泛型类型参数的 `largest` 函数；此代码尚无法编译">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/src/main.rs}}
```

</Listing>

如果我们现在编译此代码，我们将得到此错误：

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/output.txt}}
```

帮助文本提到了 `std::cmp::PartialOrd`，这是一个 trait，我们将在下一节中讨论 trait。现在，要知道此错误说明 `largest` 的主体不会对所有可能的 `T` 类型都有效。因为我们想在主体中比较类型 `T` 的值，我们只能使用其值可以排序的类型。为了启用比较，标准库具有 `std::cmp::PartialOrd` trait，你可以在类型上实现它（有关此 trait 的更多信息，请参阅附录 C）。要修复代码清单 10-5，我们可以遵循帮助文本的建议，并将对 `T` 有效的类型限制为仅实现 `PartialOrd` 的类型。然后该清单将编译，因为标准库在 `i32` 和 `char` 上都实现了 `PartialOrd`。

### 在结构体定义中

我们还可以使用 `<>` 语法定义一个或多个字段使用泛型类型参数的结构体。代码清单 10-6 定义了一个 `Point<T>` 结构体，用于保存任何类型的 `x` 和 `y` 坐标值。

<Listing number="10-6" file-name="src/main.rs" caption="一个 `Point<T>` 结构体，保存类型 `T` 的 `x` 和 `y` 值">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-06/src/main.rs}}
```

</Listing>

在结构体定义中使用泛型的语法类似于函数定义中使用的语法。首先，我们在结构体名称后面的尖括号内声明类型参数的名称。然后，我们在结构体定义中使用泛型类型，否则我们会在那里指定具体数据类型。

注意，因为我们只使用一个泛型类型来定义 `Point<T>`，此定义说明 `Point<T>` 结构体在某个类型 `T` 上是泛型的，字段 `x` 和 `y` _都是_ 相同类型，无论该类型是什么。如果我们创建一个 `Point<T>` 实例，该实例具有不同类型的值，如代码清单 10-7 所示，我们的代码将无法编译。

<Listing number="10-7" file-name="src/main.rs" caption="字段 `x` 和 `y` 必须是相同类型，因为两者都具有相同的泛型数据类型 `T`。">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/src/main.rs}}
```

</Listing>

在这个例子中，当我们将整数值 `5` 分配给 `x` 时，我们让编译器知道泛型类型 `T` 将是此 `Point<T>` 实例的整数。然后，当我们为 `y` 指定 `4.0` 时，我们已经将其定义为与 `x` 具有相同的类型，我们将得到一个类型不匹配错误，如下所示：

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/output.txt}}
```

要定义一个 `Point` 结构体，其中 `x` 和 `y` 都是泛型但可能具有不同的类型，我们可以使用多个泛型类型参数。例如，在代码清单 10-8 中，我们将 `Point` 的定义更改为在类型 `T` 和 `U` 上是泛型的，其中 `x` 是类型 `T`，`y` 是类型 `U`。

<Listing number="10-8" file-name="src/main.rs" caption="一个 `Point<T, U>` 在两种类型上是泛型的，因此 `x` 和 `y` 可以是不同类型的值">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-08/src/main.rs}}
```

</Listing>

现在显示的所有 `Point` 实例都是允许的！你可以在定义中使用任意多个泛型类型参数，但使用超过几个会使你的代码难以阅读。如果你发现代码中需要大量泛型类型，这可能表明你的代码需要重构为更小的部分。

### 在枚举定义中

就像我们对结构体所做的那样，我们可以定义枚举以在其变体中保存泛型数据类型。让我们再看一下标准库提供的 `Option<T>` 枚举，我们在第 6 章中使用过：

```rust
enum Option<T> {
    Some(T),
    None,
}
```

这个定义现在应该对你更有意义。如你所见，`Option<T>` 枚举在类型 `T` 上是泛型的，有两个变体：`Some`，它保存一个类型 `T` 的值，以及一个不保存任何值的 `None` 变体。通过使用 `Option<T>` 枚举，我们可以表达可选值的抽象概念，并且因为 `Option<T>` 是泛型的，无论可选值的类型是什么，我们都可以使用此抽象。

枚举也可以使用多个泛型类型。我们在第 9 章使用的 `Result` 枚举的定义就是一个例子：

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`Result` 枚举在两种类型 `T` 和 `E` 上是泛型的，有两个变体：`Ok`，它保存类型 `T` 的值，以及 `Err`，它保存类型 `E` 的值。此定义使 `Result` 枚举在我们有可能会成功（返回某种类型 `T` 的值）或失败（返回某种类型 `E` 的错误）的操作的任何地方都方便使用。事实上，这就是我们在代码清单 9-3 中用来打开文件的内容，其中 `T` 在文件成功打开时填充为类型 `std::fs::File`，`E` 在打开文件时出现问题填充为类型 `std::io::Error`。

当你在代码中识别出多个结构体或枚举定义仅在它们保存的值的类型上不同的情况时，你可以通过使用泛型类型来避免重复。

### 在方法定义中

我们可以在结构体和枚举上实现方法（就像我们在第 5 章中所做的那样），并在它们的定义中也使用泛型类型。代码清单 10-9 显示了我们在代码清单 10-6 中定义的 `Point<T>` 结构体，并在其上实现了一个名为 `x` 的方法。

<Listing number="10-9" file-name="src/main.rs" caption="在 `Point<T>` 结构体上实现名为 `x` 的方法，该方法将返回对类型 `T` 的 `x` 字段的引用">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-09/src/main.rs}}
```

</Listing>

这里，我们在 `Point<T>` 上定义了一个名为 `x` 的方法，它返回字段 `x` 中数据的引用。

注意，我们必须在 `impl` 之后声明 `T`，以便我们可以使用 `T` 来指定我们在类型 `Point<T>` 上实现方法。通过在 `impl` 之后将 `T` 声明为泛型类型，Rust 可以识别 `Point` 中尖括号内的类型是泛型类型而不是具体类型。我们可以为这个泛型参数选择与结构体定义中声明的泛型参数不同的名称，但使用相同的名称是约定。如果你在声明泛型类型的 `impl` 内编写方法，该方法将在类型的任何实例上定义，无论最终替换泛型类型的具体类型是什么。

在类型上定义方法时，我们还可以指定泛型类型的约束。例如，我们可以仅在 `Point<f32>` 实例上实现方法，而不是在任何泛型类型的 `Point<T>` 实例上实现。在代码清单 10-10 中，我们使用具体类型 `f32`，这意味着我们在 `impl` 之后不声明任何类型。

<Listing number="10-10" file-name="src/main.rs" caption="仅适用于泛型类型参数 `T` 具有特定具体类型的结构体的 `impl` 块">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-10/src/main.rs:here}}
```

</Listing>

此代码意味着类型 `Point<f32>` 将有一个 `distance_from_origin` 方法；`T` 不是 `f32` 类型的其他 `Point<T>` 实例将没有定义此方法。该方法测量我们的点距坐标 (0.0, 0.0) 处的点的距离，并使用仅适用于浮点类型的数学运算。

结构体定义中的泛型类型参数并不总是与你在该结构体的方法签名中使用的参数相同。代码清单 10-11 使用泛型类型 `X1` 和 `Y1` 用于 `Point` 结构体，使用 `X2` 和 `Y2` 用于 `mixup` 方法签名，以使示例更清晰。该方法创建一个新的 `Point` 实例，其 `x` 值来自 `self` `Point`（类型 `X1`），`y` 值来自传入的 `Point`（类型 `Y2`）。

<Listing number="10-11" file-name="src/main.rs" caption="一个使用与其结构体定义不同的泛型类型的方法">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-11/src/main.rs}}
```

</Listing>

在 `main` 中，我们定义了一个 `Point`，其 `x` 为 `i32`（值为 `5`），`y` 为 `f64`（值为 `10.4`）。变量 `p2` 是一个 `Point` 结构体，其 `x` 为字符串切片（值为 `"Hello"`），`y` 为 `char`（值为 `c`）。在 `p1` 上使用参数 `p2` 调用 `mixup` 给我们 `p3`，它将有一个 `i32` 的 `x`，因为 `x` 来自 `p1`。变量 `p3` 将有一个 `char` 的 `y`，因为 `y` 来自 `p2`。`println!` 宏调用将打印 `p3.x = 5, p3.y = c`。

此示例的目的是演示一种情况，其中一些泛型参数用 `impl` 声明，一些用方法定义声明。这里，泛型参数 `X1` 和 `Y1` 在 `impl` 之后声明，因为它们与结构体定义一起使用。泛型参数 `X2` 和 `Y2` 在 `fn mixup` 之后声明，因为它们仅与方法相关。

### 使用泛型的代码性能

你可能想知道使用泛型类型参数时是否有运行时成本。好消息是使用泛型类型不会使你的程序运行得比使用具体类型慢。

Rust 通过在编译时对使用泛型的代码执行单态化来实现这一点。_单态化_ 是通过在编译时填充使用的具体类型将泛型代码转换为特定代码的过程。在此过程中，编译器执行与我们用于在代码清单 10-5 中创建泛型函数的步骤相反的操作：编译器查看调用泛型代码的所有位置，并为调用泛型代码的具体类型生成代码。

让我们通过使用标准库的泛型 `Option<T>` 枚举来看看这是如何工作的：

```rust
let integer = Some(5);
let float = Some(5.0);
```

当 Rust 编译此代码时，它执行单态化。在此过程中，编译器读取在 `Option<T>` 实例中使用的值，并识别两种 `Option<T>`：一种是 `i32`，另一种是 `f64`。因此，它将 `Option<T>` 的泛型定义扩展为专门针对 `i32` 和 `f64` 的两个定义，从而用特定定义替换泛型定义。

单态化版本的代码类似于以下内容（编译器使用与我们在此处用于说明的名称不同的名称）：

<Listing file-name="src/main.rs">

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

</Listing>

泛型 `Option<T>` 被编译器创建的特定定义替换。因为 Rust 将泛型代码编译为在每个实例中指定类型的代码，所以我们使用泛型不会产生运行时成本。当代码运行时，它的执行就像我们手动复制每个定义一样。单态化过程使 Rust 的泛型在运行时非常高效。
