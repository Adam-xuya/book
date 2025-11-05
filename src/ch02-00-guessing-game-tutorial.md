# 编写猜数字游戏

让我们通过一起完成一个实践项目来深入 Rust！本章通过向您展示如何在真实程序中使用它们来介绍一些常见的 Rust 概念。您将学习 `let`、`match`、方法、关联函数、外部 crate 等等！在接下来的章节中，我们将更详细地探讨这些概念。在本章中，您将只练习基础知识。

我们将实现一个经典的初学者编程问题：猜数字游戏。它的工作原理如下：程序将生成一个 1 到 100 之间的随机整数。然后它将提示玩家输入猜测。输入猜测后，程序将指示猜测是太低还是太高。如果猜测正确，游戏将打印祝贺消息并退出。

## 设置新项目

要设置新项目，请转到您在第 1 章中创建的 _projects_ 目录，并使用 Cargo 创建新项目，如下所示：

```console
$ cargo new guessing_game
$ cd guessing_game
```

第一个命令 `cargo new` 将项目名称（`guessing_game`）作为第一个参数。第二个命令切换到新项目的目录。

查看生成的 _Cargo.toml_ 文件：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial
rm -rf no-listing-01-cargo-new
cargo new no-listing-01-cargo-new --name guessing_game
cd no-listing-01-cargo-new
cargo run > output.txt 2>&1
cd ../../..
-->

<span class="filename">文件名：Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/Cargo.toml}}
```

如您在第 1 章中看到的，`cargo new` 为您生成了一个 "Hello, world!" 程序。查看 _src/main.rs_ 文件：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/src/main.rs}}
```

现在让我们使用 `cargo run` 命令在同一步骤中编译并运行这个 "Hello, world!" 程序：

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/output.txt}}
```

当您需要快速迭代项目时，`run` 命令很有用，就像我们在这个游戏中要做的那样，在继续下一个之前快速测试每次迭代。

重新打开 _src/main.rs_ 文件。您将在此文件中编写所有代码。

## 处理猜测

猜数字游戏程序的第一部分将要求用户输入，处理该输入，并检查输入是否符合预期形式。首先，我们将允许玩家输入猜测。将清单 2-1 中的代码输入到 _src/main.rs_ 中。

<Listing number="2-1" file-name="src/main.rs" caption="从用户获取猜测并打印的代码">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:all}}
```

</Listing>

这段代码包含大量信息，所以让我们逐行查看。要获取用户输入然后将结果作为输出打印，我们需要将 `io` 输入/输出库引入作用域。`io` 库来自标准库，称为 `std`：

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:io}}
```

默认情况下，Rust 在标准库中定义了一组项目，并将其引入每个程序的作用域。这组项目称为_预导入_，您可以在[标准库文档][prelude]中查看其中的所有内容。

如果您想使用的类型不在预导入中，您必须使用 `use` 语句显式将该类型引入作用域。使用 `std::io` 库为您提供了许多有用的功能，包括接受用户输入的能力。

如您在第 1 章中看到的，`main` 函数是程序的入口点：

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:main}}
```

`fn` 语法声明一个新函数；括号 `()` 表示没有参数；花括号 `{` 开始函数体。

您在第 1 章中也学到，`println!` 是一个将字符串打印到屏幕的宏：

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print}}
```

这段代码正在打印一个提示，说明游戏是什么并要求用户输入。

### 使用变量存储值

接下来，我们将创建一个_变量_来存储用户输入，如下所示：

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:string}}
```

现在程序变得有趣了！在这一小行中有很多事情发生。我们使用 `let` 语句创建变量。这是另一个示例：

```rust,ignore
let apples = 5;
```

这行创建了一个名为 `apples` 的新变量，并将其绑定到值 `5`。在 Rust 中，变量默认是不可变的，这意味着一旦我们给变量一个值，该值就不会改变。我们将在第 3 章的["变量和可变性"][variables-and-mutability]<!-- ignore -->部分详细讨论这个概念。要使变量可变，我们在变量名之前添加 `mut`：

```rust,ignore
let apples = 5; // 不可变
let mut bananas = 5; // 可变
```

> 注意：`//` 语法开始一个注释，该注释持续到行尾。Rust 忽略注释中的所有内容。我们将在[第 3 章][comments]<!-- ignore -->中更详细地讨论注释。

回到猜数字游戏程序，您现在知道 `let mut guess` 将引入一个名为 `guess` 的可变变量。等号 (`=`) 告诉 Rust 我们现在想要将某些内容绑定到变量。等号右侧是 `guess` 绑定到的值，它是调用 `String::new` 的结果，该函数返回 `String` 的新实例。[`String`][string]<!-- ignore --> 是标准库提供的字符串类型，是可增长的 UTF-8 编码文本。

`::new` 行中的 `::` 语法表示 `new` 是 `String` 类型的关联函数。_关联函数_是在类型上实现的函数，在这种情况下是 `String`。这个 `new` 函数创建一个新的空字符串。您会在许多类型上找到 `new` 函数，因为它是创建某种新值的函数的常见名称。

总的来说，`let mut guess = String::new();` 行创建了一个可变变量，该变量当前绑定到 `String` 的新空实例。哇！

### 接收用户输入

回想一下，我们在程序的第一行使用 `use std::io;` 包含了标准库的输入/输出功能。现在我们将从 `io` 模块调用 `stdin` 函数，这将允许我们处理用户输入：

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:read}}
```

如果我们在程序开始时没有使用 `use std::io;` 导入 `io` 模块，我们仍然可以通过将此函数调用写为 `std::io::stdin` 来使用该函数。`stdin` 函数返回 [`std::io::Stdin`][iostdin]<!-- ignore --> 的实例，这是一种表示终端标准输入句柄的类型。

接下来，`.read_line(&mut guess)` 行在标准输入句柄上调用 [`read_line`][read_line]<!--
ignore --> 方法以从用户获取输入。我们还传递 `&mut guess` 作为 `read_line` 的参数，以告诉它存储用户输入的字符串。`read_line` 的完整工作是将用户输入到标准输入的任何内容附加到字符串中（而不覆盖其内容），因此我们将该字符串作为参数传递。字符串参数需要是可变的，以便该方法可以更改字符串的内容。

`&` 表示此参数是一个_引用_，它为您提供了一种方法，让代码的多个部分访问一个数据片段，而无需多次将该数据复制到内存中。引用是一个复杂的功能，Rust 的主要优势之一是使用引用的安全性和易用性。您不需要了解很多这些细节来完成此程序。现在，您只需要知道，与变量一样，引用默认是不可变的。因此，您需要编写 `&mut guess` 而不是 `&guess` 来使其可变。（第 4 章将更详细地解释引用。）

<!-- Old headings. Do not remove or links may break. -->

<a id="handling-potential-failure-with-the-result-type"></a>

### 使用 `Result` 处理潜在失败

我们仍在处理这行代码。我们现在正在讨论第三行文本，但请注意它仍然是单个逻辑代码行的一部分。下一部分是此方法：

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:expect}}
```

我们可以将此代码写为：

```rust,ignore
io::stdin().read_line(&mut guess).expect("Failed to read line");
```

但是，一行很长很难阅读，所以最好将其拆分。在使用 `.method_name()` 语法调用方法时，引入换行符和其他空格来帮助拆分长行通常是明智的。现在让我们讨论这行代码的作用。

如前所述，`read_line` 将用户输入的任何内容放入我们传递给它的字符串中，但它也返回一个 `Result` 值。[`Result`][result]<!--
ignore --> 是一个[_枚举_][enums]<!-- ignore -->，通常称为 _enum_，这是一种可以处于多种可能状态之一的类型。我们称每个可能的状态为_变体_。

[第 6 章][enums]<!-- ignore --> 将更详细地介绍枚举。这些 `Result` 类型的目的是编码错误处理信息。

`Result` 的变体是 `Ok` 和 `Err`。`Ok` 变体表示操作成功，它包含成功生成的值。`Err` 变体意味着操作失败，它包含有关操作如何或为何失败的信息。

`Result` 类型的值（与任何类型的值一样）在其上定义了方法。`Result` 的实例有一个 [`expect` 方法][expect]<!-- ignore -->，您可以调用它。如果此 `Result` 实例是 `Err` 值，`expect` 将导致程序崩溃并显示您作为参数传递给 `expect` 的消息。如果 `read_line` 方法返回 `Err`，它可能是来自底层操作系统的错误的结果。如果此 `Result` 实例是 `Ok` 值，`expect` 将获取 `Ok` 持有的返回值并将其返回给您，以便您可以使用它。在这种情况下，该值是用户输入的字节数。

如果您不调用 `expect`，程序将编译，但您会收到警告：

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-02-without-expect/output.txt}}
```

Rust 警告您尚未使用从 `read_line` 返回的 `Result` 值，这表明程序尚未处理可能的错误。

抑制警告的正确方法是实际编写错误处理代码，但在我们的情况下，我们只想在出现问题时使程序崩溃，因此我们可以使用 `expect`。您将在[第
9 章][recover]<!-- ignore -->中了解如何从错误中恢复。

### 使用 `println!` 占位符打印值

除了右花括号，到目前为止代码中还有一行需要讨论：

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print_guess}}
```

这行打印现在包含用户输入的字符串。`{}` 花括号集是一个占位符：将 `{}` 视为小螃蟹钳子，将值固定在适当的位置。打印变量的值时，变量名可以放在花括号内。打印表达式求值的结果时，在格式字符串中放置空花括号，然后在格式字符串后跟一个逗号分隔的表达式列表，以相同的顺序在每个空花括号占位符中打印。在 `println!` 的一次调用中打印变量和表达式的结果如下所示：

```rust
let x = 5;
let y = 10;

println!("x = {x} and y + 2 = {}", y + 2);
```

此代码将打印 `x = 5 and y + 2 = 12`。

### 测试第一部分

让我们测试猜数字游戏的第一部分。使用 `cargo run` 运行它：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-01/
cargo clean
cargo run
input 6 -->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 6.44s
     Running `target/debug/guessing_game`
Guess the number!
Please input your guess.
6
You guessed: 6
```

此时，游戏的第一部分已完成：我们从键盘获取输入然后打印它。

## 生成秘密数字

接下来，我们需要生成一个用户将尝试猜测的秘密数字。秘密数字应该每次都不同，这样游戏才能多次玩得有趣。我们将使用 1 到 100 之间的随机数，这样游戏不会太难。Rust 尚未在其标准库中包含随机数功能。但是，Rust 团队确实提供了一个 [`rand`
crate][randcrate] 具有所述功能。

<!-- Old headings. Do not remove or links may break. -->
<a id="using-a-crate-to-get-more-functionality"></a>

### 使用 Crate 增加功能

请记住，crate 是 Rust 源代码文件的集合。我们一直在构建的项目是一个二进制 crate，它是一个可执行文件。`rand` crate 是一个库 crate，包含旨在在其他程序中使用的代码，不能单独执行。

Cargo 对外部 crate 的协调是 Cargo 真正闪耀的地方。在我们编写使用 `rand` 的代码之前，我们需要修改 _Cargo.toml_ 文件以将 `rand` crate 包含为依赖项。现在打开该文件，并在 Cargo 为您创建的 `[dependencies]` 节标题下方添加以下行。确保完全按照我们这里的方式指定 `rand`，使用此版本号，否则本教程中的代码示例可能无法工作：

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
* ch14-03-cargo-workspaces.md
-->

<span class="filename">文件名：Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:8:}}
```

在 _Cargo.toml_ 文件中，标题之后的所有内容都是该节的一部分，一直持续到另一个节开始。在 `[dependencies]` 中，您告诉 Cargo 您的项目依赖哪些外部 crate 以及您需要这些 crate 的哪些版本。在这种情况下，我们使用语义版本说明符 `0.8.5` 指定 `rand` crate。Cargo 理解[语义版本控制][semver]<!-- ignore -->（有时称为 _SemVer_），这是编写版本号的标准。说明符 `0.8.5` 实际上是 `^0.8.5` 的简写，这意味着至少 0.8.5 但低于 0.9.0 的任何版本。

Cargo 认为这些版本具有与版本 0.8.5 兼容的公共 API，此规范确保您将获得仍然可以与此章中的代码一起编译的最新补丁版本。不保证任何版本 0.9.0 或更高版本具有与以下示例使用的相同 API。

现在，在不更改任何代码的情况下，让我们构建项目，如清单 2-2 所示。

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
rm Cargo.lock
cargo clean
cargo build -->

<Listing number="2-2" caption="添加 `rand` crate 作为依赖项后运行 `cargo build` 的输出">

```console
$ cargo build
  Updating crates.io index
   Locking 15 packages to latest Rust 1.85.0 compatible versions
    Adding rand v0.8.5 (available: v0.9.0)
 Compiling proc-macro2 v1.0.93
 Compiling unicode-ident v1.0.17
 Compiling libc v0.2.170
 Compiling cfg-if v1.0.0
 Compiling byteorder v1.5.0
 Compiling getrandom v0.2.15
 Compiling rand_core v0.6.4
 Compiling quote v1.0.38
 Compiling syn v2.0.98
 Compiling zerocopy-derive v0.7.35
 Compiling zerocopy v0.7.35
 Compiling ppv-lite86 v0.2.20
 Compiling rand_chacha v0.3.1
 Compiling rand v0.8.5
 Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
  Finished `dev` profile [unoptimized + debuginfo] target(s) in 2.48s
```

</Listing>

您可能会看到不同的版本号（但由于 SemVer，它们都将与代码兼容！）和不同的行（取决于操作系统），并且行可能按不同的顺序排列。

当我们包含外部依赖项时，Cargo 从_注册表_获取该依赖项所需的所有最新版本，注册表是来自 [Crates.io][cratesio] 的数据副本。Crates.io 是 Rust 生态系统中的人们发布其开源 Rust 项目供他人使用的地方。

更新注册表后，Cargo 检查 `[dependencies]` 节并下载列出的任何尚未下载的 crate。在这种情况下，尽管我们只列出了 `rand` 作为依赖项，Cargo 还获取了 `rand` 依赖的其他 crate 才能工作。下载 crate 后，Rust 编译它们，然后使用可用的依赖项编译项目。

如果您立即再次运行 `cargo build` 而不进行任何更改，除了 `Finished` 行之外，您不会得到任何输出。Cargo 知道它已经下载并编译了依赖项，并且您在 _Cargo.toml_ 文件中没有更改任何内容。Cargo 也知道您没有更改代码的任何内容，因此它也不会重新编译代码。无事可做，它只是退出。

如果您打开 _src/main.rs_ 文件，进行微小的更改，然后保存并再次构建，您只会看到两行输出：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
touch src/main.rs
cargo build -->

```console
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
```

这些行显示 Cargo 仅使用您对 _src/main.rs_ 文件的微小更改更新构建。您的依赖项没有更改，因此 Cargo 知道它可以重用已经为这些依赖项下载和编译的内容。

<!-- Old headings. Do not remove or links may break. -->
<a id="ensuring-reproducible-builds-with-the-cargo-lock-file"></a>

#### 确保可重现的构建

Cargo 有一个机制，确保您或其他人每次构建代码时都可以重建相同的工件：Cargo 将仅使用您指定的依赖项版本，直到您另有指示。例如，假设下周 `rand` crate 的版本 0.8.6 发布，该版本包含重要的错误修复，但它也包含一个会破坏您代码的回归。为了处理这个问题，Rust 在您第一次运行 `cargo build` 时创建 _Cargo.lock_ 文件，所以我们现在在 _guessing_game_ 目录中有这个文件。

当您首次构建项目时，Cargo 会找出符合条件的所有依赖项版本，然后将它们写入 _Cargo.lock_ 文件。当您将来构建项目时，Cargo 将看到 _Cargo.lock_ 文件存在，并将使用其中指定的版本，而不是再次执行所有找出版本的工作。这使您可以自动获得可重现的构建。换句话说，由于 _Cargo.lock_ 文件，您的项目将保持在 0.8.5，直到您显式升级。因为 _Cargo.lock_ 文件对于可重现的构建很重要，它通常与项目中的其余代码一起提交到源代码控制。

#### 更新 Crate 以获取新版本

当您_确实_想要更新 crate 时，Cargo 提供 `update` 命令，它将忽略 _Cargo.lock_ 文件并找出符合您在 _Cargo.toml_ 中规范的所有最新版本。然后，Cargo 将这些版本写入 _Cargo.lock_ 文件。否则，默认情况下，Cargo 只会查找大于 0.8.5 且小于 0.9.0 的版本。如果 `rand` crate 发布了两个新版本 0.8.6 和 0.9.0，如果您运行 `cargo update`，您将看到以下内容：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
cargo update
assuming there is a new 0.8.x version of rand; otherwise use another update
as a guide to creating the hypothetical output shown here -->

```console
$ cargo update
    Updating crates.io index
     Locking 1 package to latest Rust 1.85.0 compatible version
    Updating rand v0.8.5 -> v0.8.6 (available: v0.9.0)
```

Cargo 忽略 0.9.0 版本。此时，您还会注意到 _Cargo.lock_ 文件中的更改，注明您现在使用的 `rand` crate 的版本是 0.8.6。要使用 `rand` 版本 0.9.0 或 0.9._x_ 系列中的任何版本，您必须将 _Cargo.toml_ 文件更新为如下所示：

```toml
[dependencies]
rand = "0.9.0"
```

下次运行 `cargo build` 时，Cargo 将更新可用 crate 的注册表，并根据您指定的新版本重新评估您的 `rand` 要求。

关于 [Cargo][doccargo]<!-- ignore --> 和[其生态系统][doccratesio]<!-- ignore -->还有很多要说的，我们将在第 14 章中讨论，但现在，这就是您需要知道的全部内容。Cargo 使重用库变得非常容易，因此 Rustaceans 能够编写由多个包组装而成的较小项目。

### 生成随机数

让我们开始使用 `rand` 生成一个要猜测的数字。下一步是更新 _src/main.rs_，如清单 2-3 所示。

<Listing number="2-3" file-name="src/main.rs" caption="添加代码以生成随机数">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:all}}
```

</Listing>

首先，我们添加行 `use rand::Rng;`。`Rng` trait 定义了随机数生成器实现的方法，这个 trait 必须在作用域中，我们才能使用这些方法。第 10 章将详细介绍 trait。

接下来，我们在中间添加两行。在第一行中，我们调用 `rand::thread_rng` 函数，它为我们提供我们将要使用的特定随机数生成器：一个本地于当前执行线程并由操作系统提供种子的生成器。然后，我们在随机数生成器上调用 `gen_range` 方法。此方法由我们通过 `use rand::Rng;` 语句引入作用域的 `Rng` trait 定义。`gen_range` 方法将范围表达式作为参数，并在该范围内生成随机数。我们在这里使用的范围表达式类型采用 `start..=end` 形式，并且在上下界都是包含的，因此我们需要指定 `1..=100` 来请求 1 到 100 之间的数字。

> 注意：您不会只是知道要使用哪些 trait 以及要从 crate 调用哪些方法和函数，因此每个 crate 都有使用说明的文档。Cargo 的另一个巧妙功能是运行 `cargo doc
> --open` 命令将在本地构建所有依赖项提供的文档并在浏览器中打开它。例如，如果您对 `rand` crate 中的其他功能感兴趣，请运行 `cargo doc --open` 并单击左侧边栏中的 `rand`。

第二行新行打印秘密数字。这在开发程序时很有用，能够测试它，但我们将在最终版本中删除它。如果程序在启动时立即打印答案，那就不是一个游戏了！

尝试运行程序几次：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-03/
cargo run
4
cargo run
5
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 7
Please input your guess.
4
You guessed: 4

$ cargo run
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 83
Please input your guess.
5
You guessed: 5
```

您应该得到不同的随机数，并且它们都应该是 1 到 100 之间的数字。干得好！

## 将猜测与秘密数字进行比较

现在我们有了用户输入和随机数，我们可以比较它们。该步骤如清单 2-4 所示。请注意，此代码还无法编译，我们将解释原因。

<Listing number="2-4" file-name="src/main.rs" caption="处理比较两个数字的可能返回值">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-04/src/main.rs:here}}
```

</Listing>

首先，我们添加另一个 `use` 语句，将名为 `std::cmp::Ordering` 的类型从标准库引入作用域。`Ordering` 类型是另一个枚举，具有变体 `Less`、`Greater` 和 `Equal`。这些是您比较两个值时可能出现的三种结果。

然后，我们在底部添加五行使用 `Ordering` 类型的新行。`cmp` 方法比较两个值，可以在任何可以比较的对象上调用。它接受您想要比较的任何内容的引用：在这里，它正在比较 `guess` 和 `secret_number`。然后，它返回我们通过 `use` 语句引入作用域的 `Ordering` 枚举的变体。我们使用 [`match`][match]<!-- ignore --> 表达式根据从使用 `guess` 和 `secret_number` 中的值调用 `cmp` 返回的 `Ordering` 的哪个变体来决定下一步做什么。

`match` 表达式由_分支_组成。分支由要匹配的_模式_和如果给 `match` 的值适合该分支的模式则应运行的代码组成。Rust 获取给 `match` 的值，并依次查看每个分支的模式。模式和 `match` 构造是强大的 Rust 功能：它们让您表达代码可能遇到的各种情况，并确保您处理所有这些情况。这些功能将分别在第 6 章和第 19 章中详细介绍。

让我们通过我们在这里使用的 `match` 表达式来看一个示例。假设用户猜测了 50，这次随机生成的秘密数字是 38。

当代码将 50 与 38 进行比较时，`cmp` 方法将返回 `Ordering::Greater`，因为 50 大于 38。`match` 表达式获取 `Ordering::Greater` 值并开始检查每个分支的模式。它查看第一个分支的模式 `Ordering::Less`，看到值 `Ordering::Greater` 不匹配 `Ordering::Less`，因此它忽略该分支中的代码并移动到下一个分支。下一个分支的模式是 `Ordering::Greater`，它_确实_匹配 `Ordering::Greater`！该分支中的关联代码将执行并在屏幕上打印 `Too big!`。`match` 表达式在第一次成功匹配后结束，因此在这种情况下它不会查看最后一个分支。

但是，清单 2-4 中的代码还无法编译。让我们试试：

<!--
The error numbers in this output should be that of the code **WITHOUT** the
anchor or snip comments
-->

```console
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-04/output.txt}}
```

错误的核心说明存在_类型不匹配_。Rust 有一个强大的静态类型系统。但是，它也有类型推断。当我们编写 `let mut guess = String::new()` 时，Rust 能够推断 `guess` 应该是 `String` 类型，并且没有让我们编写类型。另一方面，`secret_number` 是数字类型。Rust 的几种数字类型可以具有 1 到 100 之间的值：`i32`，32 位数字；`u32`，无符号 32 位数字；`i64`，64 位数字；以及其他。除非另有说明，否则 Rust 默认为 `i32`，这是 `secret_number` 的类型，除非您在其他地方添加会导致 Rust 推断不同数值类型的类型信息。错误的原因是 Rust 无法比较字符串和数字类型。

最终，我们希望将程序读取为输入的 `String` 转换为数字类型，以便我们可以将其与秘密数字进行数值比较。我们通过在 `main` 函数体中添加这一行来实现：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/src/main.rs:here}}
```

这行是：

```rust,ignore
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```

我们创建一个名为 `guess` 的变量。但是等等，程序不是已经有一个名为 `guess` 的变量了吗？确实有，但 Rust 允许我们用新值遮蔽 `guess` 的先前值，这很有帮助。_遮蔽_让我们重用 `guess` 变量名，而不是强迫我们创建两个唯一变量，例如 `guess_str` 和 `guess`。我们将在[第 3 章][shadowing]<!-- ignore -->中更详细地介绍这一点，但现在，要知道当您想将值从一种类型转换为另一种类型时，经常使用此功能。

我们将这个新变量绑定到表达式 `guess.trim().parse()`。表达式中的 `guess` 指的是包含输入作为字符串的原始 `guess` 变量。`String` 实例上的 `trim` 方法将消除开头和结尾的任何空格，在我们可以将字符串转换为 `u32` 之前必须这样做，`u32` 只能包含数字数据。用户必须按 <kbd>enter</kbd> 来满足 `read_line` 并输入他们的猜测，这会在字符串中添加换行符。例如，如果用户输入 <kbd>5</kbd> 并按 <kbd>enter</kbd>，`guess` 看起来像这样：`5\n`。`\n` 表示"换行"。（在 Windows 上，按 <kbd>enter</kbd> 会导致回车和换行，`\r\n`。）`trim` 方法消除 `\n` 或 `\r\n`，结果只是 `5`。

字符串上的 [`parse` 方法][parse]<!-- ignore --> 将字符串转换为另一种类型。在这里，我们使用它从字符串转换为数字。我们需要通过使用 `let guess: u32` 告诉 Rust 我们想要的精确数字类型。`guess` 后面的冒号 (`:`) 告诉 Rust 我们将注释变量的类型。Rust 有几个内置的数字类型；这里看到的 `u32` 是一个无符号的 32 位整数。对于小的正数来说，这是一个很好的默认选择。您将在[第 3 章][integers]<!-- ignore -->中了解其他数字类型。

此外，此示例程序中的 `u32` 注释以及与 `secret_number` 的比较意味着 Rust 将推断 `secret_number` 也应该是 `u32`。所以，现在比较将在两个相同类型的值之间进行！

`parse` 方法只能处理逻辑上可以转换为数字的字符，因此很容易导致错误。例如，如果字符串包含 `A👍%`，则无法将其转换为数字。因为它可能会失败，所以 `parse` 方法返回 `Result` 类型，就像 `read_line` 方法一样（在["使用 `Result` 处理潜在失败"](#handling-potential-failure-with-result)<!-- ignore -->中讨论过）。我们将通过再次使用 `expect` 方法以相同的方式处理此 `Result`。如果 `parse` 返回 `Err` `Result` 变体，因为它无法从字符串创建数字，`expect` 调用将崩溃游戏并打印我们给它的消息。如果 `parse` 可以成功将字符串转换为数字，它将返回 `Result` 的 `Ok` 变体，`expect` 将从 `Ok` 值返回我们想要的数字。

现在让我们运行程序：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/
touch src/main.rs
cargo run
  76
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.26s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 58
Please input your guess.
  76
You guessed: 76
Too big!
```

太好了！即使猜测前添加了空格，程序仍然发现用户猜测了 76。运行程序几次以验证不同输入的不同行为：正确猜测数字，猜测过高的数字，以及猜测过低的数字。

现在我们有了大部分游戏工作，但用户只能进行一次猜测。让我们通过添加循环来改变这一点！

## 使用循环允许多次猜测

`loop` 关键字创建一个无限循环。我们将添加一个循环，为用户提供更多猜测数字的机会：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-04-looping/src/main.rs:here}}
```

如您所见，我们已经将从猜测输入提示开始的所有内容移动到循环中。确保循环内的行每行再缩进四个空格，然后再次运行程序。程序现在将永远要求另一个猜测，这实际上引入了一个新问题。用户似乎无法退出！

用户始终可以通过使用键盘快捷键 <kbd>ctrl</kbd>-<kbd>C</kbd> 来中断程序。但是还有另一种方法可以逃脱这个贪婪的怪物，正如在["将猜测与秘密数字进行比较"](#comparing-the-guess-to-the-secret-number)<!-- ignore -->中的 `parse` 讨论中提到的：如果用户输入非数字答案，程序将崩溃。我们可以利用这一点让用户退出，如下所示：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-04-looping/
touch src/main.rs
cargo run
(too small guess)
(too big guess)
(correct guess)
quit
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.23s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 59
Please input your guess.
45
You guessed: 45
Too small!
Please input your guess.
60
You guessed: 60
Too big!
Please input your guess.
59
You guessed: 59
You win!
Please input your guess.
quit

thread 'main' panicked at src/main.rs:28:47:
Please type a number!: ParseIntError { kind: InvalidDigit }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

输入 `quit` 将退出游戏，但如您所注意到的，输入任何其他非数字输入也会退出。这至少可以说是次优的；我们希望游戏在猜测正确数字时也能停止。

### 在正确猜测后退出

让我们通过添加 `break` 语句来编程游戏在用户获胜时退出：

<span class="filename">文件名：src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-05-quitting/src/main.rs:here}}
```

在 `You win!` 之后添加 `break` 行使用户在正确猜测秘密数字时退出循环。退出循环也意味着退出程序，因为循环是 `main` 的最后一部分。

### 处理无效输入

为了进一步完善游戏的行为，而不是在用户输入非数字时使程序崩溃，让我们让游戏忽略非数字，以便用户可以继续猜测。我们可以通过更改 `guess` 从 `String` 转换为 `u32` 的行来实现，如清单 2-5 所示。

<Listing number="2-5" file-name="src/main.rs" caption="忽略非数字猜测并要求另一个猜测而不是使程序崩溃">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:here}}
```

</Listing>

我们从 `expect` 调用切换到 `match` 表达式，从在错误时崩溃转向处理错误。请记住，`parse` 返回 `Result` 类型，`Result` 是具有变体 `Ok` 和 `Err` 的枚举。我们在这里使用 `match` 表达式，就像我们对 `cmp` 方法的 `Ordering` 结果所做的那样。

如果 `parse` 能够成功地将字符串转换为数字，它将返回一个包含结果数字的 `Ok` 值。该 `Ok` 值将匹配第一个分支的模式，`match` 表达式将只返回 `parse` 生成并放入 `Ok` 值中的 `num` 值。该数字将最终出现在我们正在创建的新 `guess` 变量中我们想要的位置。

如果 `parse` _无法_将字符串转换为数字，它将返回一个包含有关错误的更多信息的 `Err` 值。`Err` 值不匹配第一个 `match` 分支中的 `Ok(num)` 模式，但它确实匹配第二个分支中的 `Err(_)` 模式。下划线 `_` 是一个通配符值；在这个例子中，我们说我们想要匹配所有 `Err` 值，无论它们内部有什么信息。所以，程序将执行第二个分支的代码 `continue`，它告诉程序转到 `loop` 的下一次迭代并要求另一个猜测。所以，实际上，程序忽略了 `parse` 可能遇到的所有错误！

现在程序中的所有内容都应该按预期工作。让我们试试：

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-05/
cargo run
(too small guess)
(too big guess)
foo
(correct guess)
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 61
Please input your guess.
10
You guessed: 10
Too small!
Please input your guess.
99
You guessed: 99
Too big!
Please input your guess.
foo
Please input your guess.
61
You guessed: 61
You win!
```

太棒了！通过一个微小的最终调整，我们将完成猜数字游戏。回想一下，程序仍在打印秘密数字。这对测试很有用，但它破坏了游戏。让我们删除输出秘密数字的 `println!`。清单 2-6 显示了最终代码。

<Listing number="2-6" file-name="src/main.rs" caption="完整的猜数字游戏代码">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-06/src/main.rs}}
```

</Listing>

此时，您已经成功构建了猜数字游戏。恭喜！

## 总结

这个项目是通过实践方式向您介绍许多新的 Rust 概念：`let`、`match`、函数、外部 crate 的使用等等。在接下来的几章中，您将更详细地了解这些概念。第 3 章涵盖大多数编程语言都有的概念，例如变量、数据类型和函数，并展示如何在 Rust 中使用它们。第 4 章探讨所有权，这是使 Rust 不同于其他语言的功能。第 5 章讨论结构体和方法语法，第 6 章解释枚举的工作原理。

[prelude]: https://doc.rust-lang.org/std/prelude/index.html
[variables-and-mutability]: ch03-01-variables-and-mutability.html#variables-and-mutability
[comments]: ch03-04-comments.html
[string]: https://doc.rust-lang.org/std/string/struct.String.html
[iostdin]: https://doc.rust-lang.org/std/io/struct.Stdin.html
[read_line]: https://doc.rust-lang.org/std/io/struct.Stdin.html#method.read_line
[result]: https://doc.rust-lang.org/std/result/enum.Result.html
[enums]: ch06-00-enums.html
[expect]: https://doc.rust-lang.org/std/result/enum.Result.html#method.expect
[recover]: ch09-02-recoverable-errors-with-result.html
[randcrate]: https://crates.io/crates/rand
[semver]: http://semver.org
[cratesio]: https://crates.io/
[doccargo]: https://doc.rust-lang.org/cargo/
[doccratesio]: https://doc.rust-lang.org/cargo/reference/publishing.html
[match]: ch06-02-match.html
[shadowing]: ch03-01-variables-and-mutability.html#shadowing
[parse]: https://doc.rust-lang.org/std/primitive.str.html#method.parse
[integers]: ch03-02-data-types.html#integer-types
