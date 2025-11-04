## 包和 Crate

我们将介绍的模块系统的第一部分是包和 crate。

一个 _crate_ 是 Rust 编译器一次考虑的最少代码量。即使你运行 `rustc` 而不是 `cargo` 并传递单个源代码文件（就像我们在第 1 章的["Rust 程序基础"][basics]<!-- ignore -->中所做的那样），编译器也会将该文件视为一个 crate。Crate 可以包含模块，并且模块可以在与 crate 一起编译的其他文件中定义，我们将在接下来的部分中看到。

Crate 可以有两种形式之一：二进制 crate 或库 crate。_二进制 crate_ 是可以编译为可执行文件的程序，你可以运行它们，例如命令行程序或服务器。每个必须有一个名为 `main` 的函数，该函数定义可执行文件运行时会发生什么。到目前为止，我们创建的所有 crate 都是二进制 crate。

_库 crate_ 没有 `main` 函数，它们不会编译为可执行文件。相反，它们定义旨在与多个项目共享的功能。例如，我们在[第 2 章][rand]<!-- ignore -->中使用的 `rand` crate 提供了生成随机数的功能。大多数时候，当 Rustaceans 说"crate"时，他们指的是库 crate，并且他们将"crate"与"库"的一般编程概念互换使用。

_crate 根_ 是 Rust 编译器开始并构成 crate 根模块的源文件（我们将在["使用模块控制作用域和隐私"][modules]<!-- ignore -->中深入解释模块）。

一个 _包_ 是一个或多个 crate 的集合，提供一组功能。一个包包含一个 _Cargo.toml_ 文件，该文件描述如何构建这些 crate。Cargo 实际上是一个包，它包含你一直在用来构建代码的命令行工具的二进制 crate。Cargo 包还包含二进制 crate 依赖的库 crate。其他项目可以依赖 Cargo 库 crate 来使用与 Cargo 命令行工具相同的逻辑。

一个包可以包含任意数量的二进制 crate，但最多只能有一个库 crate。一个包必须包含至少一个 crate，无论是库还是二进制 crate。

让我们看看当我们创建一个包时会发生什么。首先，我们输入命令 `cargo new my-project`：

```console
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```

在我们运行 `cargo new my-project` 之后，我们使用 `ls` 查看 Cargo 创建的内容。在 _my-project_ 目录中，有一个 _Cargo.toml_ 文件，给我们一个包。还有一个包含 _main.rs_ 的 _src_ 目录。在文本编辑器中打开 _Cargo.toml_ 并注意没有提到 _src/main.rs_。Cargo 遵循一个约定，即 _src/main.rs_ 是与包同名的二进制 crate 的 crate 根。同样，Cargo 知道如果包目录包含 _src/lib.rs_，则包包含一个与包同名的库 crate，并且 _src/lib.rs_ 是其 crate 根。Cargo 将 crate 根文件传递给 `rustc` 以构建库或二进制文件。

这里，我们有一个只包含 _src/main.rs_ 的包，这意味着它只包含一个名为 `my-project` 的二进制 crate。如果一个包包含 _src/main.rs_ 和 _src/lib.rs_，它有两个 crate：一个二进制和一个库，都与包同名。一个包可以通过将文件放在 _src/bin_ 目录中来拥有多个二进制 crate：每个文件将是一个单独的二进制 crate。

[basics]: ch01-02-hello-world.html#rust-program-basics
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number
