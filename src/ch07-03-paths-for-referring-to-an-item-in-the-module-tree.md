## 在模块树中引用项的路径

为了告诉 Rust 在模块树中哪里找到项，我们使用路径，就像在文件系统中导航时使用路径一样。要调用函数，我们需要知道它的路径。

路径可以采用两种形式：

- _绝对路径_ 是从 crate 根开始的完整路径；对于来自外部 crate 的代码，绝对路径以 crate 名称开始，对于来自当前 crate 的代码，它以字面量 `crate` 开始。
- _相对路径_ 从当前模块开始，使用 `self`、`super` 或当前模块中的标识符。

绝对路径和相对路径后跟一个或多个由双冒号（`::`）分隔的标识符。

回到代码清单 7-1，假设我们想调用 `add_to_waitlist` 函数。这等同于问：`add_to_waitlist` 函数的路径是什么？代码清单 7-3 包含代码清单 7-1，但删除了一些模块和函数。

我们将展示从 crate 根中定义的新函数 `eat_at_restaurant` 调用 `add_to_waitlist` 函数的两种方法。这些路径是正确的，但还有另一个问题会阻止此示例按原样编译。我们稍后会解释原因。

`eat_at_restaurant` 函数是我们库 crate 的公共 API 的一部分，因此我们用 `pub` 关键字标记它。在["使用 `pub` 关键字公开路径"][pub]<!-- ignore -->部分，我们将详细介绍 `pub`。

<Listing number="7-3" file-name="src/lib.rs" caption="使用绝对路径和相对路径调用 `add_to_waitlist` 函数">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-03/src/lib.rs}}
```

</Listing>

在 `eat_at_restaurant` 中第一次调用 `add_to_waitlist` 函数时，我们使用绝对路径。`add_to_waitlist` 函数与 `eat_at_restaurant` 定义在同一个 crate 中，这意味着我们可以使用 `crate` 关键字开始绝对路径。然后我们包含每个连续的模块，直到到达 `add_to_waitlist`。你可以想象一个具有相同结构的文件系统：我们将指定路径 `/front_of_house/hosting/add_to_waitlist` 来运行 `add_to_waitlist` 程序；使用 `crate` 名称从 crate 根开始就像在 shell 中使用 `/` 从文件系统根开始一样。

在 `eat_at_restaurant` 中第二次调用 `add_to_waitlist` 时，我们使用相对路径。路径以 `front_of_house` 开始，这是与 `eat_at_restaurant` 在模块树中同一级别定义的模块名称。这里的文件系统等效项将使用路径 `front_of_house/hosting/add_to_waitlist`。以模块名称开始意味着路径是相对的。

选择使用相对路径还是绝对路径是你根据项目做出的决定，这取决于你是否更可能将项定义代码与使用该项的代码分开移动或一起移动。例如，如果我们将 `front_of_house` 模块和 `eat_at_restaurant` 函数移动到名为 `customer_experience` 的模块中，我们需要更新 `add_to_waitlist` 的绝对路径，但相对路径仍然有效。但是，如果我们将 `eat_at_restaurant` 函数单独移动到名为 `dining` 的模块中，`add_to_waitlist` 调用的绝对路径将保持不变，但相对路径需要更新。我们通常更喜欢指定绝对路径，因为我们更可能希望独立地移动代码定义和项调用。

让我们尝试编译代码清单 7-3，看看为什么它还不能编译！我们得到的错误如代码清单 7-4 所示。

<Listing number="7-4" caption="构建代码清单 7-3 中的代码时的编译器错误">

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-03/output.txt}}
```

</Listing>

错误消息说模块 `hosting` 是私有的。换句话说，我们有 `hosting` 模块和 `add_to_waitlist` 函数的正确路径，但 Rust 不会让我们使用它们，因为它无法访问私有部分。在 Rust 中，所有项（函数、方法、结构体、枚举、模块和常量）默认对其父模块是私有的。如果你想让函数或结构体等项私有，你可以将它放在模块中。

父模块中的项不能使用子模块内的私有项，但子模块中的项可以使用其祖先模块中的项。这是因为子模块包装并隐藏其实现细节，但子模块可以看到定义它们的上下文。继续我们的比喻，将隐私规则想象成餐厅的后办公室：那里发生的事情对餐厅顾客来说是私有的，但办公室经理可以看到并操作他们经营的餐厅中的所有事情。

Rust 选择让模块系统以这种方式工作，以便隐藏内部实现细节成为默认行为。这样，你就知道内部代码的哪些部分可以在不破坏外部代码的情况下更改。但是，Rust 确实为你提供了通过使用 `pub` 关键字使项公开来将子模块代码的内部部分暴露给外部祖先模块的选项。

### 使用 `pub` 关键字公开路径

让我们回到代码清单 7-4 中的错误，它告诉我们 `hosting` 模块是私有的。我们希望父模块中的 `eat_at_restaurant` 函数能够访问子模块中的 `add_to_waitlist` 函数，因此我们用 `pub` 关键字标记 `hosting` 模块，如代码清单 7-5 所示。

<Listing number="7-5" file-name="src/lib.rs" caption="将 `hosting` 模块声明为 `pub` 以从 `eat_at_restaurant` 使用它">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-05/src/lib.rs:here}}
```

</Listing>

不幸的是，代码清单 7-5 中的代码仍然会导致编译器错误，如代码清单 7-6 所示。

<Listing number="7-6" caption="构建代码清单 7-5 中的代码时的编译器错误">

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-05/output.txt}}
```

</Listing>

发生了什么？在 `mod hosting` 前面添加 `pub` 关键字使模块公开。有了这个更改，如果我们能够访问 `front_of_house`，我们就可以访问 `hosting`。但是 `hosting` 的 _内容_ 仍然是私有的；使模块公开并不会使其内容公开。模块上的 `pub` 关键字只允许其祖先模块中的代码引用它，而不能访问其内部代码。因为模块是容器，仅使模块公开我们无法做太多事情；我们需要进一步选择使模块内的一个或多个项也公开。

代码清单 7-6 中的错误说 `add_to_waitlist` 函数是私有的。隐私规则适用于结构体、枚举、函数和方法以及模块。

让我们也在 `add_to_waitlist` 函数的定义之前添加 `pub` 关键字使其公开，如代码清单 7-7 所示。

<Listing number="7-7" file-name="src/lib.rs" caption="将 `pub` 关键字添加到 `mod hosting` 和 `fn add_to_waitlist` 让我们可以从 `eat_at_restaurant` 调用该函数。">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-07/src/lib.rs:here}}
```

</Listing>

现在代码将编译！要了解为什么添加 `pub` 关键字允许我们根据隐私规则在 `eat_at_restaurant` 中使用这些路径，让我们看看绝对路径和相对路径。

在绝对路径中，我们从 `crate` 开始，这是我们 crate 模块树的根。`front_of_house` 模块在 crate 根中定义。虽然 `front_of_house` 不是公开的，但因为 `eat_at_restaurant` 函数与 `front_of_house` 定义在同一个模块中（即 `eat_at_restaurant` 和 `front_of_house` 是兄弟），我们可以从 `eat_at_restaurant` 引用 `front_of_house`。接下来是用 `pub` 标记的 `hosting` 模块。我们可以访问 `hosting` 的父模块，所以我们可以访问 `hosting`。最后，`add_to_waitlist` 函数用 `pub` 标记，我们可以访问其父模块，所以这个函数调用有效！

在相对路径中，逻辑与绝对路径相同，除了第一步：不是从 crate 根开始，路径从 `front_of_house` 开始。`front_of_house` 模块与 `eat_at_restaurant` 定义在同一个模块内，所以从定义 `eat_at_restaurant` 的模块开始的相对路径有效。然后，因为 `hosting` 和 `add_to_waitlist` 用 `pub` 标记，路径的其余部分有效，这个函数调用是有效的！

如果你计划共享你的库 crate 以便其他项目可以使用你的代码，你的公共 API 是你与 crate 用户的合同，决定了他们如何与你的代码交互。管理公共 API 的更改以使人们更容易依赖你的 crate 有很多考虑因素。这些考虑超出了本书的范围；如果你对这个主题感兴趣，请参阅 [Rust API 指南][api-guidelines]。

> #### 包含二进制和库的包的最佳实践
>
> 我们提到包可以同时包含 _src/main.rs_ 二进制 crate 根和 _src/lib.rs_ 库 crate 根，并且两个 crate 默认都具有包名称。通常，包含库和二进制 crate 的这种模式的包在二进制 crate 中只有足够的代码来启动一个可执行文件，该可执行文件调用库 crate 中定义的代码。这使得其他项目可以从包提供的最大功能中受益，因为库 crate 的代码可以共享。
>
> 模块树应该在 _src/lib.rs_ 中定义。然后，任何公共项都可以通过在二进制 crate 中以包名称开始路径来使用。二进制 crate 成为库 crate 的用户，就像完全外部的 crate 会使用库 crate 一样：它只能使用公共 API。这有助于你设计一个好的 API；你不仅是作者，还是客户！
>
> 在[第 12 章][ch12]<!-- ignore -->中，我们将用一个命令行程序演示这种组织实践，该程序将包含二进制 crate 和库 crate。

### 使用 `super` 开始相对路径

我们可以通过在路径开头使用 `super` 来构造从父模块开始的相对路径，而不是从当前模块或 crate 根开始。这就像用 `..` 语法开始文件系统路径，这意味着转到父目录。使用 `super` 允许我们引用我们知道在父模块中的项，当模块与父模块密切相关但父模块可能有一天移动到模块树的其他地方时，这可以使重新排列模块树更容易。

考虑代码清单 7-8 中的代码，它模拟了厨师修复错误订单并亲自将其带给顾客的情况。在 `back_of_house` 模块中定义的 `fix_incorrect_order` 函数通过指定以 `super` 开始的 `deliver_order` 路径来调用父模块中定义的 `deliver_order` 函数。

<Listing number="7-8" file-name="src/lib.rs" caption="使用以 `super` 开始的相对路径调用函数">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-08/src/lib.rs}}
```

</Listing>

`fix_incorrect_order` 函数在 `back_of_house` 模块中，所以我们可以使用 `super` 转到 `back_of_house` 的父模块，在这种情况下是 `crate`，即根。从那里，我们查找 `deliver_order` 并找到它。成功！我们认为 `back_of_house` 模块和 `deliver_order` 函数可能会保持彼此相同的关系，如果我们要重组 crate 的模块树，它们会一起移动。因此，我们使用了 `super`，这样如果这段代码移动到不同的模块，我们将来需要更新代码的地方会更少。

### 使结构体和枚举公开

我们也可以使用 `pub` 将结构体和枚举指定为公开的，但 `pub` 与结构体和枚举的用法有一些额外的细节。如果我们在结构体定义之前使用 `pub`，我们使结构体公开，但结构体的字段仍然是私有的。我们可以逐个字段决定是否使每个字段公开。在代码清单 7-9 中，我们定义了一个公共的 `back_of_house::Breakfast` 结构体，它有一个公共的 `toast` 字段但有一个私有的 `seasonal_fruit` 字段。这模拟了餐厅中的情况，顾客可以选择随餐提供的面包类型，但厨师根据当季和库存决定哪种水果配餐。可用水果变化很快，所以顾客无法选择水果，甚至无法看到他们将得到哪种水果。

<Listing number="7-9" file-name="src/lib.rs" caption="一个有一些公共字段和一些私有字段的结构体">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-09/src/lib.rs}}
```

</Listing>

因为 `back_of_house::Breakfast` 结构体中的 `toast` 字段是公共的，在 `eat_at_restaurant` 中我们可以使用点符号写入和读取 `toast` 字段。注意，我们不能在 `eat_at_restaurant` 中使用 `seasonal_fruit` 字段，因为 `seasonal_fruit` 是私有的。尝试取消注释修改 `seasonal_fruit` 字段值的行，看看你会得到什么错误！

另外，注意因为 `back_of_house::Breakfast` 有一个私有字段，结构体需要提供一个公共的关联函数来构造 `Breakfast` 的实例（我们在这里将其命名为 `summer`）。如果 `Breakfast` 没有这样的函数，我们就无法在 `eat_at_restaurant` 中创建 `Breakfast` 的实例，因为我们无法在 `eat_at_restaurant` 中设置私有 `seasonal_fruit` 字段的值。

相比之下，如果我们使枚举公开，它的所有变体都会公开。我们只需要在 `enum` 关键字之前使用 `pub`，如代码清单 7-10 所示。

<Listing number="7-10" file-name="src/lib.rs" caption="将枚举指定为公开使其所有变体都公开。">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-10/src/lib.rs}}
```

</Listing>

因为我们使 `Appetizer` 枚举公开，我们可以在 `eat_at_restaurant` 中使用 `Soup` 和 `Salad` 变体。

枚举除非其变体公开，否则不太有用；每次都必须用 `pub` 注释所有枚举变体会很烦人，所以枚举变体的默认值是公开的。结构体通常在其字段不公开的情况下也很有用，所以结构体字段遵循一切默认私有的通用规则，除非用 `pub` 注释。

还有一个涉及 `pub` 的情况我们没有涵盖，那就是我们最后的模块系统功能：`use` 关键字。我们将首先单独介绍 `use`，然后展示如何组合 `pub` 和 `use`。

[pub]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[api-guidelines]: https://rust-lang.github.io/api-guidelines/
[ch12]: ch12-00-an-io-project.html
