<!-- Old headings. Do not remove or links may break. -->
<a id="developing-the-librarys-functionality-with-test-driven-development"></a>

## 使用测试驱动开发添加功能

既然我们已经将搜索逻辑在 _src/lib.rs_ 中与 `main` 函数分离，编写代码核心功能的测试要容易得多。我们可以直接使用各种参数调用函数并检查返回值，而不必从命令行调用我们的二进制文件。

在本节中，我们将使用测试驱动开发（TDD）过程将搜索逻辑添加到 `minigrep` 程序，步骤如下：

1. 编写一个失败的测试并运行它以确保它因你预期的原因而失败。
2. 编写或修改足够的代码以使新测试通过。
3. 重构你刚刚添加或更改的代码，并确保测试继续通过。
4. 从步骤 1 重复！

虽然这只是编写软件的许多方法之一，但 TDD 可以帮助驱动代码设计。在编写使测试通过的代码之前编写测试有助于在整个过程中保持高测试覆盖率。

我们将测试驱动实现在文件内容中实际搜索查询字符串并生成匹配查询的行列表的功能。我们将在名为 `search` 的函数中添加此功能。

### 编写失败的测试

在 _src/lib.rs_ 中，我们将添加一个 `tests` 模块和一个测试函数，就像我们在[第 11 章][ch11-anatomy]<!-- ignore -->中所做的那样。测试函数指定我们希望 `search` 函数具有的行为：它将接受查询和要搜索的文本，并只返回包含查询的文本行。代码清单 12-15 显示了此测试。

<Listing number="12-15" file-name="src/lib.rs" caption="为 `search` 函数创建一个失败的测试，用于我们希望拥有的功能">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-15/src/lib.rs:here}}
```

</Listing>

此测试搜索字符串 `"duct"`。我们正在搜索的文本是三行，其中只有一行包含 `"duct"`（注意开始双引号后的反斜杠告诉 Rust 不要在此字符串字面量内容的开头放置换行符）。我们断言从 `search` 函数返回的值只包含我们期望的行。

如果我们运行此测试，它目前会失败，因为 `unimplemented!` 宏会 panic，消息为"not implemented"。根据 TDD 原则，我们将采取一个小步骤，添加足够的代码，通过定义 `search` 函数始终返回空向量来使测试在调用函数时不 panic，如代码清单 12-16 所示。然后，测试应该编译并失败，因为空向量不匹配包含行 `"safe, fast, productive."` 的向量。

<Listing number="12-16" file-name="src/lib.rs" caption="定义足够的 `search` 函数，以便调用它不会 panic">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-16/src/lib.rs:here}}
```

</Listing>

现在让我们讨论为什么我们需要在 `search` 的签名中定义显式生命周期 `'a`，并在 `contents` 参数和返回值中使用该生命周期。回想一下[第 10 章][ch10-lifetimes]<!-- ignore -->，生命周期参数指定哪个参数生命周期与返回值的生命周期相关联。在这种情况下，我们指示返回的向量应包含引用参数 `contents` 的切片（而不是参数 `query`）的字符串切片。

换句话说，我们告诉 Rust，`search` 函数返回的数据将存活与在 `contents` 参数中传递给 `search` 函数的数据一样长。这很重要！切片 _引用的_ 数据需要有效，引用才能有效；如果编译器假设我们正在创建 `query` 而不是 `contents` 的字符串切片，它将错误地进行安全检查。

如果我们忘记生命周期注释并尝试编译此函数，我们将得到此错误：

```console
{{#include ../listings/ch12-an-io-project/output-only-02-missing-lifetimes/output.txt}}
```

Rust 无法知道我们需要两个参数中的哪一个用于输出，所以我们需要明确告诉它。注意，帮助文本建议为所有参数和输出类型指定相同的生命周期参数，这是不正确的！因为 `contents` 是包含我们所有文本的参数，我们想要返回匹配该文本的部分，我们知道 `contents` 是应该使用生命周期语法连接到返回值的唯一参数。

其他编程语言不要求你在签名中将参数连接到返回值，但随着时间的推移，这种做法会变得更容易。你可能想将此示例与第 10 章的["使用生命周期验证引用"][validating-references-with-lifetimes]<!-- ignore -->部分中的示例进行比较。

### 编写代码以使测试通过

目前，我们的测试失败是因为我们总是返回空向量。要修复它并实现 `search`，我们的程序需要遵循以下步骤：

1. 遍历内容的每一行。
2. 检查该行是否包含我们的查询字符串。
3. 如果包含，将其添加到我们返回的值列表中。
4. 如果不包含，不执行任何操作。
5. 返回匹配的结果列表。

让我们逐步完成每个步骤，从遍历行开始。

#### 使用 `lines` 方法遍历行

Rust 有一个有用的方法来处理字符串的逐行迭代，方便地命名为 `lines`，如代码清单 12-17 所示。注意，这还不能编译。

<Listing number="12-17" file-name="src/lib.rs" caption="遍历 `contents` 中的每一行">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-17/src/lib.rs:here}}
```

</Listing>

`lines` 方法返回一个迭代器。我们将在[第 13 章][ch13-iterators]<!-- ignore -->中深入讨论迭代器。但回想一下，你在[代码清单 3-5][ch3-iter]<!-- ignore -->中看到了这种使用迭代器的方式，我们在那里使用带有迭代器的 `for` 循环在集合中的每个项上运行一些代码。

#### 在每行中搜索查询

接下来，我们将检查当前行是否包含我们的查询字符串。幸运的是，字符串有一个有用的方法 `contains`，它为我们完成此操作！在 `search` 函数中添加对 `contains` 方法的调用，如代码清单 12-18 所示。注意，这仍然不能编译。

<Listing number="12-18" file-name="src/lib.rs" caption="添加功能以查看行是否包含 `query` 中的字符串">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-18/src/lib.rs:here}}
```

</Listing>

目前，我们正在构建功能。为了使代码编译，我们需要从主体返回一个值，正如我们在函数签名中表明的那样。

#### 存储匹配的行

要完成此函数，我们需要一种方法来存储我们想要返回的匹配行。为此，我们可以在 `for` 循环之前创建一个可变向量，并调用 `push` 方法将 `line` 存储在向量中。在 `for` 循环之后，我们返回向量，如代码清单 12-19 所示。

<Listing number="12-19" file-name="src/lib.rs" caption="存储匹配的行，以便我们可以返回它们">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:here}}
```

</Listing>

现在 `search` 函数应该只返回包含 `query` 的行，我们的测试应该通过。让我们运行测试：

```console
{{#include ../listings/ch12-an-io-project/listing-12-19/output.txt}}
```

我们的测试通过了，所以我们知道它有效！

此时，我们可以考虑在保持测试通过以保持相同功能的同时重构搜索函数实现的机会。搜索函数中的代码不是太糟糕，但它没有利用迭代器的一些有用功能。我们将在[第 13 章][ch13-iterators]<!-- ignore -->中回到这个示例，我们将在那里详细探索迭代器，并看看如何改进它。

现在整个程序应该工作了！让我们试试看，首先用一个应该从 Emily Dickinson 诗中返回恰好一行的单词：_frog_。

```console
{{#include ../listings/ch12-an-io-project/no-listing-02-using-search-in-run/output.txt}}
```

太棒了！现在让我们尝试一个会匹配多行的单词，比如 _body_：

```console
{{#include ../listings/ch12-an-io-project/output-only-03-multiple-matches/output.txt}}
```

最后，让我们确保当我们在诗中搜索任何地方都不存在的单词（如 _monomorphization_）时不会得到任何行：

```console
{{#include ../listings/ch12-an-io-project/output-only-04-no-matches/output.txt}}
```

太好了！我们已经构建了自己的经典工具的迷你版本，并学到了很多关于如何构建应用程序的知识。我们还学到了一些关于文件输入和输出、生命周期、测试和命令行解析的知识。

为了完善这个项目，我们将简要演示如何使用环境变量以及如何打印到标准错误，这两者在编写命令行程序时都很有用。

[validating-references-with-lifetimes]: ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[ch11-anatomy]: ch11-01-writing-tests.html#the-anatomy-of-a-test-function
[ch10-lifetimes]: ch10-03-lifetime-syntax.html
[ch3-iter]: ch03-05-control-flow.html#looping-through-a-collection-with-for
[ch13-iterators]: ch13-02-iterators.html
