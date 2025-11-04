## Hello, Cargo!

Cargo 是 Rust 的构建系统和包管理器。大多数 Rustaceans 使用此工具来管理他们的 Rust 项目，因为 Cargo 为您处理许多任务，例如构建代码、下载代码依赖的库以及构建这些库。（我们称您的代码需要的库为_依赖项_。）

最简单的 Rust 程序（如我们到目前为止编写的程序）没有任何依赖项。如果我们使用 Cargo 构建 "Hello, world!" 项目，它只会使用 Cargo 处理构建代码的部分。当您编写更复杂的 Rust 程序时，您会添加依赖项，如果您使用 Cargo 启动项目，添加依赖项将容易得多。

由于绝大多数 Rust 项目都使用 Cargo，本书的其余部分假定您也使用 Cargo。如果您使用了["安装"][installation]<!-- ignore -->部分中讨论的官方安装程序，Cargo 会随 Rust 一起安装。如果您通过其他方式安装了 Rust，请通过在终端中输入以下内容来检查是否安装了 Cargo：

```console
$ cargo --version
```

如果您看到版本号，说明您已安装！如果您看到错误（例如 `command
not found`），请查看您的安装方法的文档以确定如何单独安装 Cargo。

### 使用 Cargo 创建项目

让我们使用 Cargo 创建一个新项目，并看看它与我们原始的 "Hello, world!" 项目有何不同。导航回您的 _projects_ 目录（或您决定存储代码的任何地方）。然后，在任何操作系统上，运行以下命令：

```console
$ cargo new hello_cargo
$ cd hello_cargo
```

第一个命令创建一个名为 _hello_cargo_ 的新目录和项目。我们将项目命名为 _hello_cargo_，Cargo 在相同名称的目录中创建其文件。

进入 _hello_cargo_ 目录并列出文件。您会看到 Cargo 为我们生成了两个文件和一个目录：一个 _Cargo.toml_ 文件和一个包含 _main.rs_ 文件的 _src_ 目录。

它还初始化了一个新的 Git 仓库以及一个 _.gitignore_ 文件。如果您在现有 Git 仓库内运行 `cargo new`，则不会生成 Git 文件；您可以通过使用 `cargo new --vcs=git` 来覆盖此行为。

> 注意：Git 是一个常见的版本控制系统。您可以通过使用 `--vcs` 标志将 `cargo new` 更改为使用不同的版本控制系统或不使用版本控制系统。运行 `cargo new --help` 以查看可用选项。

在您选择的文本编辑器中打开 _Cargo.toml_。它应该类似于清单 1-2 中的代码。

<Listing number="1-2" file-name="Cargo.toml" caption="由 `cargo new` 生成的 *Cargo.toml* 的内容">

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2024"

[dependencies]
```

</Listing>

此文件采用 [_TOML_][toml]<!-- ignore -->（_Tom's Obvious, Minimal Language_）格式，这是 Cargo 的配置格式。

第一行 `[package]` 是一个节标题，表示以下语句正在配置包。当我们向此文件添加更多信息时，我们将添加其他节。

接下来三行设置 Cargo 编译程序所需的配置信息：名称、版本和要使用的 Rust 版本。我们将在[附录 E][appendix-e]<!-- ignore -->中讨论 `edition` 键。

最后一行 `[dependencies]` 是您列出项目任何依赖项的节的开始。在 Rust 中，代码包被称为_ crate_。我们不需要此项目的任何其他 crate，但在第 2 章的第一个项目中会需要，因此我们将在那时使用此依赖项节。

现在打开 _src/main.rs_ 并查看：

<span class="filename">文件名：src/main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

Cargo 为您生成了一个 "Hello, world!" 程序，就像我们在清单 1-1 中编写的程序一样！到目前为止，我们的项目与 Cargo 生成的项目之间的区别是 Cargo 将代码放在 _src_ 目录中，并且我们在顶级目录中有一个 _Cargo.toml_ 配置文件。

Cargo 期望您的源文件位于 _src_ 目录内。顶级项目目录仅用于 README 文件、许可证信息、配置文件以及任何其他与代码无关的内容。使用 Cargo 有助于您组织项目。每个东西都有其位置，每个东西都在其位置上。

如果您启动了一个不使用 Cargo 的项目（就像我们对 "Hello, world!" 项目所做的那样），您可以将其转换为使用 Cargo 的项目。将项目代码移动到 _src_ 目录并创建适当的 _Cargo.toml_ 文件。获得该 _Cargo.toml_ 文件的一个简单方法是运行 `cargo init`，它将自动为您创建。

### 构建和运行 Cargo 项目

现在让我们看看使用 Cargo 构建和运行 "Hello, world!" 程序时有什么不同！从您的 _hello_cargo_ 目录，通过输入以下命令来构建项目：

```console
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
```

此命令在 _target/debug/hello_cargo_（或在 Windows 上为 _target\debug\hello_cargo.exe_）中创建可执行文件，而不是在当前目录中。由于默认构建是调试构建，Cargo 将二进制文件放在名为 _debug_ 的目录中。您可以使用以下命令运行可执行文件：

```console
$ ./target/debug/hello_cargo # or .\target\debug\hello_cargo.exe on Windows
Hello, world!
```

如果一切顺利，`Hello, world!` 应该打印到终端。首次运行 `cargo
build` 还会导致 Cargo 在顶级创建一个新文件：_Cargo.lock_。此文件跟踪项目中依赖项的确切版本。此项目没有依赖项，因此文件有点稀疏。您永远不需要手动更改此文件；Cargo 会为您管理其内容。

我们刚刚使用 `cargo build` 构建了一个项目，并使用 `./target/debug/hello_cargo` 运行它，但我们也可以使用 `cargo run` 在一个命令中编译代码然后运行生成的可执行文件：

```console
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

使用 `cargo run` 比必须记住运行 `cargo build` 然后使用二进制文件的完整路径更方便，因此大多数开发人员使用 `cargo run`。

请注意，这次我们没有看到表明 Cargo 正在编译 `hello_cargo` 的输出。Cargo 发现文件没有更改，因此它没有重新构建，只是运行了二进制文件。如果您修改了源代码，Cargo 会在运行之前重新构建项目，您会看到此输出：

```console
$ cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Cargo 还提供了一个名为 `cargo check` 的命令。此命令快速检查您的代码以确保它编译，但不生成可执行文件：

```console
$ cargo check
   Checking hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

为什么您不想要可执行文件？通常，`cargo check` 比 `cargo build` 快得多，因为它跳过了生成可执行文件的步骤。如果您在编写代码时不断检查您的工作，使用 `cargo check` 将加快让您知道项目是否仍在编译的过程！因此，许多 Rustaceans 在编写程序时定期运行 `cargo check` 以确保它编译。然后，当他们准备使用可执行文件时，他们运行 `cargo build`。

让我们总结一下到目前为止我们学到的关于 Cargo 的知识：

- 我们可以使用 `cargo new` 创建项目。
- 我们可以使用 `cargo build` 构建项目。
- 我们可以使用 `cargo run` 一步构建和运行项目。
- 我们可以使用 `cargo check` 在不生成二进制文件的情况下构建项目以检查错误。
- Cargo 将构建结果存储在 _target/debug_ 目录中，而不是将其保存在与代码相同的目录中。

使用 Cargo 的另一个优点是，无论您使用哪个操作系统，命令都是相同的。因此，在这一点上，我们将不再提供针对 Linux 和 macOS 与 Windows 的特定说明。

### 构建发布版本

当您的项目最终准备好发布时，您可以使用 `cargo build
--release` 使用优化来编译它。此命令将在 _target/release_ 而不是 _target/debug_ 中创建可执行文件。优化使您的 Rust 代码运行得更快，但启用它们会延长程序编译所需的时间。这就是为什么有两个不同的配置文件：一个用于开发，当您想要快速且频繁地重新构建时；另一个用于构建您将提供给用户的最终程序，该程序不会重复重新构建，并且将尽可能快地运行。如果您正在对代码的运行时间进行基准测试，请确保运行 `cargo build --release` 并使用 _target/release_ 中的可执行文件进行基准测试。

<!-- Old headings. Do not remove or links may break. -->
<a id="cargo-as-convention"></a>

### 利用 Cargo 的约定

对于简单项目，Cargo 相对于仅使用 `rustc` 并没有提供太多价值，但随着程序变得更加复杂，它将证明其价值。一旦程序增长到多个文件或需要依赖项，让 Cargo 协调构建就容易得多。

尽管 `hello_cargo` 项目很简单，但它现在使用了许多您将在 Rust 职业生涯的其余部分中使用的真实工具。实际上，要处理任何现有项目，您可以使用以下命令使用 Git 检出代码，切换到该项目目录，然后构建：

```console
$ git clone example.org/someproject
$ cd someproject
$ cargo build
```

有关 Cargo 的更多信息，请查看[其文档][cargo]。

## 总结

您已经在 Rust 之旅中有了一个良好的开端！在本章中，您学习了如何：

- 使用 `rustup` 安装最新稳定版本的 Rust。
- 更新到更新的 Rust 版本。
- 打开本地安装的文档。
- 直接使用 `rustc` 编写和运行 "Hello, world!" 程序。
- 使用 Cargo 的约定创建和运行新项目。

这是构建一个更实质性的程序以习惯阅读和编写 Rust 代码的好时机。因此，在第 2 章中，我们将构建一个猜数字游戏程序。如果您更愿意从学习常见编程概念在 Rust 中的工作原理开始，请参见第 3 章，然后返回第 2 章。

[installation]: ch01-01-installation.html#installation
[toml]: https://toml.io
[appendix-e]: appendix-05-editions.html
[cargo]: https://doc.rust-lang.org/cargo/
