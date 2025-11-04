## 模式语法

在本节中，我们收集所有在模式中有效的语法，并讨论为什么以及何时你可能想要使用每一种。

### 匹配字面量

如你在第6章中看到的，你可以直接匹配模式与字面量。以下代码给出了一些示例：

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-01-literals/src/main.rs:here}}
```

此代码打印 `one`，因为 `x` 中的值是 `1`。当你希望代码在获得特定具体值时采取行动时，此语法很有用。

### 匹配命名变量

命名变量是匹配任何值的不可反驳模式，我们在本书中多次使用它们。但是，当你在 `match`、`if let` 或 `while let` 表达式中使用命名变量时，有一个复杂性。因为这些类型的表达式中的每一种都开始一个新的作用域，作为这些表达式内部模式的一部分声明的变量将遮蔽具有相同名称的外部变量，就像所有变量的情况一样。在代码清单19-11中，我们声明一个名为 `x` 的变量，值为 `Some(5)`，以及一个名为 `y` 的变量，值为 `10`。然后我们在值 `x` 上创建一个 `match` 表达式。查看匹配分支中的模式和末尾的 `println!`，并尝试在运行此代码或进一步阅读之前弄清楚代码将打印什么。

<Listing number="19-11" file-name="src/main.rs" caption="一个 `match` 表达式，其中一个分支引入一个新变量，遮蔽现有变量 `y`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-11/src/main.rs:here}}
```

</Listing>

让我们看看当 `match` 表达式运行时会发生什么。第一个匹配分支中的模式不匹配 `x` 的定义值，所以代码继续。

第二个匹配分支中的模式引入一个名为 `y` 的新变量，它将匹配 `Some` 值内的任何值。因为我们在 `match` 表达式内的新作用域中，这是一个新的 `y` 变量，不是我们在开始时声明的值为 `10` 的 `y`。这个新的 `y` 绑定将匹配 `Some` 内的任何值，这就是我们在 `x` 中的内容。因此，这个新的 `y` 绑定到 `x` 中 `Some` 的内部值。该值是 `5`，所以该分支的表达式执行并打印 `Matched, y = 5`。

如果 `x` 是 `None` 值而不是 `Some(5)`，前两个分支中的模式将不匹配，所以值将匹配下划线。我们没有在下划线分支的模式中引入 `x` 变量，所以表达式中的 `x` 仍然是未被遮蔽的外部 `x`。在这种假设情况下，`match` 将打印 `Default case, x = None`。

当 `match` 表达式完成时，其作用域结束，内部 `y` 的作用域也结束。最后的 `println!` 产生 `at the end: x = Some(5), y = 10`。

要创建一个比较外部 `x` 和 `y` 值的 `match` 表达式，而不是引入遮蔽现有 `y` 变量的新变量，我们需要使用匹配守卫条件。我们将在后面的[“使用匹配守卫添加条件”](#adding-conditionals-with-match-guards)<!-- ignore -->部分讨论匹配守卫。

<!-- Old headings. Do not remove or links may break. -->
<a id="multiple-patterns"></a>

### 匹配多个模式

在 `match` 表达式中，你可以使用 `|` 语法匹配多个模式，这是模式_或_运算符。例如，在以下代码中，我们将 `x` 的值与匹配分支匹配，第一个分支有一个_或_选项，这意味着如果 `x` 的值匹配该分支中的任一值，该分支的代码将运行：


```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-02-multiple-patterns/src/main.rs:here}}
```

此代码打印 `one or two`。

### 使用 `..=` 匹配值范围

`..=` 语法允许我们匹配包含值范围。在以下代码中，当模式匹配给定范围内的任何值时，该分支将执行：

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-03-ranges/src/main.rs:here}}
```

如果 `x` 是 `1`、`2`、`3`、`4` 或 `5`，第一个分支将匹配。此语法对于多个匹配值比使用 `|` 运算符表达相同想法更方便；如果我们要使用 `|`，我们必须指定 `1 | 2 | 3 | 4 | 5`。指定范围要短得多，特别是如果我们想要匹配，比如 1 到 1,000 之间的任何数字！

编译器在编译时检查范围是否为空，并且因为 Rust 可以判断范围是否为空或不为空的唯一类型是 `char` 和数值，范围只允许使用数值或 `char` 值。

这是一个使用 `char` 值范围的示例：

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-04-ranges-of-char/src/main.rs:here}}
```

Rust 可以判断 `'c'` 在第一个模式的范围内并打印 `early ASCII letter`。

### 解构以拆分值

我们也可以使用模式来解构结构体、枚举和元组，以使用这些值的不同部分。让我们逐个值进行。

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-structs"></a>

#### 结构体

代码清单19-12显示了一个 `Point` 结构体，有两个字段 `x` 和 `y`，我们可以使用带有 `let` 语句的模式来拆分它。

<Listing number="19-12" file-name="src/main.rs" caption="将结构体的字段解构为单独的变量">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-12/src/main.rs}}
```

</Listing>

此代码创建变量 `a` 和 `b`，它们匹配 `p` 结构体的 `x` 和 `y` 字段的值。此示例显示模式中变量的名称不必匹配结构体的字段名称。但是，将变量名称匹配到字段名称是常见的，以便更容易记住哪些变量来自哪些字段。由于这种常见用法，并且因为编写 `let Point { x: x, y: y } = p;` 包含大量重复，Rust 为匹配结构体字段的模式提供了简写：你只需要列出结构体字段的名称，从模式创建的变量将具有相同的名称。代码清单19-13的行为与代码清单19-12中的代码相同，但在 `let` 模式中创建的变量是 `x` 和 `y` 而不是 `a` 和 `b`。

<Listing number="19-13" file-name="src/main.rs" caption="使用结构体字段简写解构结构体字段">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-13/src/main.rs}}
```

</Listing>

此代码创建变量 `x` 和 `y`，它们匹配 `p` 变量的 `x` 和 `y` 字段。结果是变量 `x` 和 `y` 包含来自 `p` 结构体的值。

我们也可以使用字面值作为结构体模式的一部分进行解构，而不是为所有字段创建变量。这样做允许我们测试某些字段的特定值，同时创建变量来解构其他字段。

在代码清单19-14中，我们有一个 `match` 表达式，它将 `Point` 值分为三种情况：直接位于 `x` 轴上的点（当 `y = 0` 时为真）、位于 `y` 轴上的点（`x = 0`）或不在任一轴上。

<Listing number="19-14" file-name="src/main.rs" caption="在一个模式中解构和匹配字面值">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-14/src/main.rs:here}}
```

</Listing>

第一个分支将通过指定 `y` 字段在其值匹配字面量 `0` 时匹配来匹配位于 `x` 轴上的任何点。模式仍然创建一个 `x` 变量，我们可以在该分支的代码中使用。

类似地，第二个分支通过指定 `x` 字段在其值为 `0` 时匹配来匹配 `y` 轴上的任何点，并为 `y` 字段的值创建变量 `y`。第三个分支不指定任何字面量，所以它匹配任何其他 `Point` 并为 `x` 和 `y` 字段创建变量。

在此示例中，值 `p` 通过 `x` 包含 `0` 匹配第二个分支，所以此代码将打印 `On the y axis at 7`。

请记住，`match` 表达式在找到第一个匹配模式后停止检查分支，所以即使 `Point { x: 0, y: 0}` 在 `x` 轴和 `y` 轴上，此代码也只会打印 `On the x axis at 0`。

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-enums"></a>

#### 枚举

我们在本书中解构过枚举（例如，第6章中的代码清单6-5），但我们还没有明确讨论解构枚举的模式对应于枚举内存储数据的方式。例如，在代码清单19-15中，我们使用代码清单6-2中的 `Message` 枚举，并编写一个 `match`，其模式将解构每个内部值。

<Listing number="19-15" file-name="src/main.rs" caption="解构保存不同类型值的枚举变体">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-15/src/main.rs}}
```

</Listing>

此代码将打印 `Change color to red 0, green 160, and blue 255`。尝试更改 `msg` 的值以查看其他分支的代码运行。

对于没有任何数据的枚举变体，如 `Message::Quit`，我们无法进一步解构值。我们只能匹配字面量 `Message::Quit` 值，并且该模式中没有变量。

对于类似结构体的枚举变体，如 `Message::Move`，我们可以使用类似于我们指定匹配结构体的模式。在变体名称之后，我们放置花括号，然后列出带变量的字段，以便我们拆分各个部分以在该分支的代码中使用。这里我们使用与代码清单19-13中相同的简写形式。

对于类似元组的枚举变体，如保存一个元素元组的 `Message::Write` 和保存三个元素元组的 `Message::ChangeColor`，模式类似于我们指定匹配元组的模式。模式中变量的数量必须与我们匹配的变体中的元素数量匹配。

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-nested-structs-and-enums"></a>

#### 嵌套结构体和枚举

到目前为止，我们的示例都只是匹配一层深度的结构体或枚举，但匹配也可以处理嵌套项！例如，我们可以重构代码清单19-15中的代码以支持 `ChangeColor` 消息中的 RGB 和 HSV 颜色，如代码清单19-16所示。

<Listing number="19-16" caption="匹配嵌套枚举">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-16/src/main.rs}}
```

</Listing>

`match` 表达式中第一个分支的模式匹配包含 `Color::Rgb` 变体的 `Message::ChangeColor` 枚举变体；然后，模式绑定到三个内部 `i32` 值。第二个分支的模式也匹配 `Message::ChangeColor` 枚举变体，但内部枚举匹配 `Color::Hsv` 而不是。我们可以在一个 `match` 表达式中指定这些复杂条件，即使涉及两个枚举。

<!-- Old headings. Do not remove or links may break. -->

<a id="destructuring-structs-and-tuples"></a>

#### 结构体和元组

我们可以以更复杂的方式混合、匹配和嵌套解构模式。以下示例显示了一个复杂的解构，我们在元组内嵌套结构体和元组，并解构所有原始值：

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-05-destructuring-structs-and-tuples/src/main.rs:here}}
```

此代码让我们将复杂类型分解为其组成部分，以便我们可以单独使用我们感兴趣的值。

使用模式进行解构是一种方便的方式来单独使用值的各个部分，例如结构体中每个字段的值。

### 忽略模式中的值

你已经看到有时忽略模式中的值很有用，例如在 `match` 的最后一个分支中，以获得一个实际上不做任何事情但确实考虑了所有剩余可能值的捕获所有内容。有几种方法可以忽略模式中的整个值或值的部分：使用 `_` 模式（你已经看到过），在另一个模式中使用 `_` 模式，使用以下划线开头的名称，或使用 `..` 忽略值的剩余部分。让我们探索如何以及为什么使用这些模式中的每一种。

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-an-entire-value-with-_"></a>

#### 使用 `_` 忽略整个值

我们已经使用下划线作为通配符模式，它将匹配任何值但不绑定到值。这在 `match` 表达式的最后一个分支中特别有用，但我们也可以在任何模式中使用它，包括函数参数，如代码清单19-17所示。

<Listing number="19-17" file-name="src/main.rs" caption="在函数签名中使用 `_`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-17/src/main.rs}}
```

</Listing>

此代码将完全忽略作为第一个参数传递的值 `3`，并将打印 `This code only uses the y parameter: 4`。

在大多数情况下，当你不再需要特定函数参数时，你会更改签名，使其不包括未使用的参数。忽略函数参数在以下情况下特别有用：例如，当你实现 trait 时需要特定类型签名，但实现中的函数主体不需要其中一个参数。然后你可以避免收到关于未使用函数参数的编译器警告，就像你使用名称时一样。

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-parts-of-a-value-with-a-nested-_"></a>

#### 使用嵌套 `_` 忽略值的部分

我们也可以在另一个模式内使用 `_` 来仅忽略值的部分，例如，当我们只想测试值的部分但在我们要运行的相应代码中对其他部分没有用处时。代码清单19-18显示了负责管理设置值的代码。业务要求是用户不应该被允许覆盖设置的现有自定义，但如果当前未设置，可以取消设置并给它一个值。

<Listing number="19-18" caption="当我们不需要使用 `Some` 内的值时，在匹配 `Some` 变体的模式中使用下划线">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-18/src/main.rs:here}}
```

</Listing>

此代码将打印 `Can't overwrite an existing customized value`，然后打印 `setting is Some(5)`。在第一个匹配分支中，我们不需要匹配或使用任一 `Some` 变体中的值，但我们确实需要测试 `setting_value` 和 `new_setting_value` 都是 `Some` 变体的情况。在这种情况下，我们打印不更改 `setting_value` 的原因，并且它不会更改。

在所有其他情况（如果 `setting_value` 或 `new_setting_value` 是 `None`）中，由第二个分支中的 `_` 模式表示，我们希望允许 `new_setting_value` 成为 `setting_value`。

我们也可以在一个模式内的多个位置使用下划线来忽略特定值。代码清单19-19显示了忽略五个项目元组中第二个和第四个值的示例。

<Listing number="19-19" caption="忽略元组的多个部分">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-19/src/main.rs}}
```

</Listing>

此代码将打印 `Some numbers: 2, 8, 32`，值 `4` 和 `16` 将被忽略。

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-an-unused-variable-by-starting-its-name-with-_"></a>

#### 通过以下划线开头其名称来忽略未使用的变量

如果你创建一个变量但不在任何地方使用它，Rust 通常会发出警告，因为未使用的变量可能是错误。但是，有时能够创建一个你尚未使用的变量很有用，例如当你在原型设计或刚刚开始项目时。在这种情况下，你可以通过以下划线开头变量名称来告诉 Rust 不要警告你有关未使用的变量。在代码清单19-20中，我们创建了两个未使用的变量，但当我们编译此代码时，我们应该只收到关于其中一个的警告。

<Listing number="19-20" file-name="src/main.rs" caption="以下划线开头变量名称以避免收到未使用变量警告">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-20/src/main.rs}}
```

</Listing>

在这里，我们收到关于不使用变量 `y` 的警告，但我们没有收到关于不使用 `_x` 的警告。

请注意，仅使用 `_` 和使用以下划线开头的名称之间有细微差别。语法 `_x` 仍然将值绑定到变量，而 `_` 根本不绑定。为了显示这种区别很重要的情况，代码清单19-21将为我们提供一个错误。

<Listing number="19-21" caption="以下划线开头的未使用变量仍然绑定值，这可能会取得值的所有权。">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-21/src/main.rs:here}}
```

</Listing>

我们会收到错误，因为 `s` 值仍将被移动到 `_s`，这阻止我们再次使用 `s`。但是，单独使用下划线永远不会绑定到值。代码清单19-22将编译而没有任何错误，因为 `s` 不会被移动到 `_`。

<Listing number="19-22" caption="使用下划线不会绑定值。">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-22/src/main.rs:here}}
```

</Listing>

此代码工作正常，因为我们永远不会将 `s` 绑定到任何东西；它不会被移动。

<a id="ignoring-remaining-parts-of-a-value-with-"></a>

#### 使用 `..` 忽略值的剩余部分

对于具有许多部分的值，我们可以使用 `..` 语法来使用特定部分并忽略其余部分，避免需要为每个忽略的值列出下划线。`..` 模式忽略我们在模式的其余部分未明确匹配的值的任何部分。在代码清单19-23中，我们有一个 `Point` 结构体，它在三维空间中保存坐标。在 `match` 表达式中，我们只想操作 `x` 坐标并忽略 `y` 和 `z` 字段中的值。

<Listing number="19-23" caption="通过使用 `..` 忽略 `Point` 的所有字段，除了 `x`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-23/src/main.rs:here}}
```

</Listing>

我们列出 `x` 值，然后只包含 `..` 模式。这比必须列出 `y: _` 和 `z: _` 更快，特别是当我们在只有一两个字段相关的情况下处理具有许多字段的结构体时。

语法 `..` 将扩展到它需要的尽可能多的值。代码清单19-24显示了如何将 `..` 与元组一起使用。

<Listing number="19-24" file-name="src/main.rs" caption="仅匹配元组中的第一个和最后一个值，忽略所有其他值">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-24/src/main.rs}}
```

</Listing>

在此代码中，第一个和最后一个值用 `first` 和 `last` 匹配。`..` 将匹配并忽略中间的所有内容。

但是，使用 `..` 必须是明确的。如果不清楚哪些值用于匹配，哪些应该被忽略，Rust 会给我们一个错误。代码清单19-25显示了以模糊方式使用 `..` 的示例，因此它不会编译。

<Listing number="19-25" file-name="src/main.rs" caption="尝试以模糊方式使用 `..`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-25/src/main.rs}}
```

</Listing>

当我们编译此示例时，我们得到此错误：

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-25/output.txt}}
```

Rust 无法确定在匹配值为 `second` 之前要忽略元组中的多少个值，以及之后要忽略多少个进一步的值。此代码可能意味着我们想要忽略 `2`，将 `second` 绑定到 `4`，然后忽略 `8`、`16` 和 `32`；或者我们想要忽略 `2` 和 `4`，将 `second` 绑定到 `8`，然后忽略 `16` 和 `32`；等等。变量名 `second` 对 Rust 没有任何特殊含义，所以我们得到编译器错误，因为像这样在两个地方使用 `..` 是模糊的。

<!-- Old headings. Do not remove or links may break. -->

<a id="extra-conditionals-with-match-guards"></a>

### 使用匹配守卫添加条件

_匹配守卫_是额外的 `if` 条件，在 `match` 分支中的模式之后指定，也必须匹配才能选择该分支。匹配守卫对于表达比单独模式允许的更复杂的想法很有用。但请注意，它们仅在 `match` 表达式中可用，不在 `if let` 或 `while let` 表达式中可用。

条件可以使用模式中创建的变量。代码清单19-26显示了一个 `match`，其中第一个分支具有模式 `Some(x)` 并且还具有匹配守卫 `if x % 2 == 0`（如果数字是偶数，则为 `true`）。

<Listing number="19-26" caption="向模式添加匹配守卫">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-26/src/main.rs:here}}
```

</Listing>

此示例将打印 `The number 4 is even`。当 `num` 与第一个分支中的模式进行比较时，它匹配，因为 `Some(4)` 匹配 `Some(x)`。然后，匹配守卫检查 `x` 除以 2 的余数是否等于 0，因为它是，所以选择第一个分支。

如果 `num` 是 `Some(5)` 而不是，第一个分支中的匹配守卫将是 `false`，因为 5 除以 2 的余数是 1，不等于 0。Rust 然后会转到第二个分支，它将匹配，因为第二个分支没有匹配守卫，因此匹配任何 `Some` 变体。

无法在模式内表达 `if x % 2 == 0` 条件，所以匹配守卫使我们能够表达此逻辑。这种额外表达能力的缺点是，当涉及匹配守卫表达式时，编译器不会尝试检查穷尽性。

在讨论代码清单19-11时，我们提到我们可以使用匹配守卫来解决我们的模式遮蔽问题。回想一下，我们在 `match` 表达式内的模式中创建了一个新变量，而不是使用 `match` 外部的变量。该新变量意味着我们无法针对外部变量的值进行测试。代码清单19-27显示了如何使用匹配守卫来修复此问题。

<Listing number="19-27" file-name="src/main.rs" caption="使用匹配守卫测试与外部变量的相等性">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-27/src/main.rs}}
```

</Listing>

此代码现在将打印 `Default case, x = Some(5)`。第二个匹配分支中的模式不会引入新变量 `y`，这会遮蔽外部 `y`，这意味着我们可以在匹配守卫中使用外部 `y`。而不是将模式指定为 `Some(y)`，这会遮蔽外部 `y`，我们指定 `Some(n)`。这创建了一个新变量 `n`，它不会遮蔽任何东西，因为 `match` 外部没有 `n` 变量。

匹配守卫 `if n == y` 不是模式，因此不会引入新变量。这个 `y` _是_外部 `y` 而不是遮蔽它的新 `y`，我们可以通过将 `n` 与 `y` 进行比较来查找与外部 `y` 具有相同值的值。

你也可以在匹配守卫中使用_或_运算符 `|` 来指定多个模式；匹配守卫条件将应用于所有模式。代码清单19-28显示了将使用 `|` 的模式与匹配守卫组合时的优先级。此示例的重要部分是 `if y` 匹配守卫适用于 `4`、`5`_和_`6`，尽管看起来 `if y` 只适用于 `6`。

<Listing number="19-28" caption="将多个模式与匹配守卫组合">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-28/src/main.rs:here}}
```

</Listing>

匹配条件声明，只有当 `x` 的值等于 `4`、`5` 或 `6`_并且_如果 `y` 是 `true` 时，该分支才匹配。当此代码运行时，第一个分支的模式匹配，因为 `x` 是 `4`，但匹配守卫 `if y` 是 `false`，所以不选择第一个分支。代码移动到第二个分支，它确实匹配，此程序打印 `no`。原因是 `if` 条件适用于整个模式 `4 | 5 | 6`，而不仅仅是最后一个值 `6`。换句话说，匹配守卫相对于模式的优先级行为如下：

```text
(4 | 5 | 6) if y => ...
```

而不是这样：

```text
4 | 5 | (6 if y) => ...
```

运行代码后，优先级行为很明显：如果匹配守卫仅应用于使用 `|` 运算符指定的值列表中的最终值，该分支将匹配，程序将打印 `yes`。

<!-- Old headings. Do not remove or links may break. -->

<a id="-bindings"></a>

### 使用 `@` 绑定

_at_ 运算符 `@` 让我们创建一个变量，该变量在我们测试该值以进行模式匹配的同时保存值。在代码清单19-29中，我们想要测试 `Message::Hello` `id` 字段在范围 `3..=7` 内。我们还想将值绑定到变量 `id`，以便我们可以在与该分支关联的代码中使用它。

<Listing number="19-29" caption="使用 `@` 在模式中绑定值，同时测试它">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-29/src/main.rs:here}}
```

</Listing>

此示例将打印 `Found an id in range: 5`。通过在范围 `3..=7` 之前指定 `id @`，我们在名为 `id` 的变量中捕获匹配范围的任何值，同时测试该值是否匹配范围模式。

在第二个分支中，我们只在模式中指定了一个范围，与该分支关联的代码没有包含 `id` 字段实际值的变量。`id` 字段的值可能是 10、11 或 12，但与该模式一起使用的代码不知道它是哪一个。模式代码无法使用来自 `id` 字段的值，因为我们没有将 `id` 值保存在变量中。

在最后一个分支中，我们指定了一个没有范围的变量，我们确实在名为 `id` 的变量中具有可用于分支代码的值。原因是我们使用了结构体字段简写语法。但在此分支中，我们没有对 `id` 字段中的值应用任何测试，就像我们对前两个分支所做的那样：任何值都会匹配此模式。

使用 `@` 让我们在一个模式内测试值并将其保存在变量中。

## 总结

Rust 的模式在区分不同类型的数据方面非常有用。当在 `match` 表达式中使用时，Rust 确保你的模式覆盖每个可能的值，否则你的程序不会编译。`let` 语句和函数参数中的模式使这些构造更有用，能够将值解构为更小的部分并将这些部分分配给变量。我们可以创建简单或复杂的模式以满足我们的需求。

接下来，对于本书的倒数第二章，我们将查看 Rust 各种功能的一些高级方面。
