## Hello, World!

现在您已经安装了 Rust，是时候编写您的第一个 Rust 程序了。学习新语言时，传统做法是编写一个向屏幕打印文本 `Hello, world!` 的小程序，所以我们在这里也这样做！

> 注意：本书假定您对命令行有基本了解。Rust 对您的编辑或工具或代码所在的位置没有特定要求，因此如果您更喜欢使用 IDE 而不是命令行，请随时使用您喜欢的 IDE。许多 IDE 现在都有一定程度的 Rust 支持；请查看 IDE 的文档以获取详细信息。Rust 团队一直专注于通过 `rust-analyzer` 启用出色的 IDE 支持。有关更多详细信息，请参见[附录 D][devtools]<!-- ignore -->。

<!-- Old headings. Do not remove or links may break. -->
<a id="creating-a-project-directory"></a>

### 项目目录设置

您将首先创建一个目录来存储您的 Rust 代码。Rust 不关心您的代码在哪里，但对于本书中的练习和项目，我们建议在您的主目录中创建一个 _projects_ 目录，并将所有项目保存在那里。

打开终端并输入以下命令以创建 _projects_ 目录，并在 _projects_ 目录中为 "Hello, world!" 项目创建一个目录。

对于 Linux、macOS 和 Windows 上的 PowerShell，请输入：

```console
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

对于 Windows CMD，请输入：

```cmd
> mkdir "%USERPROFILE%\projects"
> cd /d "%USERPROFILE%\projects"
> mkdir hello_world
> cd hello_world
```

<!-- Old headings. Do not remove or links may break. -->
<a id="writing-and-running-a-rust-program"></a>

### Rust 程序基础

接下来，创建一个新的源文件并将其命名为 _main.rs_。Rust 文件总是以 _.rs_ 扩展名结尾。如果您的文件名使用多个单词，约定是使用下划线分隔它们。例如，使用 _hello_world.rs_ 而不是 _helloworld.rs_。

现在打开您刚刚创建的 _main.rs_ 文件，并输入清单 1-1 中的代码。

<Listing number="1-1" file-name="main.rs" caption="一个打印 `Hello, world!` 的程序">

```rust
fn main() {
    println!("Hello, world!");
}
```

</Listing>

保存文件并返回到终端窗口中的 _~/projects/hello_world_ 目录。在 Linux 或 macOS 上，输入以下命令来编译和运行文件：

```console
$ rustc main.rs
$ ./main
Hello, world!
```

在 Windows 上，请输入 `.\main` 而不是 `./main`：

```powershell
> rustc main.rs
> .\main
Hello, world!
```

无论您的操作系统是什么，字符串 `Hello, world!` 都应该打印到终端。如果您没有看到此输出，请返回安装部分的["故障排除"][troubleshooting]<!-- ignore -->部分以获取帮助。

如果 `Hello, world!` 确实打印了，恭喜！您已经正式编写了一个 Rust 程序。这使您成为 Rust 程序员——欢迎！

<!-- Old headings. Do not remove or links may break. -->

<a id="anatomy-of-a-rust-program"></a>

### Rust 程序的剖析

让我们详细审查这个 "Hello, world!" 程序。这是拼图的第一块：

```rust
fn main() {

}
```

这些行定义了一个名为 `main` 的函数。`main` 函数很特殊：它始终是每个可执行 Rust 程序中首先运行的代码。在这里，第一行声明了一个名为 `main` 的函数，它没有参数且不返回任何内容。如果有参数，它们将放在括号 (`()`) 内。

函数体用 `{}` 包裹。Rust 要求所有函数体周围都有花括号。将左花括号放在函数声明的同一行上，并在它们之间添加一个空格，这是良好的风格。

> 注意：如果您想在整个 Rust 项目中坚持标准风格，可以使用名为 `rustfmt` 的自动格式化工具来格式化您的代码（有关 `rustfmt` 的更多信息，请参见
> [附录 D][devtools]<!-- ignore -->）。Rust 团队已将此工具包含在标准 Rust 发行版中，就像 `rustc` 一样，因此它应该已经安装在您的计算机上！

`main` 函数的主体包含以下代码：

```rust
println!("Hello, world!");
```

这一行完成了这个小程序中的所有工作：它将文本打印到屏幕。这里有三个重要的细节需要注意。

首先，`println!` 调用 Rust 宏。如果它调用的是函数，则会输入为 `println`（不带 `!`）。Rust 宏是一种编写生成代码以扩展 Rust 语法的方法，我们将在[第 20 章][ch20-macros]<!-- ignore -->中更详细地讨论它们。现在，您只需要知道使用 `!` 意味着您正在调用宏而不是普通函数，并且宏并不总是遵循与函数相同的规则。

其次，您看到 `"Hello, world!"` 字符串。我们将此字符串作为参数传递给 `println!`，字符串被打印到屏幕。

第三，我们以分号 (`;`) 结束该行，这表明此表达式已结束，下一个表达式已准备好开始。Rust 代码的大多数行都以分号结尾。

<!-- Old headings. Do not remove or links may break. -->
<a id="compiling-and-running-are-separate-steps"></a>

### 编译和执行

您刚刚运行了一个新创建的程序，让我们检查过程中的每个步骤。

在运行 Rust 程序之前，您必须使用 Rust 编译器编译它，方法是输入 `rustc` 命令并将源文件的名称传递给它，如下所示：

```console
$ rustc main.rs
```

如果您有 C 或 C++ 背景，您会注意到这类似于 `gcc` 或 `clang`。编译成功后，Rust 会输出一个二进制可执行文件。

在 Linux、macOS 和 Windows 上的 PowerShell 中，您可以通过在 shell 中输入 `ls` 命令来查看可执行文件：

```console
$ ls
main  main.rs
```

在 Linux 和 macOS 上，您会看到两个文件。在 Windows 上的 PowerShell 中，您会看到与使用 CMD 相同的三个文件。在 Windows 上的 CMD 中，您将输入以下内容：

```cmd
> dir /B %= the /B option says to only show the file names =%
main.exe
main.pdb
main.rs
```

这显示了带有 _.rs_ 扩展名的源代码文件、可执行文件（在 Windows 上是 _main.exe_，但在所有其他平台上都是 _main_），以及在使用 Windows 时，一个包含调试信息的文件，扩展名为 _.pdb_。从这里开始，您运行 _main_ 或 _main.exe_ 文件，如下所示：

```console
$ ./main # or .\main on Windows
```

如果您的 _main.rs_ 是您的 "Hello, world!" 程序，这行会将 `Hello,
world!` 打印到您的终端。

如果您更熟悉动态语言（如 Ruby、Python 或 JavaScript），您可能不习惯将编译和运行程序作为单独的步骤。Rust 是一种_提前编译_的语言，这意味着您可以编译程序并将可执行文件提供给其他人，他们可以在没有安装 Rust 的情况下运行它。如果您给某人一个 _.rb_、_.py_ 或 _.js_ 文件，他们需要安装 Ruby、Python 或 JavaScript 实现（分别）。但在这些语言中，您只需要一个命令来编译和运行程序。语言设计中的一切都是权衡。

仅使用 `rustc` 编译对于简单程序来说是可以的，但随着项目的增长，您将希望管理所有选项并使其易于共享代码。接下来，我们将向您介绍 Cargo 工具，它将帮助您编写真实的 Rust 程序。

[troubleshooting]: ch01-01-installation.html#troubleshooting
[devtools]: appendix-04-useful-development-tools.html
[ch20-macros]: ch20-05-macros.html
