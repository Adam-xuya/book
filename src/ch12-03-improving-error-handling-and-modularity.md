## 重构以提高模块化和错误处理

为了改进我们的程序，我们将修复与程序结构以及它如何处理潜在错误相关的四个问题。首先，我们的 `main` 函数现在执行两个任务：它解析参数和读取文件。随着程序增长，`main` 函数处理的独立任务数量将增加。随着函数获得更多职责，它变得更加难以推理、更难测试，并且更难在不破坏其某个部分的情况下进行更改。最好分离功能，以便每个函数负责一个任务。

这个问题也与第二个问题相关：虽然 `query` 和 `file_path` 是我们程序的配置变量，但像 `contents` 这样的变量用于执行程序的逻辑。`main` 变得越长，我们需要引入作用域的变量就越多；作用域中的变量越多，跟踪每个变量的目的就越困难。最好将配置变量分组到一个结构中，以便清楚地表达它们的目的。

第三个问题是我们使用 `expect` 在读取文件失败时打印错误消息，但错误消息只是打印 `Should have been able to read the file`。读取文件可能以多种方式失败：例如，文件可能丢失，或者我们可能没有打开它的权限。现在，无论情况如何，我们都会为所有情况打印相同的错误消息，这不会给用户任何信息！

第四，我们使用 `expect` 来处理错误，如果用户在没有指定足够参数的情况下运行我们的程序，他们将从 Rust 得到一个 `index out of bounds` 错误，该错误没有清楚地解释问题。最好所有的错误处理代码都在一个地方，这样未来的维护者如果错误处理逻辑需要更改，只需要咨询一个地方的代码。将所有错误处理代码放在一个地方还将确保我们打印的消息对我们的最终用户有意义。

让我们通过重构项目来解决这四个问题。

<!-- Old headings. Do not remove or links may break. -->

<a id="separation-of-concerns-for-binary-projects"></a>

### 在二进制项目中分离关注点

将多个任务的责任分配给 `main` 函数的组织问题在许多二进制项目中很常见。因此，许多 Rust 程序员发现在 `main` 函数开始变大时，拆分二进制程序的独立关注点很有用。此过程有以下步骤：

- 将程序拆分为 _main.rs_ 文件和 _lib.rs_ 文件，并将程序的逻辑移到 _lib.rs_。
- 只要命令行解析逻辑很小，它可以保留在 `main` 函数中。
- 当命令行解析逻辑开始变得复杂时，将其从 `main` 函数提取到其他函数或类型中。

在此过程之后保留在 `main` 函数中的职责应限于以下内容：

- 使用参数值调用命令行解析逻辑
- 设置任何其他配置
- 调用 _lib.rs_ 中的 `run` 函数
- 如果 `run` 返回错误，则处理错误

这种模式是关于分离关注点：_main.rs_ 处理运行程序，_lib.rs_ 处理手头任务的所有逻辑。因为你无法直接测试 `main` 函数，这种结构通过将逻辑移出 `main` 函数来让你测试所有程序逻辑。保留在 `main` 函数中的代码将足够小，可以通过阅读来验证其正确性。让我们按照此过程重新设计我们的程序。

#### 提取参数解析器

我们将把解析参数的功能提取到 `main` 将调用的函数中。代码清单 12-5 显示了 `main` 函数的新开始，它调用一个新函数 `parse_config`，我们将在 _src/main.rs_ 中定义它。

<Listing number="12-5" file-name="src/main.rs" caption="从 `main` 提取 `parse_config` 函数">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-05/src/main.rs:here}}
```

</Listing>

我们仍然将命令行参数收集到向量中，但在 `main` 函数内，我们不是将索引 1 处的参数值分配给变量 `query` 并将索引 2 处的参数值分配给变量 `file_path`，而是将整个向量传递给 `parse_config` 函数。然后，`parse_config` 函数保存确定哪个参数进入哪个变量的逻辑，并将值传回 `main`。我们仍然在 `main` 中创建 `query` 和 `file_path` 变量，但 `main` 不再负责确定命令行参数和变量如何对应。

对于我们的小程序来说，这种重新设计可能看起来有些过度，但我们正在以小步骤、增量步骤进行重构。进行此更改后，再次运行程序以验证参数解析仍然有效。经常检查你的进度是好的，以帮助在问题发生时识别问题的原因。

#### 分组配置值

我们可以采取另一个小步骤来进一步改进 `parse_config` 函数。目前，我们返回一个元组，但然后我们立即再次将该元组分解为各个部分。这表明我们可能还没有正确的抽象。

另一个显示有改进空间的指标是 `parse_config` 的 `config` 部分，这意味着我们返回的两个值是相关的，并且都是一个配置值的一部分。我们目前除了通过将两个值分组到元组中之外，没有在数据结构中传达此含义；我们将把两个值放入一个结构体中，并为每个结构体字段指定一个有意义的名称。这样做将使此代码的未来维护者更容易理解不同值如何相互关联以及它们的目的是什么。

代码清单 12-6 显示了对 `parse_config` 函数的改进。

<Listing number="12-6" file-name="src/main.rs" caption="重构 `parse_config` 以返回 `Config` 结构体的实例">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-06/src/main.rs:here}}
```

</Listing>

我们添加了一个名为 `Config` 的结构体，定义为具有名为 `query` 和 `file_path` 的字段。`parse_config` 的签名现在表明它返回一个 `Config` 值。在 `parse_config` 的主体中，我们过去返回引用 `args` 中 `String` 值的字符串切片，现在我们定义 `Config` 包含拥有的 `String` 值。`main` 中的 `args` 变量是参数值的所有者，只让 `parse_config` 函数借用它们，这意味着如果 `Config` 试图获取 `args` 中值的所有权，我们将违反 Rust 的借用规则。

我们可以通过多种方式管理 `String` 数据；最简单但效率稍低的方法是调用值的 `clone` 方法。这将为 `Config` 实例创建数据的完整副本以拥有，这比存储对字符串数据的引用花费更多时间和内存。但是，克隆数据也使我们的代码非常直接，因为我们不必管理引用的生命周期；在这种情况下，放弃一点性能来获得简单性是值得的权衡。

> ### 使用 `clone` 的权衡
>
> 许多 Rustaceans 倾向于避免使用 `clone` 来修复所有权问题，因为它的运行时成本。在[第 13 章][ch13]<!-- ignore -->中，你将学习如何在这种情况下使用更高效的方法。但现在，复制几个字符串以继续取得进展是可以的，因为你只会进行一次这些复制，并且你的文件路径和查询字符串非常小。最好有一个工作但有点低效的程序，而不是在第一次尝试时尝试过度优化代码。当你对 Rust 更有经验时，从最高效的解决方案开始会更容易，但现在，调用 `clone` 是完全可接受的。

我们已经更新了 `main`，以便它将 `parse_config` 返回的 `Config` 实例放在名为 `config` 的变量中，并且我们更新了以前使用单独的 `query` 和 `file_path` 变量的代码，以便它现在使用 `Config` 结构体上的字段。

现在我们的代码更清楚地传达了 `query` 和 `file_path` 是相关的，并且它们的目的是配置程序将如何工作。任何使用这些值的代码都知道在 `config` 实例中查找它们，这些字段以其目的命名。

#### 为 `Config` 创建构造函数

到目前为止，我们已经从 `main` 中提取了负责解析命令行参数的逻辑，并将其放在 `parse_config` 函数中。这样做帮助我们看到 `query` 和 `file_path` 值是相关的，这种关系应该在代码中传达。然后，我们添加了一个 `Config` 结构体来命名 `query` 和 `file_path` 的相关目的，并能够从 `parse_config` 函数返回值的名称作为结构体字段名称。

所以，既然 `parse_config` 函数的目的是创建 `Config` 实例，我们可以将 `parse_config` 从普通函数更改为名为 `new` 的函数，该函数与 `Config` 结构体关联。进行此更改将使代码更符合习惯。我们可以通过调用 `String::new` 来创建标准库中类型的实例，例如 `String`。类似地，通过将 `parse_config` 更改为与 `Config` 关联的 `new` 函数，我们将能够通过调用 `Config::new` 来创建 `Config` 实例。代码清单 12-7 显示了我们需要进行的更改。

<Listing number="12-7" file-name="src/main.rs" caption="将 `parse_config` 更改为 `Config::new`">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-07/src/main.rs:here}}
```

</Listing>

我们已经更新了 `main`，在调用 `parse_config` 的地方改为调用 `Config::new`。我们已经将 `parse_config` 的名称更改为 `new`，并将其移动到 `impl` 块内，这将 `new` 函数与 `Config` 关联。尝试再次编译此代码以确保它工作。

### 修复错误处理

现在我们将修复错误处理。回想一下，如果向量包含少于三个项，尝试访问 `args` 向量中索引 1 或索引 2 处的值将导致程序 panic。尝试在没有参数的情况下运行程序；它将看起来像这样：

```console
{{#include ../listings/ch12-an-io-project/listing-12-07/output.txt}}
```

行 `index out of bounds: the len is 1 but the index is 1` 是面向程序员的错误消息。它不会帮助我们的最终用户理解他们应该做什么。让我们现在修复它。

#### 改进错误消息

在代码清单 12-8 中，我们在 `new` 函数中添加一个检查，在访问索引 1 和索引 2 之前验证切片是否足够长。如果切片不够长，程序 panic 并显示更好的错误消息。

<Listing number="12-8" file-name="src/main.rs" caption="添加参数数量检查">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-08/src/main.rs:here}}
```

</Listing>

此代码类似于[我们在代码清单 9-13 中编写的 `Guess::new` 函数][ch9-custom-types]<!-- ignore -->，当 `value` 参数超出有效值范围时，我们调用 `panic!`。我们在这里不是检查值的范围，而是检查 `args` 的长度是否至少为 `3`，函数的其余部分可以在满足此条件的假设下操作。如果 `args` 少于三个项，此条件将为 `true`，我们调用 `panic!` 宏以立即结束程序。

在 `new` 中添加了这几行代码后，让我们再次在没有参数的情况下运行程序，看看现在的错误是什么样子：

```console
{{#include ../listings/ch12-an-io-project/listing-12-08/output.txt}}
```

此输出更好：我们现在有一个合理的错误消息。但是，我们也有我们不想给用户的额外信息。也许我们在代码清单 9-13 中使用的技术不是这里使用的最佳技术：调用 `panic!` 更适合编程问题而不是使用问题，[如第 9 章中讨论的][ch9-error-guidelines]<!-- ignore -->。相反，我们将使用你在第 9 章学到的另一种技术——[返回 `Result`][ch9-result]<!-- ignore -->，该结果指示成功或错误。

<!-- Old headings. Do not remove or links may break. -->

<a id="returning-a-result-from-new-instead-of-calling-panic"></a>

#### 返回 `Result` 而不是调用 `panic!`

我们可以返回一个 `Result` 值，在成功情况下包含 `Config` 实例，在错误情况下描述问题。我们还将函数名从 `new` 更改为 `build`，因为许多程序员期望 `new` 函数永远不会失败。当 `Config::build` 与 `main` 通信时，我们可以使用 `Result` 类型来指示存在问题。然后，我们可以更改 `main` 以将 `Err` 变体转换为对我们用户更实用的错误，而不包含调用 `panic!` 导致的关于 `thread 'main'` 和 `RUST_BACKTRACE` 的周围文本。

代码清单 12-9 显示我们需要对现在称为 `Config::build` 的函数的返回值和函数主体进行的更改，以返回 `Result`。注意，在我们也更新 `main` 之前，这不会编译，我们将在下一个清单中这样做。

<Listing number="12-9" file-name="src/main.rs" caption="从 `Config::build` 返回 `Result`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-09/src/main.rs:here}}
```

</Listing>

我们的 `build` 函数返回一个 `Result`，在成功情况下包含 `Config` 实例，在错误情况下包含字符串字面量。我们的错误值将始终是具有 `'static` 生命周期的字符串字面量。

我们在函数主体中进行了两个更改：当用户没有传递足够参数时，我们现在返回 `Err` 值，而不是调用 `panic!`，并且我们将 `Config` 返回值包装在 `Ok` 中。这些更改使函数符合其新类型签名。

从 `Config::build` 返回 `Err` 值允许 `main` 函数处理从 `build` 函数返回的 `Result` 值，并在错误情况下更干净地退出进程。

<!-- Old headings. Do not remove or links may break. -->

<a id="calling-confignew-and-handling-errors"></a>

#### 调用 `Config::build` 和处理错误

为了处理错误情况并打印用户友好的消息，我们需要更新 `main` 来处理 `Config::build` 返回的 `Result`，如代码清单 12-10 所示。我们还将从 `panic!` 中取出以非零错误代码退出命令行工具的责任，而是手动实现它。非零退出状态是一种约定，用于向调用我们程序的进程发出信号，表示程序以错误状态退出。

<Listing number="12-10" file-name="src/main.rs" caption="如果构建 `Config` 失败，则以错误代码退出">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-10/src/main.rs:here}}
```

</Listing>

在此清单中，我们使用了一个我们尚未详细介绍的方法：`unwrap_or_else`，它由标准库在 `Result<T, E>` 上定义。使用 `unwrap_or_else` 允许我们定义一些自定义的、非 `panic!` 的错误处理。如果 `Result` 是 `Ok` 值，此方法的行为类似于 `unwrap`：它返回 `Ok` 包装的内部值。但是，如果值是 `Err` 值，此方法调用闭包中的代码，闭包是我们定义并作为参数传递给 `unwrap_or_else` 的匿名函数。我们将在[第 13 章][ch13]<!-- ignore -->中更详细地介绍闭包。现在，你只需要知道 `unwrap_or_else` 会将 `Err` 的内部值（在这种情况下是我们在代码清单 12-9 中添加的静态字符串 `"not enough arguments"`）传递给出现在垂直管道之间的参数 `err` 中的闭包。然后，闭包中的代码可以在运行时使用 `err` 值。

我们添加了一个新的 `use` 行，将标准库中的 `process` 引入作用域。将在错误情况下运行的闭包中的代码只有两行：我们打印 `err` 值，然后调用 `process::exit`。`process::exit` 函数将立即停止程序并返回作为退出状态代码传递的数字。这类似于我们在代码清单 12-8 中使用的基于 `panic!` 的处理，但我们不再获得所有额外的输出。让我们试试：

```console
{{#include ../listings/ch12-an-io-project/listing-12-10/output.txt}}
```

太好了！此输出对我们的用户友好得多。

<!-- Old headings. Do not remove or links may break. -->

<a id="extracting-logic-from-the-main-function"></a>

### 从 `main` 提取逻辑

既然我们已经完成了配置解析的重构，让我们转向程序的逻辑。正如我们在["在二进制项目中分离关注点"](#separation-of-concerns-for-binary-projects)<!-- ignore -->中所述，我们将提取一个名为 `run` 的函数，该函数将保存 `main` 函数中当前不涉及设置配置或处理错误的所有逻辑。完成后，`main` 函数将简洁且易于通过检查验证，我们将能够为所有其他逻辑编写测试。

代码清单 12-11 显示了提取 `run` 函数的小增量改进。

<Listing number="12-11" file-name="src/main.rs" caption="提取包含程序其余逻辑的 `run` 函数">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-11/src/main.rs:here}}
```

</Listing>

`run` 函数现在包含来自 `main` 的所有剩余逻辑，从读取文件开始。`run` 函数将 `Config` 实例作为参数。

<!-- Old headings. Do not remove or links may break. -->

<a id="returning-errors-from-the-run-function"></a>

#### 从 `run` 返回错误

将剩余的程序逻辑分离到 `run` 函数后，我们可以改进错误处理，就像我们在代码清单 12-9 中对 `Config::build` 所做的那样。`run` 函数将在出现问题时返回 `Result<T, E>`，而不是通过调用 `expect` 允许程序 panic。这将使我们能够进一步将围绕处理错误的逻辑整合到 `main` 中，以用户友好的方式。代码清单 12-12 显示我们需要对 `run` 的签名和主体进行的更改。

<Listing number="12-12" file-name="src/main.rs" caption="将 `run` 函数更改为返回 `Result`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-12/src/main.rs:here}}
```

</Listing>

我们在这里进行了三个重大更改。首先，我们将 `run` 函数的返回类型更改为 `Result<(), Box<dyn Error>>`。此函数以前返回单元类型 `()`，我们将其保留为 `Ok` 情况中返回的值。

对于错误类型，我们使用了 trait 对象 `Box<dyn Error>`（我们在顶部使用 `use` 语句将 `std::error::Error` 引入作用域）。我们将在[第 18 章][ch18]<!-- ignore -->中介绍 trait 对象。现在，只需知道 `Box<dyn Error>` 意味着函数将返回实现 `Error` trait 的类型，但我们不必指定返回值将是什么特定类型。这使我们能够灵活地返回在不同错误情况下可能不同类型的错误值。`dyn` 关键字是 _dynamic_ 的缩写。

其次，我们移除了对 `expect` 的调用，改为使用 `?` 操作符，正如我们在[第 9 章][ch9-question-mark]<!-- ignore -->中讨论的。`?` 不会在错误时 `panic!`，而是从当前函数返回错误值供调用者处理。

第三，`run` 函数现在在成功情况下返回 `Ok` 值。我们在签名中将 `run` 函数的成功类型声明为 `()`，这意味着我们需要将单元类型值包装在 `Ok` 值中。这个 `Ok(())` 语法可能一开始看起来有点奇怪。但像这样使用 `()` 是习惯用法，表示我们仅为了副作用而调用 `run`；它不返回我们需要 to 的值。

当你运行此代码时，它将编译但会显示警告：

```console
{{#include ../listings/ch12-an-io-project/listing-12-12/output.txt}}
```

Rust 告诉我们代码忽略了 `Result` 值，并且 `Result` 值可能指示发生了错误。但我们没有检查是否有错误，编译器提醒我们可能想在这里有一些错误处理代码！让我们现在纠正这个问题。

#### 在 `main` 中处理从 `run` 返回的错误

我们将检查错误并使用类似于我们在代码清单 12-10 中对 `Config::build` 使用的技术来处理它们，但略有不同：

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/no-listing-01-handling-errors-in-main/src/main.rs:here}}
```

我们使用 `if let` 而不是 `unwrap_or_else` 来检查 `run` 是否返回 `Err` 值，如果是，则调用 `process::exit(1)`。`run` 函数不会返回我们想要像 `Config::build` 返回 `Config` 实例那样 `unwrap` 的值。因为 `run` 在成功情况下返回 `()`，我们只关心检测错误，所以我们不需要 `unwrap_or_else` 来返回解包的值，这只会是 `()`。

`if let` 和 `unwrap_or_else` 函数的主体在两种情况下都是相同的：我们打印错误并退出。

### 将代码拆分为库 Crate

我们的 `minigrep` 项目到目前为止看起来不错！现在我们将拆分 _src/main.rs_ 文件并将一些代码放入 _src/lib.rs_ 文件中。这样，我们可以测试代码，并拥有一个职责更少的 _src/main.rs_ 文件。

让我们在 _src/lib.rs_ 中定义负责搜索文本的代码，而不是在 _src/main.rs_ 中，这将让我们（或使用我们 `minigrep` 库的任何人）从比我们的 `minigrep` 二进制文件更多的上下文中调用搜索函数。

首先，让我们在 _src/lib.rs_ 中定义 `search` 函数签名，如代码清单 12-13 所示，主体调用 `unimplemented!` 宏。当我们填充实现时，我们将更详细地解释签名。

<Listing number="12-13" file-name="src/lib.rs" caption="在 *src/lib.rs* 中定义 `search` 函数">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-13/src/lib.rs}}
```

</Listing>

我们在函数定义上使用了 `pub` 关键字，将 `search` 指定为我们库 crate 公共 API 的一部分。我们现在有一个可以从二进制 crate 使用并且可以测试的库 crate！

现在我们需要将 _src/lib.rs_ 中定义的代码引入二进制 crate 的作用域（在 _src/main.rs_ 中）并调用它，如代码清单 12-14 所示。

<Listing number="12-14" file-name="src/main.rs" caption="在 *src/main.rs* 中使用 `minigrep` 库 crate 的 `search` 函数">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-14/src/main.rs:here}}
```

</Listing>

我们添加一个 `use minigrep::search` 行，将 `search` 函数从库 crate 引入二进制 crate 的作用域。然后，在 `run` 函数中，我们不是打印文件的内容，而是调用 `search` 函数并传递 `config.query` 值和 `contents` 作为参数。然后，`run` 将使用 `for` 循环打印从 `search` 返回的每个与查询匹配的行。这也是删除 `main` 函数中显示查询和文件路径的 `println!` 调用的好时机，以便我们的程序只打印搜索结果（如果不发生错误）。

注意，搜索函数将在任何打印发生之前将所有结果收集到它返回的向量中。在搜索大文件时，此实现可能显示结果很慢，因为结果不会在找到时打印；我们将在第 13 章讨论使用迭代器修复此问题的可能方法。

哇！这有很多工作，但我们已经为未来的成功奠定了基础。现在处理错误要容易得多，我们已经使代码更加模块化。从现在开始，我们几乎所有的工作都将在 _src/lib.rs_ 中完成。

让我们利用这种新发现的模块化来做一些用旧代码很难但用新代码很容易的事情：我们将编写一些测试！

[ch13]: ch13-00-functional-features.html
[ch9-custom-types]: ch09-03-to-panic-or-not-to-panic.html#creating-custom-types-for-validation
[ch9-error-guidelines]: ch09-03-to-panic-or-not-to-panic.html#guidelines-for-error-handling
[ch9-result]: ch09-02-recoverable-errors-with-result.html
[ch18]: ch18-00-oop.html
[ch9-question-mark]: ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator
