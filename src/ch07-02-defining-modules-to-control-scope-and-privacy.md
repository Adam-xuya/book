<!-- Old headings. Do not remove or links may break. -->

<a id="defining-modules-to-control-scope-and-privacy"></a>

## 使用模块控制作用域和隐私

在本节中，我们将讨论模块以及模块系统的其他部分，即 _路径_（允许你命名项）、将路径引入作用域的 `use` 关键字以及使项公开的 `pub` 关键字。我们还将讨论 `as` 关键字、外部包和 glob 操作符。

### 模块速查表

在我们深入了解模块和路径的细节之前，这里提供了一个快速参考，说明模块、路径、`use` 关键字和 `pub` 关键字如何在编译器中工作，以及大多数开发人员如何组织他们的代码。我们将在本章中通过示例介绍这些规则，但这是一个很好的参考点，可以提醒你模块是如何工作的。

- **从 crate 根开始**：编译 crate 时，编译器首先在 crate 根文件（通常是库 crate 的 _src/lib.rs_ 和二进制 crate 的 _src/main.rs_）中查找要编译的代码。
- **声明模块**：在 crate 根文件中，你可以声明新模块；假设你用 `mod garden;` 声明一个"garden"模块。编译器将在以下位置查找模块的代码：
  - 内联，在花括号内，替换 `mod garden` 后面的分号
  - 在文件 _src/garden.rs_ 中
  - 在文件 _src/garden/mod.rs_ 中
- **声明子模块**：在 crate 根以外的任何文件中，你可以声明子模块。例如，你可以在 _src/garden.rs_ 中声明 `mod vegetables;`。编译器将在以父模块命名的目录中的以下位置查找子模块的代码：
  - 内联，直接在 `mod vegetables` 之后，在花括号内而不是分号
  - 在文件 _src/garden/vegetables.rs_ 中
  - 在文件 _src/garden/vegetables/mod.rs_ 中
- **模块中代码的路径**：一旦模块成为 crate 的一部分，只要隐私规则允许，你就可以从同一 crate 的任何其他地方使用路径引用该模块中的代码。例如，花园蔬菜模块中的 `Asparagus` 类型可以在 `crate::garden::vegetables::Asparagus` 找到。
- **私有 vs. 公共**：模块内的代码默认对其父模块是私有的。要使模块公开，请使用 `pub mod` 而不是 `mod` 声明它。要使公共模块内的项也公开，请在它们的声明之前使用 `pub`。
- **`use` 关键字**：在作用域内，`use` 关键字创建项的快捷方式以减少长路径的重复。在任何可以引用 `crate::garden::vegetables::Asparagus` 的作用域中，你可以使用 `use crate::garden::vegetables::Asparagus;` 创建快捷方式，然后你只需要在该作用域中编写 `Asparagus` 即可使用该类型。

这里，我们创建一个名为 `backyard` 的二进制 crate 来说明这些规则。crate 的目录也命名为 _backyard_，包含以下文件和目录：

```text
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

在这种情况下，crate 根文件是 _src/main.rs_，它包含：

<Listing file-name="src/main.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/main.rs}}
```

</Listing>

`pub mod garden;` 行告诉编译器包含它在 _src/garden.rs_ 中找到的代码，即：

<Listing file-name="src/garden.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden.rs}}
```

</Listing>

这里，`pub mod vegetables;` 意味着 _src/garden/vegetables.rs_ 中的代码也被包含。该代码是：

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden/vegetables.rs}}
```

现在让我们深入了解这些规则的细节并演示它们的实际应用！

### 在模块中组织相关代码

_模块_ 让我们在 crate 内组织代码以提高可读性和易于重用。模块还允许我们控制项的 _隐私_，因为模块内的代码默认是私有的。私有项是内部实现细节，不对外部使用。我们可以选择使模块及其中的项公开，这将暴露它们以允许外部代码使用和依赖它们。

作为一个例子，让我们编写一个提供餐厅功能的库 crate。我们将定义函数的签名但保留函数体为空，以专注于代码的组织而不是餐厅的实现。

在餐饮业中，餐厅的某些部分被称为前厅（front of house），其他部分被称为后厅（back of house）。_前厅_ 是顾客所在的地方；这包括接待员安排顾客就座的地方、服务员接受订单和付款的地方以及调酒师制作饮料的地方。_后厅_ 是厨师和厨工在厨房工作的地方、洗碗工清理的地方以及管理人员做行政工作的地方。

要以这种方式构建我们的 crate，我们可以将其函数组织到嵌套模块中。通过运行 `cargo new restaurant --lib` 创建一个名为 `restaurant` 的新库。然后，将代码清单 7-1 中的代码输入到 _src/lib.rs_ 中以定义一些模块和函数签名；这段代码是前厅部分。

<Listing number="7-1" file-name="src/lib.rs" caption="一个 `front_of_house` 模块，包含其他模块，然后包含函数">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-01/src/lib.rs}}
```

</Listing>

我们使用 `mod` 关键字后跟模块名称（在这种情况下是 `front_of_house`）来定义模块。然后模块的主体放在花括号内。在模块内部，我们可以放置其他模块，就像在这种情况下使用 `hosting` 和 `serving` 模块一样。模块还可以保存其他项的定义，例如结构体、枚举、常量、trait，以及如代码清单 7-1 中的函数。

通过使用模块，我们可以将相关定义组合在一起并命名它们为什么相关。使用此代码的程序员可以基于组导航代码，而不必阅读所有定义，从而更容易找到与他们相关的定义。向此代码添加新功能的程序员会知道在哪里放置代码以保持程序的组织性。

早些时候，我们提到 _src/main.rs_ 和 _src/lib.rs_ 被称为 _crate 根_。它们名称的原因是这两个文件中的任何一个的内容在 crate 的模块结构（称为 _模块树_）的根部形成一个名为 `crate` 的模块。

代码清单 7-2 显示了代码清单 7-1 中结构的模块树。

<Listing number="7-2" caption="代码清单 7-1 中代码的模块树">

```text
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

</Listing>

这棵树显示了某些模块如何嵌套在其他模块内；例如，`hosting` 嵌套在 `front_of_house` 内。这棵树还显示某些模块是 _兄弟_，意味着它们在同一个模块中定义；`hosting` 和 `serving` 是在 `front_of_house` 内定义的兄弟。如果模块 A 包含在模块 B 内，我们说模块 A 是模块 B 的 _子模块_，模块 B 是模块 A 的 _父模块_。注意，整个模块树植根于名为 `crate` 的隐式模块下。

模块树可能会让你想起计算机上的文件系统目录树；这是一个非常恰当的类比！就像文件系统中的目录一样，你使用模块来组织代码。就像目录中的文件一样，我们需要一种方法来找到我们的模块。
