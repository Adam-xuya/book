## 如何编写测试

_测试_ 是 Rust 函数，用于验证非测试代码是否按预期方式运行。测试函数的主体通常执行这三个操作：

- 设置任何需要的数据或状态。
- 运行你想要测试的代码。
- 断言结果是预期的结果。

让我们看看 Rust 专门为编写执行这些操作的测试而提供的功能，包括 `test` 属性、一些宏和 `should_panic` 属性。

<!-- Old headings. Do not remove or links may break. -->

<a id="the-anatomy-of-a-test-function"></a>

### 构建测试函数

最简单的，Rust 中的测试是一个用 `test` 属性注释的函数。属性是关于 Rust 代码片段的元数据；一个例子是我们在第 5 章中与结构体一起使用的 `derive` 属性。要将函数转换为测试函数，请在 `fn` 之前的行上添加 `#[test]`。当你使用 `cargo test` 命令运行测试时，Rust 构建一个测试运行器二进制文件，该文件运行注释的函数并报告每个测试函数是通过还是失败。

每当我们使用 Cargo 创建新的库项目时，会自动为我们生成一个包含测试函数的测试模块。此模块为你提供了编写测试的模板，这样你就不必在每次启动新项目时查找确切的结构和语法。你可以添加任意数量的额外测试函数和任意数量的测试模块！

在我们实际测试任何代码之前，我们将通过实验模板测试来探索测试工作的一些方面。然后，我们将编写一些真实的测试，这些测试调用我们编写的一些代码并断言其行为是正确的。

让我们创建一个名为 `adder` 的新库项目，它将添加两个数字：

```console
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

你的 `adder` 库中的 _src/lib.rs_ 文件内容应该看起来像代码清单 11-1。

<Listing number="11-1" file-name="src/lib.rs" caption="由 `cargo new` 自动生成的代码">

<!-- manual-regeneration
cd listings/ch11-writing-automated-tests
rm -rf listing-11-01
cargo new listing-11-01 --lib --name adder
cd listing-11-01
echo "$ cargo test" > output.txt
RUSTFLAGS="-A unused_variables -A dead_code" RUST_TEST_THREADS=1 cargo test >> output.txt 2>&1
git diff output.txt # commit any relevant changes; discard irrelevant ones
cd ../../..
-->

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

</Listing>

文件以一个示例 `add` 函数开头，这样我们就有了要测试的东西。

现在，让我们只关注 `it_works` 函数。注意 `#[test]` 注释：此属性表示这是一个测试函数，因此测试运行器知道将此函数视为测试。我们也可能在 `tests` 模块中有非测试函数来帮助设置常见场景或执行常见操作，所以我们总是需要指示哪些函数是测试。

示例函数主体使用 `assert_eq!` 宏来断言 `result`（包含使用 2 和 2 调用 `add` 的结果）等于 4。此断言作为典型测试格式的示例。让我们运行它看看这个测试通过。

`cargo test` 命令运行我们项目中的所有测试，如代码清单 11-2 所示。

<Listing number="11-2" caption="运行自动生成的测试的输出">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-01/output.txt}}
```

</Listing>

Cargo 编译并运行了测试。我们看到行 `running 1 test`。下一行显示生成的测试函数的名称，称为 `tests::it_works`，运行该测试的结果是 `ok`。总体摘要 `test result: ok.` 意味着所有测试都通过了，读取 `1 passed; 0 failed` 的部分总计通过或失败的测试数量。

可以将测试标记为忽略，以便它不在特定实例中运行；我们将在本章后面的["除非特别请求否则忽略测试"][ignoring]<!-- ignore -->部分介绍这一点。因为我们在这里没有这样做，摘要显示 `0 ignored`。我们还可以向 `cargo test` 命令传递参数以仅运行名称与字符串匹配的测试；这称为 _过滤_，我们将在["按名称运行测试子集"][subset]<!-- ignore -->部分介绍它。在这里，我们没有过滤正在运行的测试，所以摘要的末尾显示 `0 filtered out`。

`0 measured` 统计信息用于测量性能的基准测试。基准测试在撰写本文时仅在 nightly Rust 中可用。有关基准测试的更多信息，请参阅[关于基准测试的文档][bench]。

从 `Doc-tests adder` 开始的测试输出的下一部分用于任何文档测试的结果。我们还没有任何文档测试，但 Rust 可以编译出现在我们 API 文档中的任何代码示例。此功能有助于保持文档和代码同步！我们将在第 14 章的["文档注释作为测试"][doc-comments]<!-- ignore -->部分讨论如何编写文档测试。现在，我们将忽略 `Doc-tests` 输出。

让我们开始根据我们自己的需求自定义测试。首先，将 `it_works` 函数的名称更改为不同的名称，例如 `exploration`，如下所示：

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/src/lib.rs}}
```

然后，再次运行 `cargo test`。输出现在显示 `exploration` 而不是 `it_works`：

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/output.txt}}
```

现在我们将添加另一个测试，但这次我们将创建一个失败的测试！当测试函数中的某些内容 panic 时，测试会失败。每个测试都在新线程中运行，当主线程看到测试线程已死亡时，测试被标记为失败。在第 9 章中，我们讨论了 panic 的最简单方法是调用 `panic!` 宏。输入名为 `another` 的新测试作为函数，因此你的 _src/lib.rs_ 文件看起来像代码清单 11-3。

<Listing number="11-3" file-name="src/lib.rs" caption="添加第二个测试，因为调用 `panic!` 宏而失败">

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-03/src/lib.rs}}
```

</Listing>

使用 `cargo test` 再次运行测试。输出应该看起来像代码清单 11-4，它显示我们的 `exploration` 测试通过而 `another` 失败。

<Listing number="11-4" caption="一个测试通过而一个测试失败时的测试结果">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-03/output.txt}}
```

</Listing>

<!-- manual-regeneration
rg panicked listings/ch11-writing-automated-tests/listing-11-03/output.txt
check the line number of the panic matches the line number in the following paragraph
 -->

不是 `ok`，行 `test tests::another` 显示 `FAILED`。在单个结果和摘要之间出现两个新部分：第一个显示每个测试失败的详细原因。在这种情况下，我们得到详细信息，`tests::another` 失败了，因为它在 _src/lib.rs_ 文件的第 17 行 panic，消息为 `Make this test fail`。下一部分仅列出所有失败测试的名称，当有很多测试和大量详细的失败测试输出时，这很有用。我们可以使用失败测试的名称来仅运行该测试以更容易地调试它；我们将在["控制测试运行方式"][controlling-how-tests-are-run]<!-- ignore -->部分讨论更多运行测试的方法。

摘要行在末尾显示：总体而言，我们的测试结果是 `FAILED`。我们有一个测试通过，一个测试失败。

既然你已经看到了不同场景中测试结果的样子，让我们看看除了 `panic!` 之外的一些在测试中有用的宏。

<!-- Old headings. Do not remove or links may break. -->

<a id="checking-results-with-the-assert-macro"></a>

### 使用 `assert!` 检查结果

标准库提供的 `assert!` 宏在你想确保测试中的某个条件评估为 `true` 时很有用。我们给 `assert!` 宏一个评估为布尔值的参数。如果值为 `true`，什么都不会发生，测试通过。如果值为 `false`，`assert!` 宏调用 `panic!` 导致测试失败。使用 `assert!` 宏有助于我们检查代码是否按预期方式运行。

在第 5 章代码清单 5-15 中，我们使用了一个 `Rectangle` 结构体和一个 `can_hold` 方法，它们在这里的代码清单 11-5 中重复。让我们将此代码放在 _src/lib.rs_ 文件中，然后使用 `assert!` 宏为它编写一些测试。

<Listing number="11-5" file-name="src/lib.rs" caption="来自第 5 章的 `Rectangle` 结构体及其 `can_hold` 方法">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-05/src/lib.rs}}
```

</Listing>

`can_hold` 方法返回布尔值，这意味着它是 `assert!` 宏的完美用例。在代码清单 11-6 中，我们编写一个测试，通过创建一个宽度为 8、高度为 7 的 `Rectangle` 实例来测试 `can_hold` 方法，并断言它可以容纳另一个宽度为 5、高度为 1 的 `Rectangle` 实例。

<Listing number="11-6" file-name="src/lib.rs" caption="`can_hold` 的测试，检查较大的矩形是否确实可以容纳较小的矩形">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-06/src/lib.rs:here}}
```

</Listing>

注意 `tests` 模块内的 `use super::*;` 行。`tests` 模块是一个常规模块，遵循我们在第 7 章的["在模块树中引用项的路径"][paths-for-referring-to-an-item-in-the-module-tree]<!-- ignore -->部分介绍的通常可见性规则。因为 `tests` 模块是一个内部模块，我们需要将外部模块中正在测试的代码引入内部模块的作用域。我们在这里使用 glob，所以我们在外部模块中定义的任何内容都可用于此 `tests` 模块。

我们将测试命名为 `larger_can_hold_smaller`，并创建了我们需要的两个 `Rectangle` 实例。然后，我们调用了 `assert!` 宏并向其传递了调用 `larger.can_hold(&smaller)` 的结果。此表达式应该返回 `true`，所以我们的测试应该通过。让我们看看！

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-06/output.txt}}
```

它确实通过了！让我们添加另一个测试，这次断言较小的矩形不能容纳较大的矩形：

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/src/lib.rs:here}}
```

因为在这种情况下 `can_hold` 函数的正确结果是 `false`，我们需要在将其传递给 `assert!` 宏之前否定该结果。因此，如果 `can_hold` 返回 `false`，我们的测试将通过：

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/output.txt}}
```

两个测试都通过了！现在让我们看看当我们在代码中引入 bug 时测试结果会发生什么。我们将通过将比较宽度时的大于号（`>`）替换为小于号（`<`）来更改 `can_hold` 方法的实现：

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/src/lib.rs:here}}
```

现在运行测试会产生以下结果：

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/output.txt}}
```

我们的测试捕获了 bug！因为 `larger.width` 是 `8` 而 `smaller.width` 是 `5`，`can_hold` 中宽度的比较现在返回 `false`：8 不小于 5。

<!-- Old headings. Do not remove or links may break. -->

<a id="testing-equality-with-the-assert_eq-and-assert_ne-macros"></a>

### 使用 `assert_eq!` 和 `assert_ne!` 测试相等性

验证功能的常见方法是测试正在测试的代码的结果与你期望代码返回的值之间的相等性。你可以通过使用 `assert!` 宏并向其传递使用 `==` 操作符的表达式来做到这一点。但是，这是一个如此常见的测试，以至于标准库提供了一对宏——`assert_eq!` 和 `assert_ne!`——来更方便地执行此测试。这些宏分别比较两个参数是否相等或不相等。如果断言失败，它们还会打印两个值，这使得更容易看到测试失败的原因；相反，`assert!` 宏仅指示它获得了 `==` 表达式的 `false` 值，而不打印导致 `false` 值的值。

在代码清单 11-7 中，我们编写一个名为 `add_two` 的函数，它将 `2` 添加到其参数，然后使用 `assert_eq!` 宏测试此函数。

<Listing number="11-7" file-name="src/lib.rs" caption="使用 `assert_eq!` 宏测试函数 `add_two`">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-07/src/lib.rs}}
```

</Listing>

让我们检查它是否通过！

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-07/output.txt}}
```

我们创建一个名为 `result` 的变量，它保存调用 `add_two(2)` 的结果。然后，我们将 `result` 和 `4` 作为参数传递给 `assert_eq!` 宏。此测试的输出行是 `test tests::it_adds_two ... ok`，`ok` 文本表示我们的测试通过了！

让我们在我们的代码中引入一个 bug，看看 `assert_eq!` 在失败时是什么样子。将 `add_two` 函数的实现更改为添加 `3`：

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/src/lib.rs:here}}
```

再次运行测试：

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/output.txt}}
```

我们的测试捕获了 bug！`tests::it_adds_two` 测试失败，消息告诉我们失败的断言是 `left == right` 以及 `left` 和 `right` 值是什么。此消息帮助我们开始调试：`left` 参数，我们有调用 `add_two(2)` 的结果，是 `5`，但 `right` 参数是 `4`。你可以想象，当我们有很多测试在进行时，这将特别有用。

注意，在某些语言和测试框架中，相等性断言函数的参数称为 `expected` 和 `actual`，我们指定参数的顺序很重要。但是，在 Rust 中，它们被称为 `left` 和 `right`，我们指定期望值和代码产生的值的顺序并不重要。我们可以将测试中的断言写为 `assert_eq!(4, result)`，这将导致相同的失败消息，显示 `` assertion `left == right` failed ``。

`assert_ne!` 宏如果我们给它的两个值不相等，它将通过，如果它们相等，它将失败。当我们不确定值 _将_ 是什么但我们知道值肯定 _不应该_ 是什么时，此宏最有用。例如，如果我们正在测试一个保证以某种方式更改其输入的函数，但输入更改的方式取决于我们运行测试的星期几，最好的断言可能是函数的输出不等于输入。

在底层，`assert_eq!` 和 `assert_ne!` 宏分别使用 `==` 和 `!=` 操作符。当断言失败时，这些宏使用调试格式打印它们的参数，这意味着被比较的值必须实现 `PartialEq` 和 `Debug` trait。所有原始类型和大多数标准库类型都实现了这些 trait。对于你自己定义的结构体和枚举，你需要实现 `PartialEq` 来断言这些类型的相等性。你还需要实现 `Debug` 以便在断言失败时打印值。因为这两个 trait 都是可派生 trait，如第 5 章代码清单 5-12 中提到的，这通常就像在你的结构体或枚举定义中添加 `#[derive(PartialEq, Debug)]` 注释一样简单。有关这些和其他可派生 trait 的更多详细信息，请参阅附录 C，["可派生 Trait"][derivable-traits]<!-- ignore -->。

### 添加自定义失败消息

你还可以将自定义消息添加到 `assert!`、`assert_eq!` 和 `assert_ne!` 宏的可选参数中，以便与失败消息一起打印。在必需参数之后指定的任何参数都会传递给 `format!` 宏（在第 8 章的["使用 `+` 或 `format!` 连接"][concatenating]<!-- ignore -->中讨论），所以你可以传递一个包含 `{}` 占位符和要放入这些占位符的值的格式字符串。自定义消息对于记录断言的含义很有用；当测试失败时，你将更好地了解代码的问题所在。

例如，假设我们有一个通过名称问候人们的函数，我们想测试我们传递给函数的名称是否出现在输出中：

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-05-greeter/src/lib.rs}}
```

此程序的要求尚未达成一致，我们相当确定问候语开头的 `Hello` 文本会改变。我们决定不想在要求更改时更新测试，所以我们不检查与从 `greeting` 函数返回的值的完全相等，而是断言输出包含输入参数的文本。

现在让我们通过更改 `greeting` 以排除 `name` 来在此代码中引入 bug，看看默认测试失败是什么样子：

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/src/lib.rs:here}}
```

运行此测试会产生以下结果：

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/output.txt}}
```

此结果仅指示断言失败以及断言所在的行。更有用的失败消息将打印来自 `greeting` 函数的值。让我们添加一个自定义失败消息，该消息由格式字符串组成，占位符填充了我们从 `greeting` 函数获得的实际值：

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/src/lib.rs:here}}
```

现在当我们运行测试时，我们将得到一个更有信息的错误消息：

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/output.txt}}
```

我们可以在测试输出中看到我们实际获得的值，这将帮助我们调试发生的情况而不是我们期望发生的事情。

### 使用 `should_panic` 检查 Panic

除了检查返回值之外，检查我们的代码是否按预期处理错误条件也很重要。例如，考虑我们在第 9 章代码清单 9-13 中创建的 `Guess` 类型。使用 `Guess` 的其他代码依赖于 `Guess` 实例将仅包含 1 到 100 之间的值的保证。我们可以编写一个测试，确保尝试创建具有该范围之外的值的 `Guess` 实例会 panic。

我们通过将 `should_panic` 属性添加到测试函数来做到这一点。如果函数内的代码 panic，测试通过；如果函数内的代码不 panic，测试失败。

代码清单 11-8 显示了一个测试，它检查当我们期望它们发生时，`Guess::new` 的错误条件是否发生。

<Listing number="11-8" file-name="src/lib.rs" caption="测试条件将导致 `panic!`">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-08/src/lib.rs}}
```

</Listing>

我们将 `#[should_panic]` 属性放在 `#[test]` 属性之后，并在它应用的测试函数之前。让我们看看这个测试通过时的结果：

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-08/output.txt}}
```

看起来不错！现在让我们通过删除如果值大于 100 时 `new` 函数将 panic 的条件来在我们的代码中引入 bug：

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/src/lib.rs:here}}
```

当我们运行代码清单 11-8 中的测试时，它将失败：

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/output.txt}}
```

在这种情况下，我们没有得到非常有用的消息，但是当我们查看测试函数时，我们看到它用 `#[should_panic]` 注释。我们得到的失败意味着测试函数中的代码没有导致 panic。

使用 `should_panic` 的测试可能不精确。即使测试因与我们预期的原因不同的原因而 panic，`should_panic` 测试也会通过。为了使 `should_panic` 测试更精确，我们可以向 `should_panic` 属性添加可选的 `expected` 参数。测试工具将确保失败消息包含提供的文本。例如，考虑代码清单 11-9 中 `Guess` 的修改代码，其中 `new` 函数根据值太小还是太大而 panic 不同的消息。

<Listing number="11-9" file-name="src/lib.rs" caption="测试带有包含指定子字符串的 panic 消息的 `panic!`">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-09/src/lib.rs:here}}
```

</Listing>

此测试将通过，因为我们放在 `should_panic` 属性的 `expected` 参数中的值是 `Guess::new` 函数 panic 的消息的子字符串。我们可以指定我们期望的整个 panic 消息，在这种情况下是 `Guess value must be less than or equal to 100, got 200`。你选择指定什么取决于 panic 消息的多少是唯一的或动态的，以及你希望测试的精确程度。在这种情况下，panic 消息的子字符串足以确保测试函数中的代码执行 `else if value > 100` 情况。

要查看带有 `expected` 消息的 `should_panic` 测试失败时会发生什么，让我们通过交换 `if value < 1` 和 `else if value > 100` 块的主体再次在我们的代码中引入 bug：

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/src/lib.rs:here}}
```

这次当我们运行 `should_panic` 测试时，它将失败：

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/output.txt}}
```

失败消息表明此测试确实按我们预期的那样 panic，但 panic 消息不包含预期的字符串 `less than or equal to 100`。我们在这种情况下得到的 panic 消息是 `Guess value must be greater than or equal to 1, got 200`。现在我们可以开始找出我们的 bug 在哪里！

### 在测试中使用 `Result<T, E>`

到目前为止，我们所有的测试在失败时都会 panic。我们也可以编写使用 `Result<T, E>` 的测试！这是代码清单 11-1 中的测试，重写为使用 `Result<T, E>` 并返回 `Err` 而不是 panic：

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-10-result-in-tests/src/lib.rs:here}}
```

`it_works` 函数现在具有 `Result<(), String>` 返回类型。在函数主体中，我们不是调用 `assert_eq!` 宏，而是在测试通过时返回 `Ok(())`，在测试失败时返回包含 `String` 的 `Err`。

编写测试以便它们返回 `Result<T, E>` 使你可以使用测试主体中的问号操作符，这可能是编写如果其中任何操作返回 `Err` 变体应该失败的测试的便捷方式。

你不能在使用 `Result<T, E>` 的测试上使用 `#[should_panic]` 注释。要断言操作返回 `Err` 变体，_不要_ 在 `Result<T, E>` 值上使用问号操作符。相反，使用 `assert!(value.is_err())`。

既然你知道几种编写测试的方法，让我们看看当我们运行测试时会发生什么，并探索我们可以与 `cargo test` 一起使用的不同选项。

[concatenating]: ch08-02-strings.html#concatenating-with--or-format
[bench]: ../unstable-book/library-features/test.html
[ignoring]: ch11-02-running-tests.html#ignoring-tests-unless-specifically-requested
[subset]: ch11-02-running-tests.html#running-a-subset-of-tests-by-name
[controlling-how-tests-are-run]: ch11-02-running-tests.html#controlling-how-tests-are-run
[derivable-traits]: appendix-03-derivable-traits.html
[doc-comments]: ch14-02-publishing-to-crates-io.html#documentation-comments-as-tests
[paths-for-referring-to-an-item-in-the-module-tree]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
