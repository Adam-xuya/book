## 改进我们的 I/O 项目

有了关于迭代器的新知识，我们可以通过使用迭代器使代码中的位置更清晰和简洁来改进第 12 章中的 I/O 项目。让我们看看迭代器如何改进我们对 `Config::build` 函数和 `search` 函数的实现。

### 使用迭代器移除 `clone`

在代码清单 12-6 中，我们添加了接受 `String` 值切片并通过索引到切片并克隆值来创建 `Config` 结构体实例的代码，允许 `Config` 结构体拥有这些值。在代码清单 13-17 中，我们复制了 `Config::build` 函数的实现，就像它在代码清单 12-23 中一样。

<Listing number="13-17" file-name="src/main.rs" caption="代码清单 12-23 中 `Config::build` 函数的复制">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-23-reproduced/src/main.rs:ch13}}
```

</Listing>

当时，我们说不要担心低效的 `clone` 调用，因为我们将来会移除它们。好吧，现在是时候了！

我们需要在这里使用 `clone`，因为我们在参数 `args` 中有一个包含 `String` 元素的切片，但 `build` 函数不拥有 `args`。为了返回 `Config` 实例的所有权，我们必须克隆 `Config` 的 `query` 和 `file_path` 字段中的值，以便 `Config` 实例可以拥有其值。

有了关于迭代器的新知识，我们可以将 `build` 函数更改为获取迭代器的所有权作为其参数，而不是借用切片。我们将使用迭代器功能而不是检查切片长度并索引到特定位置的代码。这将澄清 `Config::build` 函数正在做什么，因为迭代器将访问值。

一旦 `Config::build` 获取迭代器的所有权并停止使用借用的索引操作，我们就可以将 `String` 值从迭代器移动到 `Config`，而不是调用 `clone` 并进行新分配。

#### 直接使用返回的迭代器

打开你的 I/O 项目的 _src/main.rs_ 文件，它应该看起来像这样：

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-24-reproduced/src/main.rs:ch13}}
```

我们首先将我们在代码清单 12-24 中拥有的 `main` 函数的开始更改为代码清单 13-18 中的代码，这次使用迭代器。这在我们也更新 `Config::build` 之前不会编译。

<Listing number="13-18" file-name="src/main.rs" caption="将 `env::args` 的返回值传递给 `Config::build`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-18/src/main.rs:here}}
```

</Listing>

`env::args` 函数返回一个迭代器！我们现在直接将 `env::args` 返回的迭代器的所有权传递给 `Config::build`，而不是将迭代器值收集到向量中然后传递切片给 `Config::build`。

接下来，我们需要更新 `Config::build` 的定义。让我们将 `Config::build` 的签名更改为看起来像代码清单 13-19。这仍然不会编译，因为我们需要更新函数主体。

<Listing number="13-19" file-name="src/main.rs" caption="更新 `Config::build` 的签名以期望迭代器">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-19/src/main.rs:here}}
```

</Listing>

`env::args` 函数的标准库文档显示它返回的迭代器类型是 `std::env::Args`，该类型实现 `Iterator` trait 并返回 `String` 值。

我们已经更新了 `Config::build` 函数的签名，以便参数 `args` 具有带有 trait 约束 `impl Iterator<Item = String>` 的泛型类型，而不是 `&[String]`。我们在第 10 章的["使用 Traits 作为参数"][impl-trait]<!-- ignore -->部分讨论的 `impl Trait` 语法的这种用法意味着 `args` 可以是实现 `Iterator` trait 并返回 `String` 项的任何类型。

因为我们要获取 `args` 的所有权，并且我们将通过迭代它来改变 `args`，我们可以在 `args` 参数的规范中添加 `mut` 关键字以使其可变。

<!-- Old headings. Do not remove or links may break. -->

<a id="using-iterator-trait-methods-instead-of-indexing"></a>

#### 使用 `Iterator` Trait 方法

接下来，我们将修复 `Config::build` 的主体。因为 `args` 实现 `Iterator` trait，我们知道可以在它上调用 `next` 方法！代码清单 13-20 更新了代码清单 12-23 中的代码以使用 `next` 方法。

<Listing number="13-20" file-name="src/main.rs" caption="更改 `Config::build` 的主体以使用迭代器方法">

```rust,ignore,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-20/src/main.rs:here}}
```

</Listing>

记住，`env::args` 返回值中的第一个值是程序的名称。我们想忽略它并获取下一个值，所以我们首先调用 `next` 并且不对返回值做任何事情。然后，我们调用 `next` 以获取我们想要放入 `Config` 的 `query` 字段的值。如果 `next` 返回 `Some`，我们使用 `match` 提取值。如果它返回 `None`，这意味着没有给出足够的参数，我们提前返回 `Err` 值。我们对 `file_path` 值做同样的事情。

<!-- Old headings. Do not remove or links may break. -->

<a id="making-code-clearer-with-iterator-adapters"></a>

### 使用迭代器适配器澄清代码

我们还可以在 I/O 项目中的 `search` 函数中利用迭代器，这里在代码清单 13-21 中复制了它，就像它在代码清单 12-19 中一样。

<Listing number="13-21" file-name="src/lib.rs" caption="代码清单 12-19 中 `search` 函数的实现">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:ch13}}
```

</Listing>

我们可以使用迭代器适配器方法以更简洁的方式编写此代码。这样做也让我们避免拥有可变中间 `results` 向量。函数式编程风格更喜欢最小化可变状态的数量以使代码更清晰。移除可变状态可能使未来的增强能够并行进行搜索，因为我们不必管理对 `results` 向量的并发访问。代码清单 13-22 显示了此更改。

<Listing number="13-22" file-name="src/lib.rs" caption="在 `search` 函数的实现中使用迭代器适配器方法">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-22/src/lib.rs:here}}
```

</Listing>

回想一下，`search` 函数的目的是返回 `contents` 中包含 `query` 的所有行。类似于代码清单 13-16 中的 `filter` 示例，此代码使用 `filter` 适配器只保留 `line.contains(query)` 返回 `true` 的行。然后，我们使用 `collect` 将匹配的行收集到另一个向量中。简单多了！随意进行相同的更改，在 `search_case_insensitive` 函数中也使用迭代器方法。

为了进一步改进，通过移除对 `collect` 的调用并将返回类型更改为 `impl Iterator<Item = &'a str>`，从 `search` 函数返回迭代器，以便函数成为迭代器适配器。注意，你也需要更新测试！在进行此更改之前和之后使用 `minigrep` 工具搜索大文件，以观察行为的差异。在进行此更改之前，程序在收集所有结果之前不会打印任何结果，但在此更改之后，结果将在找到每个匹配行时打印，因为 `run` 函数中的 `for` 循环能够利用迭代器的惰性。

<!-- Old headings. Do not remove or links may break. -->

<a id="choosing-between-loops-or-iterators"></a>

### 在循环和迭代器之间选择

下一个逻辑问题是你应该在自己的代码中选择哪种风格以及为什么：代码清单 13-21 中的原始实现或代码清单 13-22 中使用迭代器的版本（假设我们在返回之前收集所有结果而不是返回迭代器）。大多数 Rust 程序员更喜欢使用迭代器风格。一开始有点难掌握，但一旦你感受到各种迭代器适配器及其作用，迭代器可能更容易理解。而不是摆弄循环和构建新向量的各种部分，代码专注于循环的高级目标。这抽象了一些常见代码，以便更容易看到此代码独有的概念，例如迭代器中的每个元素必须通过的过滤条件。

但是这两种实现真的等价吗？直观的假设可能是较低级别的循环会更快。让我们谈谈性能。

[impl-trait]: ch10-02-traits.html#traits-as-parameters
