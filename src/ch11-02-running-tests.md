## 控制测试运行方式

就像 `cargo run` 编译你的代码然后运行生成的二进制文件一样，`cargo test` 在测试模式下编译你的代码并运行生成的测试二进制文件。`cargo test` 生成的二进制文件的默认行为是并行运行所有测试并捕获测试运行期间生成的输出，防止输出被显示并使更容易阅读与测试结果相关的输出。但是，你可以指定命令行选项来更改此默认行为。

一些命令行选项传递给 `cargo test`，一些传递给生成的测试二进制文件。为了分离这两种类型的参数，你列出传递给 `cargo test` 的参数，后跟分隔符 `--`，然后是传递给测试二进制文件的参数。运行 `cargo test --help` 显示你可以与 `cargo test` 一起使用的选项，运行 `cargo test -- --help` 显示你可以在分隔符后使用的选项。这些选项也在 [_The `rustc` Book_ 的"Tests"部分][tests]中记录。

[tests]: https://doc.rust-lang.org/rustc/tests/index.html

### 并行或连续运行测试

当你运行多个测试时，默认情况下它们使用线程并行运行，这意味着它们更快地完成运行，你会更快地获得反馈。因为测试同时运行，你必须确保你的测试不相互依赖或依赖任何共享状态，包括共享环境，例如当前工作目录或环境变量。

例如，假设你的每个测试都运行一些代码，这些代码在磁盘上创建一个名为 _test-output.txt_ 的文件并将一些数据写入该文件。然后，每个测试读取该文件中的数据并断言该文件包含特定值，这在每个测试中都不同。因为测试同时运行，一个测试可能在另一个测试写入和读取文件之间的时间内覆盖文件。第二个测试将失败，不是因为代码不正确，而是因为测试在并行运行时相互干扰。一个解决方案是确保每个测试写入不同的文件；另一个解决方案是一次运行一个测试。

如果你不想并行运行测试，或者如果你想更精细地控制使用的线程数，你可以向测试二进制文件发送 `--test-threads` 标志和你想使用的线程数。看看以下示例：

```console
$ cargo test -- --test-threads=1
```

我们将测试线程数设置为 `1`，告诉程序不要使用任何并行性。使用一个线程运行测试将比并行运行它们花费更长时间，但如果它们共享状态，测试不会相互干扰。

### 显示函数输出

默认情况下，如果测试通过，Rust 的测试库会捕获打印到标准输出的任何内容。例如，如果我们在测试中调用 `println!` 并且测试通过，我们不会在终端中看到 `println!` 输出；我们只会看到指示测试通过的行。如果测试失败，我们将看到打印到标准输出的任何内容以及失败消息的其余部分。

作为一个例子，代码清单 11-10 有一个愚蠢的函数，它打印其参数的值并返回 10，以及一个通过的测试和一个失败的测试。

<Listing number="11-10" file-name="src/lib.rs" caption="调用 `println!` 的函数的测试">

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-10/src/lib.rs}}
```

</Listing>

当我们使用 `cargo test` 运行这些测试时，我们将看到以下输出：

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-10/output.txt}}
```

注意，在此输出中，我们看不到 `I got the value 4`，这是在通过的测试运行时打印的。该输出已被捕获。来自失败测试的输出 `I got the value 8` 出现在测试摘要输出的部分中，这也显示了测试失败的原因。

如果我们也想看到通过测试的打印值，我们可以告诉 Rust 也使用 `--show-output` 显示成功测试的输出：

```console
$ cargo test -- --show-output
```

当我们使用 `--show-output` 标志再次运行代码清单 11-10 中的测试时，我们看到以下输出：

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-01-show-output/output.txt}}
```

### 按名称运行测试子集

运行完整的测试套件有时可能需要很长时间。如果你正在处理特定区域的代码，你可能只想运行与该代码相关的测试。你可以通过将你想运行的测试的名称或名称作为参数传递给 `cargo test` 来选择要运行的测试。

为了演示如何运行测试子集，我们将首先为 `add_two` 函数创建三个测试，如代码清单 11-11 所示，然后选择要运行的测试。

<Listing number="11-11" file-name="src/lib.rs" caption="三个具有三个不同名称的测试">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-11/src/lib.rs}}
```

</Listing>

如果我们运行测试而不传递任何参数，就像我们之前看到的那样，所有测试都将并行运行：

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-11/output.txt}}
```

#### 运行单个测试

我们可以将任何测试函数的名称传递给 `cargo test` 以仅运行该测试：

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-02-single-test/output.txt}}
```

只有名称为 `one_hundred` 的测试运行；其他两个测试不匹配该名称。测试输出通过显示 `2 filtered out` 让我们知道我们有更多未运行的测试。

我们不能以这种方式指定多个测试的名称；只有提供给 `cargo test` 的第一个值将被使用。但是有一种方法可以运行多个测试。

#### 过滤以运行多个测试

我们可以指定测试名称的一部分，任何名称与该值匹配的测试都将运行。例如，因为我们的两个测试名称包含 `add`，我们可以通过运行 `cargo test add` 来运行这两个：

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-03-multiple-tests/output.txt}}
```

此命令运行名称中包含 `add` 的所有测试，并过滤掉名为 `one_hundred` 的测试。还要注意，测试出现的模块成为测试名称的一部分，所以我们可以通过过滤模块名称来运行模块中的所有测试。

<!-- Old headings. Do not remove or links may break. -->

<a id="ignoring-some-tests-unless-specifically-requested"></a>

### 除非特别请求否则忽略测试

有时，几个特定的测试可能非常耗时，所以你可能希望在大多数 `cargo test` 运行期间排除它们。与其将所有你确实想运行的测试列出为参数，你可以使用 `ignore` 属性注释耗时的测试以排除它们，如下所示：

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/src/lib.rs:here}}
```

在 `#[test]` 之后，我们将 `#[ignore]` 行添加到我们要排除的测试。现在当我们运行测试时，`it_works` 运行，但 `expensive_test` 不运行：

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/output.txt}}
```

`expensive_test` 函数被列为 `ignored`。如果我们只想运行被忽略的测试，我们可以使用 `cargo test -- --ignored`：

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-04-running-ignored/output.txt}}
```

通过控制哪些测试运行，你可以确保 `cargo test` 结果将快速返回。当你处于检查 `ignored` 测试结果有意义且你有时间等待结果的时候，你可以运行 `cargo test -- --ignored` 代替。如果你想运行所有测试，无论它们是否被忽略，你可以运行 `cargo test -- --include-ignored`。
