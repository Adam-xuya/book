<!-- Old headings. Do not remove or links may break. -->

<a id="writing-error-messages-to-standard-error-instead-of-standard-output"></a>

## 将错误重定向到标准错误

目前，我们使用 `println!` 宏将所有输出写入终端。在大多数终端中，有两种输出：用于一般信息的 _标准输出_（`stdout`）和用于错误消息的 _标准错误_（`stderr`）。这种区别使用户可以选择将程序的成功输出定向到文件，但仍在屏幕上打印错误消息。

`println!` 宏只能打印到标准输出，所以我们必须使用其他东西来打印到标准错误。

### 检查错误写入的位置

首先，让我们观察 `minigrep` 打印的内容当前如何写入标准输出，包括我们想要写入标准错误而不是标准输出的任何错误消息。我们将通过将标准输出流重定向到文件同时故意引起错误来做到这一点。我们不会重定向标准错误流，所以发送到标准错误的任何内容将继续显示在屏幕上。

命令行程序应该将错误消息发送到标准错误流，这样即使我们将标准输出流重定向到文件，我们仍然可以在屏幕上看到错误消息。我们的程序目前行为不佳：我们即将看到它将错误消息输出保存到文件而不是！

为了演示此行为，我们将使用 `>` 和文件路径 _output.txt_ 运行程序，我们希望将标准输出流重定向到该文件。我们不会传递任何参数，这应该会导致错误：

```console
$ cargo run > output.txt
```

`>` 语法告诉 shell 将标准输出的内容写入 _output.txt_ 而不是屏幕。我们没有看到我们期望打印到屏幕的错误消息，所以这意味着它一定已经在文件中了。这是 _output.txt_ 包含的内容：

```text
Problem parsing arguments: not enough arguments
```

是的，我们的错误消息正在打印到标准输出。像这样的错误消息打印到标准错误更有用，这样只有成功运行的数据才会最终出现在文件中。我们将更改它。

### 将错误打印到标准错误

我们将使用代码清单 12-24 中的代码来更改错误消息的打印方式。由于我们在本章早些时候所做的重构，所有打印错误消息的代码都在一个函数 `main` 中。标准库提供了 `eprintln!` 宏，它打印到标准错误流，所以让我们将我们调用 `println!` 来打印错误的两个地方更改为使用 `eprintln!`。

<Listing number="12-24" file-name="src/main.rs" caption="使用 `eprintln!` 将错误消息写入标准错误而不是标准输出">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-24/src/main.rs:here}}
```

</Listing>

让我们现在以相同的方式再次运行程序，不带任何参数，并使用 `>` 重定向标准输出：

```console
$ cargo run > output.txt
Problem parsing arguments: not enough arguments
```

现在我们在屏幕上看到错误，_output.txt_ 不包含任何内容，这是命令行程序期望的行为。

让我们再次使用不会导致错误但仍然将标准输出重定向到文件的参数运行程序，如下所示：

```console
$ cargo run -- to poem.txt > output.txt
```

我们不会看到任何输出到终端，_output.txt_ 将包含我们的结果：

<span class="filename">Filename: output.txt</span>

```text
Are you nobody, too?
How dreary to be somebody!
```

这演示了我们现在适当地使用标准输出进行成功输出，使用标准错误进行错误输出。

## 总结

本章回顾了你到目前为止学到的一些主要概念，并涵盖了如何在 Rust 中执行常见的 I/O 操作。通过使用命令行参数、文件、环境变量和用于打印错误的 `eprintln!` 宏，你现在准备编写命令行应用程序。结合前面章节的概念，你的代码将组织良好，在适当的数据结构中有效存储数据，很好地处理错误，并经过良好测试。

接下来，我们将探索一些受函数式语言影响的 Rust 功能：闭包和迭代器。
