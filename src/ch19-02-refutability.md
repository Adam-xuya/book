## 可反驳性：模式是否可能无法匹配

模式有两种形式：可反驳和不可反驳。对于传递的任何可能值都会匹配的模式是_不可反驳的_。一个例子是语句 `let x = 5;` 中的 `x`，因为 `x` 匹配任何内容，因此无法匹配失败。对于某些可能值可能无法匹配的模式是_可反驳的_。一个例子是表达式 `if let Some(x) = a_value` 中的 `Some(x)`，因为如果 `a_value` 变量中的值是 `None` 而不是 `Some`，`Some(x)` 模式将不匹配。

函数参数、`let` 语句和 `for` 循环只能接受不可反驳的模式，因为当值不匹配时，程序无法做任何有意义的事情。`if let` 和 `while let` 表达式以及 `let...else` 语句接受可反驳和不可反驳的模式，但编译器警告不要使用不可反驳的模式，因为根据定义，它们旨在处理可能的失败：条件的功能在于它能够根据成功或失败执行不同操作的能力。

一般来说，你不应该担心可反驳和不可反驳模式之间的区别；但是，你确实需要熟悉可反驳性的概念，以便在错误消息中看到它时能够做出响应。在这些情况下，你将需要更改模式或使用模式的构造，具体取决于代码的预期行为。

让我们看一个示例，说明当我们尝试在 Rust 需要不可反驳模式的地方使用可反驳模式时会发生什么，反之亦然。代码清单19-8显示了一个 `let` 语句，但对于模式，我们指定了 `Some(x)`，这是一个可反驳的模式。正如你可能期望的那样，此代码不会编译。

<Listing number="19-8" caption="尝试使用 `let` 和可反驳模式">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-08/src/main.rs:here}}
```

</Listing>

如果 `some_option_value` 是 `None` 值，它将无法匹配模式 `Some(x)`，这意味着模式是可反驳的。但是，`let` 语句只能接受不可反驳的模式，因为代码无法对 `None` 值做任何有效的事情。在编译时，Rust 会抱怨我们尝试在需要不可反驳模式的地方使用可反驳模式：

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-08/output.txt}}
```

因为我们没有（也不能！）用模式 `Some(x)` 覆盖每个有效值，Rust 正确地产生了编译器错误。

如果我们在需要不可反驳模式的地方有可反驳模式，我们可以通过更改使用模式的代码来修复它：我们可以使用 `let else` 而不是使用 `let`。然后，如果模式不匹配，代码将跳过花括号中的代码，为其提供一种有效继续的方式。代码清单19-9显示了如何修复代码清单19-8中的代码。

<Listing number="19-9" caption="使用 `let...else` 和带有可反驳模式的块而不是 `let`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-09/src/main.rs:here}}
```

</Listing>

我们给代码一个出路！这段代码是完全有效的，尽管这意味着我们无法在不收到警告的情况下使用不可反驳的模式。如果我们给 `let...else` 一个总是匹配的模式，例如 `x`，如代码清单19-10所示，编译器会给出一个警告。

<Listing number="19-10" caption="尝试使用 `let...else` 和不可反驳模式">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-10/src/main.rs:here}}
```

</Listing>

Rust 抱怨使用 `let...else` 和不可反驳模式没有意义：

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-10/output.txt}}
```

因此，匹配分支必须使用可反驳的模式，除了最后一个分支，它应该使用不可反驳的模式匹配任何剩余值。Rust 允许我们在只有一个分支的 `match` 中使用不可反驳的模式，但这种语法并不是特别有用，可以用更简单的 `let` 语句替换。

现在你知道在哪里使用模式以及可反驳和不可反驳模式之间的区别，让我们涵盖所有可以用来创建模式的语法。
