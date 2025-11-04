<!-- Old headings. Do not remove or links may break. -->

<a id="the-match-control-flow-operator"></a>

## `match` 控制流构造

Rust 有一个非常强大的控制流构造，叫做 `match`，它允许你将值与一系列模式进行比较，然后根据匹配的模式执行代码。模式可以由字面值、变量名、通配符和许多其他东西组成；[第 19 章][ch19-00-patterns]<!-- ignore -->涵盖了所有不同类型的模式及其作用。`match` 的强大之处在于模式的表达力以及编译器确认所有可能情况都已处理的事实。

把 `match` 表达式想象成一台硬币分类机：硬币沿着带有各种大小孔的轨道滑下，每枚硬币都会落入它遇到的第一个适合的孔中。同样，值会通过 `match` 中的每个模式，在值“适合”的第一个模式处，值会落入关联的代码块中以便在执行时使用。

说到硬币，让我们用它们作为使用 `match` 的例子！我们可以编写一个函数，接受一枚未知的美国硬币，类似于计数机，确定它是哪种硬币并以美分返回其价值，如代码清单 6-3 所示。

<Listing number="6-3" caption="一个枚举和一个 `match` 表达式，枚举的变体作为其模式">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-03/src/main.rs:here}}
```

</Listing>

让我们分解 `value_in_cents` 函数中的 `match`。首先，我们列出 `match` 关键字，后跟一个表达式，在这种情况下是值 `coin`。这似乎与与 `if` 一起使用的条件表达式非常相似，但有一个很大的区别：使用 `if`，条件需要计算为布尔值，但这里它可以是任何类型。在这个例子中，`coin` 的类型是我们第一行定义的 `Coin` 枚举。

接下来是 `match` 分支。分支有两个部分：一个模式和一些代码。这里的第一个分支有一个模式，即值 `Coin::Penny`，然后是 `=>` 运算符，它分隔模式和要运行的代码。在这种情况下，代码只是值 `1`。每个分支都用逗号与下一个分支分隔。

当 `match` 表达式执行时，它按顺序将结果值与每个分支的模式进行比较。如果模式匹配值，则执行与该模式关联的代码。如果该模式不匹配值，执行继续到下一个分支，就像在硬币分类机中一样。我们可以根据需要拥有任意多的分支：在代码清单 6-3 中，我们的 `match` 有四个分支。

与每个分支关联的代码是一个表达式，匹配分支中表达式的结果值是整个 `match` 表达式返回的值。

如果匹配分支代码很短，我们通常不使用花括号，就像代码清单 6-3 中每个分支只返回一个值一样。如果你想在匹配分支中运行多行代码，必须使用花括号，并且分支后面的逗号是可选的。例如，以下代码在每次使用 `Coin::Penny` 调用该方法时打印“Lucky penny!”，但它仍然返回块的最后一个值 `1`：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-08-match-arm-multiple-lines/src/main.rs:here}}
```

### 绑定值的模式

匹配分支的另一个有用特性是它们可以绑定到匹配模式的值部分。这就是我们如何从枚举变体中提取值。

作为一个例子，让我们更改我们的枚举变体之一以在其中保存数据。从 1999 年到 2008 年，美国铸造了 25 美分硬币，一面印有 50 个州中每个州的不同设计。没有其他硬币有州设计，所以只有 25 美分硬币有这个额外值。我们可以通过将 `Quarter` 变体更改为包含存储在其中的 `UsState` 值来将此信息添加到我们的 `enum` 中，如代码清单 6-4 所示。

<Listing number="6-4" caption="一个 `Coin` 枚举，其中 `Quarter` 变体也保存一个 `UsState` 值">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-04/src/main.rs:here}}
```

</Listing>

让我们想象一个朋友正在尝试收集所有 50 个州的 25 美分硬币。当我们按硬币类型对零钱进行分类时，我们也会说出与每个 25 美分硬币关联的州名称，这样如果它是我们的朋友没有的，他们可以将其添加到他们的收藏中。

在此代码的匹配表达式中，我们将一个名为 `state` 的变量添加到匹配变体 `Coin::Quarter` 的模式中。当 `Coin::Quarter` 匹配时，`state` 变量将绑定到该 25 美分硬币的州值。然后，我们可以在该分支的代码中使用 `state`，就像这样：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-09-variable-in-pattern/src/main.rs:here}}
```

如果我们要调用 `value_in_cents(Coin::Quarter(UsState::Alaska))`，`coin` 将是 `Coin::Quarter(UsState::Alaska)`。当我们将该值与每个匹配分支进行比较时，没有一个匹配，直到我们到达 `Coin::Quarter(state)`。在这一点上，`state` 的绑定将是值 `UsState::Alaska`。然后我们可以在 `println!` 表达式中使用该绑定，从而从 `Quarter` 的 `Coin` 枚举变体中获取内部州值。

<!-- Old headings. Do not remove or links may break. -->

<a id="matching-with-optiont"></a>

### `Option<T>` `match` 模式

在上一节中，我们想要在使用 `Option<T>` 时从 `Some` 情况中获取内部 `T` 值；我们也可以使用 `match` 处理 `Option<T>`，就像我们对 `Coin` 枚举所做的那样！我们不是比较硬币，而是比较 `Option<T>` 的变体，但 `match` 表达式的工作方式保持不变。

假设我们想编写一个接受 `Option<i32>` 的函数，如果里面有值，则将该值加 1。如果里面没有值，该函数应该返回 `None` 值，不尝试执行任何操作。

由于 `match`，这个函数很容易编写，看起来像代码清单 6-5。

<Listing number="6-5" caption="一个在 `Option<i32>` 上使用 `match` 表达式的函数">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:here}}
```

</Listing>

让我们更详细地检查 `plus_one` 的第一次执行。当我们调用 `plus_one(five)` 时，`plus_one` 主体中的变量 `x` 将具有值 `Some(5)`。然后我们将其与每个匹配分支进行比较：

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

`Some(5)` 值不匹配模式 `None`，所以我们继续到下一个分支：

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:second_arm}}
```

`Some(5)` 是否匹配 `Some(i)`？是的！我们有相同的变体。`i` 绑定到 `Some` 中包含的值，所以 `i` 取值 `5`。然后执行匹配分支中的代码，因此我们将 `i` 的值加 1，并创建一个新的 `Some` 值，其中包含我们的总数 `6`。

现在让我们考虑代码清单 6-5 中 `plus_one` 的第二次调用，其中 `x` 是 `None`。我们进入 `match` 并与第一个分支进行比较：

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

它匹配！没有值可以相加，所以程序停止并返回 `=>` 右侧的 `None` 值。因为第一个分支匹配，所以不会比较其他分支。

将 `match` 和枚举结合使用在许多情况下都很有用。你会经常在 Rust 代码中看到这种模式：对枚举进行 `match`，将变量绑定到内部数据，然后基于它执行代码。一开始有点棘手，但一旦你习惯了，你会希望所有语言都有它。它一直是用户的最爱。

### 匹配是穷尽的

我们需要讨论 `match` 的另一个方面：分支的模式必须涵盖所有可能性。考虑这个版本的 `plus_one` 函数，它有一个错误并且不会编译：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/src/main.rs:here}}
```

我们没有处理 `None` 情况，所以这段代码会导致错误。幸运的是，这是 Rust 知道如何捕获的错误。如果我们尝试编译这段代码，我们会得到这个错误：

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/output.txt}}
```

Rust 知道我们没有涵盖所有可能的情况，甚至知道我们忘记了哪个模式！Rust 中的匹配是 _穷尽的_：我们必须穷尽每一个可能性，代码才能有效。特别是在 `Option<T>` 的情况下，当 Rust 阻止我们忘记显式处理 `None` 情况时，它保护我们不会在可能为 null 时假设我们有一个值，从而使前面讨论的十亿美元错误不可能发生。

### 通配模式和 `_` 占位符

使用枚举，我们还可以对几个特定值采取特殊操作，但对所有其他值采取一个默认操作。想象我们正在实现一个游戏，如果你掷骰子得到 3，你的玩家不会移动，而是会得到一顶漂亮的新帽子。如果你掷出 7，你的玩家会失去一顶漂亮的帽子。对于所有其他值，你的玩家会在游戏板上移动该数量的空格。这是一个实现该逻辑的 `match`，掷骰子的结果是硬编码的而不是随机值，所有其他逻辑由没有主体的函数表示，因为实际实现它们超出了此示例的范围：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-15-binding-catchall/src/main.rs:here}}
```

对于前两个分支，模式是字面值 `3` 和 `7`。对于涵盖所有其他可能值的最后一个分支，模式是我们选择命名为 `other` 的变量。为 `other` 分支运行的代码通过将其传递给 `move_player` 函数来使用该变量。

这段代码可以编译，即使我们没有列出 `u8` 可以拥有的所有可能值，因为最后一个模式将匹配所有未明确列出的值。这个通配模式满足了 `match` 必须是穷尽的要求。注意，我们必须将通配分支放在最后，因为模式是按顺序评估的。如果我们早些时候放置了通配分支，其他分支将永远不会运行，所以如果我们 after a catch-all 添加分支，Rust 会警告我们！

Rust 还有一个模式，当我们想要通配但不想 _使用_ 通配模式中的值时可以使用：`_` 是一个匹配任何值且不绑定到该值的特殊模式。这告诉 Rust 我们不会使用该值，所以 Rust 不会警告我们未使用的变量。

让我们再次更改游戏规则：现在，如果你掷出除 3 或 7 之外的任何值，你必须再次掷骰子。我们不再需要使用通配值，所以我们可以将代码更改为使用 `_` 而不是名为 `other` 的变量：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-16-underscore-catchall/src/main.rs:here}}
```

这个例子也满足穷尽性要求，因为我们在最后一个分支中明确忽略了所有其他值；我们没有忘记任何事情。

最后，我们将再次更改游戏规则，这样如果你掷出除 3 或 7 之外的任何值，在你的回合中不会发生其他任何事情。我们可以通过使用单元值（我们在["元组类型"][tuples]<!-- ignore -->部分提到的空元组类型）作为与 `_` 分支一起的代码来表达这一点：

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-17-underscore-unit/src/main.rs:here}}
```

在这里，我们明确告诉 Rust 我们不会使用与早期分支中的模式不匹配的任何其他值，并且在这种情况下我们不想运行任何代码。

关于模式和匹配，我们将在[第 19 章][ch19-00-patterns]<!-- ignore -->中介绍更多内容。现在，我们将继续学习 `if let` 语法，这在 `match` 表达式有点冗长的情况下很有用。

[tuples]: ch03-02-data-types.html#the-tuple-type
[ch19-00-patterns]: ch19-00-patterns.html
