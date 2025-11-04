## 安装

第一步是安装 Rust。我们将通过 `rustup` 下载 Rust，这是一个用于管理 Rust 版本和相关工具的命令行工具。您需要互联网连接才能下载。

> 注意：如果您出于某种原因不想使用 `rustup`，请查看[其他 Rust 安装方法页面][otherinstall]以获取更多选项。

以下步骤将安装 Rust 编译器的最新稳定版本。Rust 的稳定性保证确保本书中所有可以编译的示例将继续在新版本的 Rust 中编译。输出在不同版本之间可能略有不同，因为 Rust 经常改进错误消息和警告。换句话说，您使用这些步骤安装的任何较新的稳定版本的 Rust 都应该与本书的内容一起正常工作。

> ### 命令行符号
>
> 在本章和整本书中，我们将显示一些在终端中使用的命令。您应该在终端中输入的所有行都以 `$` 开头。您不需要输入 `$` 字符；它是显示的命令行提示符，用于指示每个命令的开始。不以 `$` 开头的行通常显示上一个命令的输出。此外，特定于 PowerShell 的示例将使用 `>` 而不是 `$`。

### 在 Linux 或 macOS 上安装 `rustup`

如果您使用 Linux 或 macOS，请打开终端并输入以下命令：

```console
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

该命令下载一个脚本并开始安装 `rustup` 工具，该工具会安装最新稳定版本的 Rust。系统可能会提示您输入密码。如果安装成功，将出现以下行：

```text
Rust is installed now. Great!
```

您还需要一个_链接器_，这是 Rust 用来将其编译输出合并到一个文件中的程序。您可能已经有一个了。如果您遇到链接器错误，您应该安装 C 编译器，它通常会包含一个链接器。C 编译器也很有用，因为一些常见的 Rust 包依赖于 C 代码，需要 C 编译器。

在 macOS 上，您可以通过运行以下命令获取 C 编译器：

```console
$ xcode-select --install
```

Linux 用户通常应该根据其发行版的文档安装 GCC 或 Clang。例如，如果您使用 Ubuntu，可以安装 `build-essential` 包。

### 在 Windows 上安装 `rustup`

在 Windows 上，访问 [https://www.rust-lang.org/tools/install][install]<!-- ignore
--> 并按照安装 Rust 的说明进行操作。在安装过程中的某个时刻，系统会提示您安装 Visual Studio。这提供了编译程序所需的链接器和本机库。如果您在此步骤需要更多帮助，请参阅
[https://rust-lang.github.io/rustup/installation/windows-msvc.html][msvc]<!--
ignore -->。

本书的其余部分使用在 _cmd.exe_ 和 PowerShell 中都可以工作的命令。如果有特定差异，我们将说明使用哪个。

### 故障排除

要检查您是否正确安装了 Rust，请打开 shell 并输入以下行：

```console
$ rustc --version
```

您应该看到最新稳定版本的版本号、提交哈希和提交日期，格式如下：

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

如果您看到此信息，则表示您已成功安装 Rust！如果您没有看到此信息，请检查 Rust 是否在您的 `%PATH%` 系统变量中，如下所示。

在 Windows CMD 中，使用：

```console
> echo %PATH%
```

在 PowerShell 中，使用：

```powershell
> echo $env:Path
```

在 Linux 和 macOS 中，使用：

```console
$ echo $PATH
```

如果这一切都正确，但 Rust 仍然无法工作，您可以在许多地方获得帮助。在[社区页面][community]上了解如何与其他 Rustaceans（我们对自己的一个有趣的昵称）取得联系。

### 更新和卸载

一旦通过 `rustup` 安装了 Rust，更新到新发布的版本很容易。在您的 shell 中，运行以下更新脚本：

```console
$ rustup update
```

要卸载 Rust 和 `rustup`，请在您的 shell 中运行以下卸载脚本：

```console
$ rustup self uninstall
```

<!-- Old headings. Do not remove or links may break. -->
<a id="local-documentation"></a>

### 阅读本地文档

Rust 的安装还包括文档的本地副本，以便您可以离线阅读。运行 `rustup doc` 在浏览器中打开本地文档。

每当标准库提供类型或函数，而您不确定它的作用或如何使用它时，请使用应用程序编程接口（API）文档来查找！

<!-- Old headings. Do not remove or links may break. -->
<a id="text-editors-and-integrated-development-environments"></a>

### 使用文本编辑器和 IDE

本书不假设您使用什么工具来编写 Rust 代码。几乎任何文本编辑器都可以完成这项工作！但是，许多文本编辑器和集成开发环境（IDE）都具有对 Rust 的内置支持。您始终可以在 Rust 网站上的[工具页面][tools]找到相当多编辑器和 IDE 的最新列表。

### 离线使用本书

在几个示例中，我们将使用标准库之外的 Rust 包。要完成这些示例，您需要互联网连接，或者提前下载这些依赖项。要提前下载依赖项，您可以运行以下命令。（我们稍后会详细解释 `cargo` 是什么以及每个命令的作用。）

```console
$ cargo new get-dependencies
$ cd get-dependencies
$ cargo add rand@0.8.5 trpl@0.2.0
```

这将缓存这些包的下载，这样您以后就不需要下载它们了。一旦您运行了此命令，就不需要保留 `get-dependencies` 文件夹。如果您运行了此命令，您可以在本书的其余部分中对所有 `cargo` 命令使用 `--offline` 标志，以使用这些缓存的版本而不是尝试使用网络。

[otherinstall]: https://forge.rust-lang.org/infra/other-installation-methods.html
[install]: https://www.rust-lang.org/tools/install
[msvc]: https://rust-lang.github.io/rustup/installation/windows-msvc.html
[community]: https://www.rust-lang.org/community
[tools]: https://www.rust-lang.org/tools
