<!-- Old headings. Do not remove or links may break. -->

<a id="treating-smart-pointers-like-regular-references-with-the-deref-trait"></a>
<a id="treating-smart-pointers-like-regular-references-with-deref"></a>

## 将智能指针视为常规引用

实现 `Deref` trait 允许你自定义 _解引用操作符_ `*` 的行为（不要与乘法或 glob 操作符混淆）。通过以智能指针可以像常规引用一样对待的方式实现 `Deref`，你可以编写对引用进行操作的代码，并将该代码与智能指针一起使用。

让我们首先看看解引用操作符如何处理常规引用。然后，我们将尝试定义一个行为类似于 `Box<T>` 的自定义类型，并看看为什么解引用操作符在我们新定义的类型上不像引用那样工作。我们将探索如何实现 `Deref` trait 使智能指针能够以类似于引用的方式工作。然后，我们将查看 Rust 的 deref coercion 功能以及它如何让我们使用引用或智能指针。

<!-- Old headings. Do not remove or links may break. -->

<a id="following-the-pointer-to-the-value-with-the-dereference-operator"></a>
<a id="following-the-pointer-to-the-value"></a>

### 跟随引用到值

常规引用是指针的一种类型，将指针视为指向存储在别处的值的箭头的一种方式。在代码清单 15-6 中，我们创建对 `i32` 值的引用，然后使用解引用操作符跟随引用到值。

<Listing number="15-6" file-name="src/main.rs" caption="使用解引用操作符跟随引用到 `i32` 值">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-06/src/main.rs}}
```

</Listing>

变量 `x` 保存 `i32` 值 `5`。我们将 `y` 设置为对 `x` 的引用。我们可以断言 `x` 等于 `5`。但是，如果我们想对 `y` 中的值进行断言，我们必须使用 `*y` 来跟随引用到它指向的值（因此，_解引用_），以便编译器可以比较实际值。一旦我们解引用 `y`，我们就可以访问 `y` 指向的整数值，我们可以将其与 `5` 进行比较。

如果我们尝试编写 `assert_eq!(5, y);` 而不是，我们将得到此编译错误：

```console
{{#include ../listings/ch15-smart-pointers/output-only-01-comparing-to-reference/output.txt}}
```

不允许比较数字和对数字的引用，因为它们是不同的类型。我们必须使用解引用操作符来跟随引用到它指向的值。

### 像引用一样使用 `Box<T>`

我们可以重写代码清单 15-6 中的代码以使用 `Box<T>` 而不是引用；在代码清单 15-7 中，在 `Box<T>` 上使用的解引用操作符的功能与在代码清单 15-6 中在引用上使用的解引用操作符的功能相同。

<Listing number="15-7" file-name="src/main.rs" caption="在 `Box<i32>` 上使用解引用操作符">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-07/src/main.rs}}
```

</Listing>

代码清单 15-7 和代码清单 15-6 之间的主要区别是，这里我们将 `y` 设置为指向 `x` 的复制值的 box 实例，而不是指向 `x` 值的引用。在最后一个断言中，我们可以使用解引用操作符来跟随 box 的指针，就像当 `y` 是引用时我们所做的那样。接下来，我们将通过定义我们自己的 box 类型来探索 `Box<T>` 的特殊之处，它使我们能够使用解引用操作符。

### 定义我们自己的智能指针

让我们构建一个类似于标准库提供的 `Box<T>` 类型的包装器类型，以体验智能指针类型默认情况下如何与引用行为不同。然后，我们将查看如何添加使用解引用操作符的能力。

> 注意：我们即将构建的 `MyBox<T>` 类型与真正的 `Box<T>` 之间有一个重大区别：我们的版本不会将其数据存储在堆上。我们专注于此示例的 `Deref`，所以数据实际存储的位置不如类似指针的行为重要。

`Box<T>` 类型最终定义为具有一个元素的元组结构体，所以代码清单 15-8 以相同方式定义 `MyBox<T>` 类型。我们还将定义一个 `new` 函数以匹配在 `Box<T>` 上定义的 `new` 函数。

<Listing number="15-8" file-name="src/main.rs" caption="定义 `MyBox<T>` 类型">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-08/src/main.rs:here}}
```

</Listing>

我们定义一个名为 `MyBox` 的结构体，并声明一个泛型参数 `T`，因为我们希望我们的类型保存任何类型的值。`MyBox` 类型是具有一个类型 `T` 元素的元组结构体。`MyBox::new` 函数接受一个类型 `T` 的参数，并返回保存传入值的 `MyBox` 实例。

让我们尝试将代码清单 15-7 中的 `main` 函数添加到代码清单 15-8 中，并将其更改为使用我们定义的 `MyBox<T>` 类型而不是 `Box<T>`。代码清单 15-9 中的代码不会编译，因为 Rust 不知道如何解引用 `MyBox`。

<Listing number="15-9" file-name="src/main.rs" caption="尝试以与使用引用和 `Box<T>` 相同的方式使用 `MyBox<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-09/src/main.rs:here}}
```

</Listing>

这是结果编译错误：

```console
{{#include ../listings/ch15-smart-pointers/listing-15-09/output.txt}}
```

我们的 `MyBox<T>` 类型不能被解引用，因为我们还没有在我们的类型上实现该能力。为了启用使用 `*` 操作符的解引用，我们实现 `Deref` trait。

<!-- Old headings. Do not remove or links may break. -->

<a id="treating-a-type-like-a-reference-by-implementing-the-deref-trait"></a>

### 实现 `Deref` Trait

正如在第 10 章的["在类型上实现 Trait"][impl-trait]<!-- ignore -->中讨论的，要实现 trait，我们需要为 trait 的必需方法提供实现。标准库提供的 `Deref` trait 要求我们实现一个名为 `deref` 的方法，该方法借用 `self` 并返回对内部数据的引用。代码清单 15-10 包含要添加到 `MyBox<T>` 定义的 `Deref` 实现。

<Listing number="15-10" file-name="src/main.rs" caption="在 `MyBox<T>` 上实现 `Deref`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-10/src/main.rs:here}}
```

</Listing>

`type Target = T;` 语法为 `Deref` trait 定义关联类型以供使用。关联类型是声明泛型参数的稍微不同的方式，但你现在不需要担心它们；我们将在第 20 章更详细地介绍它们。

我们用 `&self.0` 填充 `deref` 方法的主体，以便 `deref` 返回我们想使用 `*` 操作符访问的值的引用；回想第 5 章的["使用元组结构体创建不同类型"][tuple-structs]<!-- ignore -->，`.0` 访问元组结构体中的第一个值。代码清单 15-9 中在 `MyBox<T>` 值上调用 `*` 的 `main` 函数现在编译，并且断言通过！

没有 `Deref` trait，编译器只能解引用 `&` 引用。`deref` 方法为编译器提供了获取实现 `Deref` 的任何类型的值并调用 `deref` 方法以获取它知道如何解引用的引用的能力。

当我们在代码清单 15-9 中输入 `*y` 时，在幕后 Rust 实际运行此代码：

```rust,ignore
*(y.deref())
```

Rust 将 `*` 操作符替换为对 `deref` 方法的调用，然后是普通解引用，这样我们就不必考虑是否需要调用 `deref` 方法。此 Rust 功能让我们编写的代码无论我们拥有常规引用还是实现 `Deref` 的类型都能相同地工作。

`deref` 方法返回对值的引用的原因，以及 `*(y.deref())` 中括号外的普通解引用仍然必要的原因，与所有权系统有关。如果 `deref` 方法直接返回值而不是对值的引用，值将从 `self` 移出。在这种情况下或在我们使用解引用操作符的大多数情况下，我们不想获取 `MyBox<T>` 内部值的所有权。

注意，`*` 操作符被替换为对 `deref` 方法的调用，然后是对 `*` 操作符的调用，每次我们在代码中使用 `*` 时只调用一次。因为 `*` 操作符的替换不会无限递归，我们最终得到类型 `i32` 的数据，这与代码清单 15-9 中 `assert_eq!` 中的 `5` 匹配。

<!-- Old headings. Do not remove or links may break. -->

<a id="implicit-deref-coercions-with-functions-and-methods"></a>
<a id="using-deref-coercions-in-functions-and-methods"></a>

### 在函数和方法中使用 Deref Coercion

_Deref coercion_ 将实现对 `Deref` trait 的类型的引用转换为对另一种类型的引用。例如，deref coercion 可以将 `&String` 转换为 `&str`，因为 `String` 实现 `Deref` trait，使其返回 `&str`。Deref coercion 是 Rust 对函数和方法的参数执行的便利功能，它只适用于实现 `Deref` trait 的类型。当我们将对特定类型值的引用作为参数传递给函数或方法，而该函数或方法定义中的参数类型不匹配时，它会自动发生。对 `deref` 方法的调用序列将我们提供的类型转换为参数所需的类型。

Deref coercion 被添加到 Rust 中，以便编写函数和方法调用的程序员不需要添加那么多显式引用和使用 `&` 和 `*` 的解引用。deref coercion 功能还让我们编写更多可以与引用或智能指针一起工作的代码。

为了查看 deref coercion 的实际效果，让我们使用我们在代码清单 15-8 中定义的 `MyBox<T>` 类型以及我们在代码清单 15-10 中添加的 `Deref` 实现。代码清单 15-11 显示了具有字符串切片参数的函数定义。

<Listing number="15-11" file-name="src/main.rs" caption="一个 `hello` 函数，参数 `name` 的类型为 `&str`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-11/src/main.rs:here}}
```

</Listing>

我们可以使用字符串切片作为参数调用 `hello` 函数，例如 `hello("Rust");`。Deref coercion 使得使用对类型 `MyBox<String>` 值的引用调用 `hello` 成为可能，如代码清单 15-12 所示。

<Listing number="15-12" file-name="src/main.rs" caption="使用对 `MyBox<String>` 值的引用调用 `hello`，由于 deref coercion 而起作用">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-12/src/main.rs:here}}
```

</Listing>

这里我们使用参数 `&m` 调用 `hello` 函数，这是对 `MyBox<String>` 值的引用。因为我们在代码清单 15-10 中在 `MyBox<T>` 上实现了 `Deref` trait，Rust 可以通过调用 `deref` 将 `&MyBox<String>` 转换为 `&String`。标准库在 `String` 上提供了 `Deref` 的实现，它返回字符串切片，这在 `Deref` 的 API 文档中。Rust 再次调用 `deref` 以将 `&String` 转换为 `&str`，这与 `hello` 函数的定义匹配。

如果 Rust 没有实现 deref coercion，我们将不得不编写代码清单 15-13 中的代码而不是代码清单 15-12 中的代码，以使用类型 `&MyBox<String>` 的值调用 `hello`。

<Listing number="15-13" file-name="src/main.rs" caption="如果 Rust 没有 deref coercion，我们将不得不编写的代码">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-13/src/main.rs:here}}
```

</Listing>

`(*m)` 将 `MyBox<String>` 解引用为 `String`。然后，`&` 和 `[..]` 获取等于整个字符串的 `String` 的字符串切片，以匹配 `hello` 的签名。没有 deref coercions 的代码更难阅读、编写和理解，因为所有这些符号都涉及其中。Deref coercion 允许 Rust 自动为我们处理这些转换。

当为所涉及的类型定义 `Deref` trait 时，Rust 将分析类型并根据需要使用 `Deref::deref` 多次以获取匹配参数类型的引用。需要插入 `Deref::deref` 的次数在编译时解析，所以利用 deref coercion 没有运行时损失！

<!-- Old headings. Do not remove or links may break. -->

<a id="how-deref-coercion-interacts-with-mutability"></a>

### 处理可变引用的 Deref Coercion

类似于你如何使用 `Deref` trait 覆盖不可变引用上的 `*` 操作符，你可以使用 `DerefMut` trait 覆盖可变引用上的 `*` 操作符。

Rust 在三种情况下进行 deref coercion：

1. 从 `&T` 到 `&U` 当 `T: Deref<Target=U>`
2. 从 `&mut T` 到 `&mut U` 当 `T: DerefMut<Target=U>`
3. 从 `&mut T` 到 `&U` 当 `T: Deref<Target=U>`

前两种情况相同，除了第二种实现可变性。第一种情况说明如果你有 `&T`，并且 `T` 实现对某个类型 `U` 的 `Deref`，你可以透明地获得 `&U`。第二种情况说明相同的 deref coercion 发生在可变引用上。

第三种情况更棘手：Rust 也会将可变引用强制转换为不可变引用。但反过来 _不可能_：不可变引用永远不会强制转换为可变引用。由于借用规则，如果你有可变引用，该可变引用必须是对该数据的唯一引用（否则，程序不会编译）。将一个可变引用转换为一个不可变引用永远不会破坏借用规则。将不可变引用转换为可变引用将要求初始不可变引用是对该数据的唯一不可变引用，但借用规则不保证这一点。因此，Rust 不能假设将不可变引用转换为可变引用是可能的。

[impl-trait]: ch10-02-traits.html#implementing-a-trait-on-a-type
[tuple-structs]: ch05-01-defining-structs.html#creating-different-types-with-tuple-structs
