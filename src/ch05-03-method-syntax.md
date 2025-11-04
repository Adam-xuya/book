## 方法

方法类似于函数：我们使用 `fn` 关键字和名称声明它们，它们可以有参数和返回值，并且它们包含在从其他地方调用该方法时运行的一些代码。与函数不同，方法在结构体（或枚举或 trait 对象，我们分别在第 6 章和[第 18 章][trait-objects]<!-- ignore -->中讨论）的上下文中定义，它们的第一个参数总是 `self`，它表示正在调用该方法的结构体实例。

<!-- Old headings. Do not remove or links may break. -->

<a id="defining-methods"></a>

### 方法语法

让我们更改将 `Rectangle` 实例作为参数的 `area` 函数，而是在 `Rectangle` 结构体上定义 `area` 方法，如清单 5-13 所示。

<Listing number="5-13" file-name="src/main.rs" caption="在 `Rectangle` 结构体上定义 `area` 方法">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-13/src/main.rs}}
```

</Listing>

为了在 `Rectangle` 的上下文中定义函数，我们为 `Rectangle` 启动一个 `impl`（实现）块。此 `impl` 块内的所有内容都将与 `Rectangle` 类型关联。然后，我们将 `area` 函数移动到 `impl` 花括号内，并将第一个（在这种情况下，也是唯一的）参数更改为签名和主体内各处的 `self`。在 `main` 中，我们调用 `area` 函数并将 `rect1` 作为参数传递的地方，我们可以改为使用_方法语法_在我们的 `Rectangle` 实例上调用 `area` 方法。方法语法位于实例之后：我们添加一个点，后跟方法名、括号和任何参数。

在 `area` 的签名中，我们使用 `&self` 而不是 `rectangle: &Rectangle`。`&self` 实际上是 `self: &Self` 的简写。在 `impl` 块内，类型 `Self` 是 `impl` 块所针对的类型的别名。方法必须在其第一个参数中有一个名为 `self` 的 `Self` 类型参数，所以 Rust 允许您仅在第一个参数位置使用名称 `self` 来缩写。请注意，我们仍然需要在 `self` 简写前面使用 `&` 来指示此方法借用 `Self` 实例，就像我们在 `rectangle: &Rectangle` 中所做的那样。方法可以获取 `self` 的所有权，不可变地借用 `self`（就像我们在这里所做的那样），或可变地借用 `self`，就像它们可以任何其他参数一样。

我们在这里选择 `&self` 的原因与我们在函数版本中使用 `&Rectangle` 的原因相同：我们不想获取所有权，我们只想读取结构体中的数据，而不是写入它。如果我们想要更改作为方法操作的一部分调用方法的实例，我们将使用 `&mut self` 作为第一个参数。拥有一个通过仅使用 `self` 作为第一个参数来获取实例所有权的方法是罕见的；当方法将 `self` 转换为其他东西并且您想要防止调用者在转换后使用原始实例时，通常使用此技术。

使用方法而不是函数的主要原因，除了提供方法语法和不必在每个方法的签名中重复 `self` 的类型之外，是为了组织。我们已经将所有可以对类型的实例执行的操作放在一个 `impl` 块中，而不是让我们代码的未来用户在我们提供的库中的各个地方搜索 `Rectangle` 的功能。

请注意，我们可以选择给方法与结构体的字段之一相同的名称。例如，我们可以在 `Rectangle` 上定义一个方法，该方法也命名为 `width`：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-06-method-field-interaction/src/main.rs:here}}
```

</Listing>

这里，我们选择使 `width` 方法如果实例的 `width` 字段中的值大于 `0` 则返回 `true`，如果值为 `0` 则返回 `false`：我们可以将同名方法中的字段用于任何目的。在 `main` 中，当我们在 `rect1.width` 后跟括号时，Rust 知道我们指的是方法 `width`。当我们不使用括号时，Rust 知道我们指的是字段 `width`。

通常，但不总是，当我们给方法与字段相同的名称时，我们希望它只返回字段中的值而不做其他任何事情。这样的方法称为_访问器_，Rust 不会像其他语言那样自动为结构体字段实现它们。访问器很有用，因为您可以使字段私有但方法公开，从而作为类型的公共 API 的一部分启用对该字段的只读访问。我们将在[第 7 章][public]<!-- ignore -->中讨论什么是公共和私有，以及如何将字段或方法指定为公共或私有。

> ### `->` 运算符在哪里？
>
> 在 C 和 C++ 中，使用两个不同的运算符来调用方法：如果您直接在对象上调用方法，则使用 `.`；如果您在指向对象的指针上调用方法并需要首先解引用指针，则使用 `->`。换句话说，如果 `object` 是指针，`object->something()` 类似于 `(*object).something()`。
>
> Rust 没有等同于 `->` 运算符；相反，Rust 有一个称为_自动引用和解引用_的功能。调用方法是 Rust 中具有此行为的少数地方之一。
>
> 这是它的工作原理：当您使用 `object.something()` 调用方法时，Rust 自动添加 `&`、`&mut` 或 `*`，以便 `object` 匹配方法的签名。换句话说，以下是相同的：
>
> <!-- CAN'T EXTRACT SEE BUG https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust
> # #[derive(Debug,Copy,Clone)]
> # struct Point {
> #     x: f64,
> #     y: f64,
> # }
> #
> # impl Point {
> #    fn distance(&self, other: &Point) -> f64 {
> #        let x_squared = f64::powi(other.x - self.x, 2);
> #        let y_squared = f64::powi(other.y - self.y, 2);
> #
> #        f64::sqrt(x_squared + y_squared)
> #    }
> # }
> # let p1 = Point { x: 0.0, y: 0.0 };
> # let p2 = Point { x: 5.0, y: 6.5 };
> p1.distance(&p2);
> (&p1).distance(&p2);
> ```
>
> 第一个看起来更清晰。这种自动引用行为有效，因为方法有一个明确的接收者——`self` 的类型。给定方法的接收者和名称，Rust 可以明确地确定方法是读取（`&self`）、改变（`&mut self`）还是消耗（`self`）。Rust 使方法接收者的借用隐式这一事实是使所有权在实践中符合人体工程学的重要组成部分。

### 具有更多参数的方法

让我们通过在 `Rectangle` 结构体上实现第二个方法来练习使用方法。这次我们希望 `Rectangle` 的实例接受另一个 `Rectangle` 实例，如果第二个 `Rectangle` 可以完全适合 `self`（第一个 `Rectangle`），则返回 `true`；否则，它应该返回 `false`。也就是说，一旦我们定义了 `can_hold` 方法，我们希望能够编写清单 5-14 中显示的程序。

<Listing number="5-14" file-name="src/main.rs" caption="使用尚未编写的 `can_hold` 方法">

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-14/src/main.rs}}
```

</Listing>

预期输出将如下所示，因为 `rect2` 的两个维度都小于 `rect1` 的维度，但 `rect3` 比 `rect1` 宽：

```text
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

我们知道我们想要定义一个方法，所以它将在 `impl Rectangle` 块内。方法名称将是 `can_hold`，它将接受另一个 `Rectangle` 的不可变借用作为参数。我们可以通过查看调用该方法的代码来告诉参数的类型：
`rect1.can_hold(&rect2)` 传入 `&rect2`，这是对 `rect2`（`Rectangle` 的实例）的不可变借用。这是有道理的，因为我们只需要读取 `rect2`（而不是写入，这意味着我们需要可变借用），我们希望 `main` 保留 `rect2` 的所有权，以便我们可以在调用 `can_hold` 方法后再次使用它。`can_hold` 的返回值将是布尔值，实现将检查 `self` 的宽度和高度是否分别大于另一个 `Rectangle` 的宽度和高度。让我们从清单 5-13 中的 `impl` 块添加新的 `can_hold` 方法，如清单 5-15 所示。

<Listing number="5-15" file-name="src/main.rs" caption="在 `Rectangle` 上实现 `can_hold` 方法，该方法接受另一个 `Rectangle` 实例作为参数">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-15/src/main.rs:here}}
```

</Listing>

当我们使用清单 5-14 中的 `main` 函数运行此代码时，我们将得到所需的输出。方法可以采用多个参数，我们在 `self` 参数之后将它们添加到签名中，这些参数的工作方式就像函数中的参数一样。

### 关联函数

在 `impl` 块内定义的所有函数都称为_关联函数_，因为它们与 `impl` 后命名的类型关联。我们可以定义没有 `self` 作为第一个参数的关联函数（因此不是方法），因为它们不需要类型的实例来工作。我们已经使用过这样的函数：在 `String` 类型上定义的 `String::from` 函数。

不是方法的关联函数通常用于将返回结构体新实例的构造函数。这些通常称为 `new`，但 `new` 不是特殊名称，也不是内置在语言中的。例如，我们可以选择提供一个名为 `square` 的关联函数，该函数将具有一个维度参数并使用该参数作为宽度和高度，从而使创建正方形 `Rectangle` 更容易，而不必指定相同的值两次：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-03-associated-functions/src/main.rs:here}}
```

函数的返回类型和函数体中的 `Self` 关键字是出现在 `impl` 关键字后的类型的别名，在这种情况下是 `Rectangle`。

要调用此关联函数，我们使用带有结构体名称的 `::` 语法；`let sq = Rectangle::square(3);` 是一个示例。此函数由结构体命名空间化：`::` 语法用于关联函数和由模块创建的命名空间。我们将在[第 7 章][modules]<!-- ignore -->中讨论模块。

### 多个 `impl` 块

每个结构体允许有多个 `impl` 块。例如，清单 5-15 等同于清单 5-16 中显示的代码，该代码在每个方法都有自己的 `impl` 块中。

<Listing number="5-16" caption="使用多个 `impl` 块重写清单 5-15">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-16/src/main.rs:here}}
```

</Listing>

没有理由将这些方法分离到多个 `impl` 块中，但这是有效的语法。我们将在第 10 章看到一个多个 `impl` 块有用的案例，我们将在那里讨论泛型类型和 trait。

## 总结

结构体让您创建对您的域有意义的自定义类型。通过使用结构体，您可以将相关的数据片段保持连接在一起，并为每个片段命名以使您的代码清晰。在 `impl` 块中，您可以定义与您的类型关联的函数，方法是让您指定结构体实例具有的行为的一种关联函数。

但结构体不是您可以创建自定义类型的唯一方式：让我们转向 Rust 的枚举功能，为您的工具箱添加另一个工具。

[enums]: ch06-00-enums.html
[trait-objects]: ch18-02-trait-objects.md
[public]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
