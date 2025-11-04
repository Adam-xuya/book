## 使用 `if let` 和 `let else` 的简洁控制流

`if let` 语法让你将 `if` 和 `let` 组合成一种不太冗长的方式来处理匹配一个模式的值，同时忽略其余的值。考虑代码清单 6-6 中的程序，它在 `config_max` 变量中的 `Option<u8>` 值上匹配，但只想在值是 `Some` 变体时执行代码。

<Listing number="6-6" caption="一个只关心当值是 `Some` 时执行代码的 `match`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-06/src/main.rs:here}}
```

</Listing>

如果值是 `Some`，我们通过在模式中将值绑定到变量 `max` 来打印 `Some` 变体中的值。我们不想对 `None` 值做任何事情。为了满足 `match` 表达式，我们必须在只处理一个变体后添加 `_ => ()`，这是令人讨厌的样板代码。

相反，我们可以使用 `if let` 以更短的方式编写。以下代码的行为与代码清单 6-6 中的 `match` 相同：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-12-if-let/src/main.rs:here}}
```

`if let` 语法接受一个模式和一个由等号分隔的表达式。它的工作方式与 `match` 相同，其中表达式提供给 `match`，模式是它的第一个分支。在这种情况下，模式是 `Some(max)`，`max` 绑定到 `Some` 内部的值。然后，我们可以在 `if let` 块的主体中使用 `max`，就像我们在相应的 `match` 分支中使用 `max` 一样。只有当值匹配模式时，`if let` 块中的代码才会运行。

使用 `if let` 意味着更少的输入、更少的缩进和更少的样板代码。但是，你失去了 `match` 强制执行的穷尽性检查，这确保你不会忘记处理任何情况。在 `match` 和 `if let` 之间选择取决于你在特定情况下正在做什么，以及获得简洁性是否值得失去穷尽性检查的权衡。

换句话说，你可以将 `if let` 视为 `match` 的语法糖，它在值匹配一个模式时运行代码，然后忽略所有其他值。

我们可以在 `if let` 中包含一个 `else`。与 `else` 一起的代码块与与等效的 `if let` 和 `else` 的 `match` 表达式中的 `_` 情况一起的代码块相同。回想一下代码清单 6-4 中的 `Coin` 枚举定义，其中 `Quarter` 变体也保存了一个 `UsState` 值。如果我们想在宣布 25 美分硬币的州的同时统计我们看到的所有非 25 美分硬币，我们可以使用 `match` 表达式来做到这一点，就像这样：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-13-count-and-announce-match/src/main.rs:here}}
```

或者我们可以使用 `if let` 和 `else` 表达式，就像这样：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-14-count-and-announce-if-let-else/src/main.rs:here}}
```

## 使用 `let...else` 保持在“快乐路径”上

常见的模式是当值存在时执行某些计算，否则返回默认值。继续我们带有 `UsState` 值的硬币示例，如果我们想根据 25 美分硬币上的州有多古老来说一些有趣的话，我们可能在 `UsState` 上引入一个方法来检查州的年龄，就像这样：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:state}}
```

然后，我们可能使用 `if let` 来匹配硬币类型，在条件主体中引入一个 `state` 变量，如代码清单 6-7 所示。

<Listing number="6-7" caption="通过在 `if let` 内嵌套条件来检查州是否在 1900 年存在">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:describe}}
```

</Listing>

这完成了工作，但它将工作推入了 `if let` 语句的主体中，如果要完成的工作更复杂，可能很难准确跟踪顶级分支如何关联。我们也可以利用表达式产生值的事实，要么从 `if let` 产生 `state`，要么提前返回，如代码清单 6-8 所示。（你也可以使用 `match` 做类似的事情。）

<Listing number="6-8" caption="使用 `if let` 产生值或提前返回">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-08/src/main.rs:describe}}
```

</Listing>

不过，这本身有点令人讨厌！`if let` 的一个分支产生一个值，另一个分支完全从函数返回。

为了使这个常见模式更容易表达，Rust 有 `let...else`。`let...else` 语法在左侧接受一个模式，在右侧接受一个表达式，与 `if let` 非常相似，但它没有 `if` 分支，只有一个 `else` 分支。如果模式匹配，它将在外部作用域中绑定模式中的值。如果模式 _不_ 匹配，程序将流入 `else` 分支，该分支必须从函数返回。

在代码清单 6-9 中，你可以看到当使用 `let...else` 代替 `if let` 时代码清单 6-8 的样子。

<Listing number="6-9" caption="使用 `let...else` 澄清函数中的流程">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-09/src/main.rs:describe}}
```

</Listing>

注意，这种方式它保持在函数主体中的“快乐路径”上，而不像 `if let` 那样为两个分支有显著不同的控制流。

如果你遇到程序逻辑使用 `match` 表达过于冗长的情况，请记住 `if let` 和 `let...else` 也在你的 Rust 工具箱中。

## 总结

我们现在已经介绍了如何使用枚举来创建可以是枚举值集合之一的自定义类型。我们已经展示了标准库的 `Option<T>` 类型如何帮助你使用类型系统来防止错误。当枚举值中有数据时，你可以使用 `match` 或 `if let` 来提取和使用这些值，具体取决于你需要处理多少种情况。

你的 Rust 程序现在可以使用结构体和枚举来表达你领域中的概念。创建自定义类型在你的 API 中使用可确保类型安全：编译器将确保你的函数只获得每个函数期望的类型的值。

为了向你的用户提供一个组织良好、易于使用且只暴露用户所需内容的 API，现在让我们转向 Rust 的模块。
