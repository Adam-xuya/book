## 使用 `use` 关键字将路径引入作用域

必须写出调用函数的路径可能感觉不方便和重复。在代码清单 7-7 中，无论我们选择 `add_to_waitlist` 函数的绝对路径还是相对路径，每次我们想调用 `add_to_waitlist` 时都必须指定 `front_of_house` 和 `hosting`。幸运的是，有一种方法可以简化这个过程：我们可以使用 `use` 关键字创建路径的快捷方式一次，然后在作用域的其他地方使用更短的名称。

在代码清单 7-11 中，我们将 `crate::front_of_house::hosting` 模块引入 `eat_at_restaurant` 函数的作用域，这样我们只需要指定 `hosting::add_to_waitlist` 即可在 `eat_at_restaurant` 中调用 `add_to_waitlist` 函数。

<Listing number="7-11" file-name="src/lib.rs" caption="使用 `use` 将模块引入作用域">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-11/src/lib.rs}}
```

</Listing>

在作用域中添加 `use` 和路径类似于在文件系统中创建符号链接。通过在 crate 根中添加 `use crate::front_of_house::hosting`，`hosting` 现在是该作用域中的有效名称，就像 `hosting` 模块在 crate 根中定义一样。使用 `use` 引入作用域的路径也会检查隐私，就像任何其他路径一样。

注意，`use` 只在其发生的特定作用域中创建快捷方式。代码清单 7-12 将 `eat_at_restaurant` 函数移到一个名为 `customer` 的新子模块中，这成为与 `use` 语句不同的作用域，所以函数体不会编译。

<Listing number="7-12" file-name="src/lib.rs" caption="`use` 语句仅适用于它所在的作用域。">

```rust,noplayground,test_harness,does_not_compile,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-12/src/lib.rs}}
```

</Listing>

编译器错误显示快捷方式不再适用于 `customer` 模块内：

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-12/output.txt}}
```

注意还有一个警告，`use` 在其作用域中不再使用！要修复此问题，也将 `use` 移动到 `customer` 模块内，或者在子 `customer` 模块中使用 `super::hosting` 引用父模块中的快捷方式。

### 创建惯用的 `use` 路径

在代码清单 7-11 中，你可能想知道为什么我们指定 `use crate::front_of_house::hosting`，然后在 `eat_at_restaurant` 中调用 `hosting::add_to_waitlist`，而不是将 `use` 路径一直指定到 `add_to_waitlist` 函数以实现相同的结果，如代码清单 7-13 所示。

<Listing number="7-13" file-name="src/lib.rs" caption="使用 `use` 将 `add_to_waitlist` 函数引入作用域，这是非惯用的">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-13/src/lib.rs}}
```

</Listing>

虽然代码清单 7-11 和代码清单 7-13 完成了相同的任务，但代码清单 7-11 是使用 `use` 将函数引入作用域的惯用方式。使用 `use` 将函数的父模块引入作用域意味着我们必须在调用函数时指定父模块。在调用函数时指定父模块清楚地表明该函数不是本地定义的，同时仍然最小化完整路径的重复。代码清单 7-13 中的代码不清楚 `add_to_waitlist` 定义在哪里。

另一方面，当使用 `use` 引入结构体、枚举和其他项时，指定完整路径是惯用的。代码清单 7-14 显示了将标准库的 `HashMap` 结构体引入二进制 crate 作用域的惯用方式。

<Listing number="7-14" file-name="src/main.rs" caption="以惯用方式将 `HashMap` 引入作用域">

```rust
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-14/src/main.rs}}
```

</Listing>

这种惯用法背后没有强有力的理由：这只是出现的约定，人们已经习惯了以这种方式阅读和编写 Rust 代码。

这种惯用法的例外是如果我们使用 `use` 语句将两个同名项引入作用域，因为 Rust 不允许这样做。代码清单 7-15 显示了如何将两个同名但不同父模块的 `Result` 类型引入作用域，以及如何引用它们。

<Listing number="7-15" file-name="src/lib.rs" caption="将两个同名类型引入同一作用域需要使用它们的父模块。">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-15/src/lib.rs:here}}
```

</Listing>

如你所见，使用父模块区分了两种 `Result` 类型。如果我们改为指定 `use std::fmt::Result` 和 `use std::io::Result`，我们将在同一作用域中有两种 `Result` 类型，Rust 不会知道我们在使用 `Result` 时指的是哪一个。

### 使用 `as` 关键字提供新名称

使用 `use` 将两个同名类型引入同一作用域的问题还有另一个解决方案：在路径之后，我们可以指定 `as` 和该类型的新本地名称或 _别名_。代码清单 7-16 显示了通过使用 `as` 重命名两个 `Result` 类型之一来编写代码清单 7-15 中代码的另一种方法。

<Listing number="7-16" file-name="src/lib.rs" caption="使用 `as` 关键字将类型引入作用域时重命名类型">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-16/src/lib.rs:here}}
```

</Listing>

在第二个 `use` 语句中，我们为 `std::io::Result` 类型选择了新名称 `IoResult`，这不会与我们同样引入作用域的 `std::fmt` 中的 `Result` 冲突。代码清单 7-15 和代码清单 7-16 都被认为是惯用的，所以选择取决于你！

### 使用 `pub use` 重新导出名称

当我们使用 `use` 关键字将名称引入作用域时，该名称对我们导入它的作用域是私有的。为了使该作用域外的代码能够引用该名称，就像它在该作用域中定义一样，我们可以组合 `pub` 和 `use`。这种技术称为 _重新导出_，因为我们正在将项引入作用域，但也使该项可供其他人引入到他们的作用域中。

代码清单 7-17 显示了代码清单 7-11 中的代码，根模块中的 `use` 更改为 `pub use`。

<Listing number="7-17" file-name="src/lib.rs" caption="使用 `pub use` 使名称可供新作用域中的任何代码使用">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-17/src/lib.rs}}
```

</Listing>

在此更改之前，外部代码必须使用路径 `restaurant::front_of_house::hosting::add_to_waitlist()` 调用 `add_to_waitlist` 函数，这也需要将 `front_of_house` 模块标记为 `pub`。现在这个 `pub use` 已经从根模块重新导出了 `hosting` 模块，外部代码可以使用路径 `restaurant::hosting::add_to_waitlist()`。

当你代码的内部结构与调用你代码的程序员对领域的思考方式不同时，重新导出很有用。例如，在这个餐厅比喻中，经营餐厅的人思考"前厅"和"后厅"。但访问餐厅的顾客可能不会以这些术语思考餐厅的部分。使用 `pub use`，我们可以用一种结构编写代码，但暴露不同的结构。这样做使我们的库对从事库工作的程序员和调用库的程序员都组织良好。我们将在第 14 章的["导出便捷的公共 API"][ch14-pub-use]<!-- ignore -->中查看 `pub use` 的另一个示例以及它如何影响你的 crate 的文档。

### 使用外部包

在第 2 章中，我们编写了一个使用名为 `rand` 的外部包来获取随机数的猜数字游戏项目。要在项目中使用 `rand`，我们在 _Cargo.toml_ 中添加了这行：

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch02-00-guessing-game-tutorial.md
* ch14-03-cargo-workspaces.md
-->

<Listing file-name="Cargo.toml">

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:9:}}
```

</Listing>

在 _Cargo.toml_ 中将 `rand` 添加为依赖项告诉 Cargo 从 [crates.io](https://crates.io/) 下载 `rand` 包和任何依赖项，并使 `rand` 可用于我们的项目。

然后，为了将 `rand` 定义引入我们包的作用域，我们添加了一个以 crate 名称 `rand` 开始的 `use` 行，并列出我们想要引入作用域的项。回想一下，在第 2 章的["生成随机数"][rand]<!-- ignore -->中，我们将 `Rng` trait 引入作用域并调用了 `rand::thread_rng` 函数：

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:ch07-04}}
```

Rust 社区的成员在 [crates.io](https://crates.io/) 上提供了许多包，将其中任何一个拉入你的包都涉及这些相同的步骤：在包的 _Cargo.toml_ 文件中列出它们，并使用 `use` 将它们 crate 中的项引入作用域。

注意，标准 `std` 库也是我们包外部的 crate。因为标准库随 Rust 语言一起提供，我们不需要更改 _Cargo.toml_ 来包含 `std`。但是我们确实需要使用 `use` 引用它以将其项引入我们包的作用域。例如，对于 `HashMap`，我们将使用这行：

```rust
use std::collections::HashMap;
```

这是一个以 `std` 开始的绝对路径，标准库 crate 的名称。

<!-- Old headings. Do not remove or links may break. -->

<a id="using-nested-paths-to-clean-up-large-use-lists"></a>

### 使用嵌套路径清理 `use` 列表

如果我们在同一个 crate 或同一个模块中使用多个定义的项，将每个项单独列在一行上可能会占用文件中大量的垂直空间。例如，我们在代码清单 2-4 的猜数字游戏中的这两个 `use` 语句将 `std` 中的项引入作用域：

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-01-use-std-unnested/src/main.rs:here}}
```

</Listing>

相反，我们可以使用嵌套路径在一行中将相同的项引入作用域。我们通过指定路径的公共部分，后跟两个冒号，然后用花括号包围路径不同部分的列表来实现这一点，如代码清单 7-18 所示。

<Listing number="7-18" file-name="src/main.rs" caption="指定嵌套路径以将多个具有相同前缀的项引入作用域">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-18/src/main.rs:here}}
```

</Listing>

在更大的程序中，使用嵌套路径从同一个 crate 或模块将许多项引入作用域可以大大减少所需的单独 `use` 语句数量！

我们可以在路径的任何级别使用嵌套路径，这在组合共享子路径的两个 `use` 语句时很有用。例如，代码清单 7-19 显示了两个 `use` 语句：一个将 `std::io` 引入作用域，另一个将 `std::io::Write` 引入作用域。

<Listing number="7-19" file-name="src/lib.rs" caption="两个 `use` 语句，其中一个另一个的子路径">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-19/src/lib.rs}}
```

</Listing>

这两个路径的公共部分是 `std::io`，这是完整的第一个路径。要将这两个路径合并为一个 `use` 语句，我们可以在嵌套路径中使用 `self`，如代码清单 7-20 所示。

<Listing number="7-20" file-name="src/lib.rs" caption="将代码清单 7-19 中的路径合并为一个 `use` 语句">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-20/src/lib.rs}}
```

</Listing>

这行将 `std::io` 和 `std::io::Write` 引入作用域。

<!-- Old headings. Do not remove or links may break. -->

<a id="the-glob-operator"></a>

### 使用 Glob 操作符导入项

如果我们想将路径中定义的 _所有_ 公共项引入作用域，我们可以指定该路径后跟 `*` glob 操作符：

```rust
use std::collections::*;
```

这个 `use` 语句将 `std::collections` 中定义的所有公共项引入当前作用域。使用 glob 操作符时要小心！Glob 可能使更难判断作用域中有哪些名称以及程序中使用的名称定义在哪里。此外，如果依赖项更改其定义，你导入的内容也会更改，如果你升级依赖项，如果依赖项添加了与你同一作用域中的定义同名的定义，这可能导致编译器错误。

Glob 操作符通常在测试中使用，将测试下的所有内容引入 `tests` 模块；我们将在第 11 章的["如何编写测试"][writing-tests]<!-- ignore -->中讨论这一点。Glob 操作符有时也用作预导入模式的一部分：有关该模式的更多信息，请参阅[标准库文档](../std/prelude/index.html#other-preludes)<!-- ignore -->。

[ch14-pub-use]: ch14-02-publishing-to-crates-io.html#exporting-a-convenient-public-api
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number
[writing-tests]: ch11-01-writing-tests.html#how-to-write-tests
