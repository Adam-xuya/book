## 测试组织

如本章开头提到的，测试是一门复杂的学科，不同的人使用不同的术语和组织。Rust 社区从两个主要类别来考虑测试：单元测试和集成测试。_单元测试_ 小而更专注，一次隔离测试一个模块，可以测试私有接口。_集成测试_ 完全在你的库外部，并使用你的代码，就像任何其他外部代码会使用它一样，仅使用公共接口，并可能每个测试练习多个模块。

编写这两种测试对于确保库的各个部分单独和一起按预期工作非常重要。

### 单元测试

单元测试的目的是从其余代码中隔离测试每个代码单元，以快速定位代码按预期工作和不按预期工作的位置。你将单元测试放在 _src_ 目录中，在每个文件中与它们正在测试的代码一起。约定是在每个文件中创建一个名为 `tests` 的模块来包含测试函数，并用 `cfg(test)` 注释该模块。

#### `tests` 模块和 `#[cfg(test)]`

`tests` 模块上的 `#[cfg(test)]` 注释告诉 Rust 仅在你运行 `cargo test` 时编译和运行测试代码，而不是在你运行 `cargo build` 时。这在你只想构建库时节省编译时间，并在生成的编译工件中节省空间，因为测试未包含在内。你将看到，因为集成测试进入不同的目录，它们不需要 `#[cfg(test)]` 注释。但是，因为单元测试与代码进入相同的文件，你将使用 `#[cfg(test)]` 来指定它们不应包含在编译结果中。

回想一下，当我们在本章第一部分生成新的 `adder` 项目时，Cargo 为我们生成了此代码：

<span class="filename">Filename: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

在自动生成的 `tests` 模块上，属性 `cfg` 代表 _配置_，告诉 Rust 只有在给定特定配置选项时才应包含以下项。在这种情况下，配置选项是 `test`，由 Rust 提供用于编译和运行测试。通过使用 `cfg` 属性，Cargo 仅在我们主动使用 `cargo test` 运行测试时编译我们的测试代码。这包括可能在此模块内的任何辅助函数，以及用 `#[test]` 注释的函数。

<!-- Old headings. Do not remove or links may break. -->

<a id="testing-private-functions"></a>

#### 私有函数测试

测试社区内部关于是否应该直接测试私有函数存在争议，其他语言使测试私有函数变得困难或不可能。无论你坚持哪种测试理念，Rust 的隐私规则确实允许你测试私有函数。考虑代码清单 11-12 中的代码，其中包含私有函数 `internal_adder`。

<Listing number="11-12" file-name="src/lib.rs" caption="测试私有函数">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-12/src/lib.rs}}
```

</Listing>

注意，`internal_adder` 函数未标记为 `pub`。测试只是 Rust 代码，`tests` 模块只是另一个模块。正如我们在["在模块树中引用项的路径"][paths]<!-- ignore -->中讨论的，子模块中的项可以使用其祖先模块中的项。在此测试中，我们使用 `use super::*` 将所有属于 `tests` 模块父级的项引入作用域，然后测试可以调用 `internal_adder`。如果你认为不应该测试私有函数，Rust 中没有任何东西会强制你这样做。

### 集成测试

在 Rust 中，集成测试完全在你的库外部。它们以任何其他代码会使用它的相同方式使用你的库，这意味着它们只能调用作为库公共 API 一部分的函数。它们的目的是测试库的许多部分是否正确地协同工作。单独正确工作的代码单元在集成时可能有问题，所以集成代码的测试覆盖率也很重要。要创建集成测试，你首先需要 _tests_ 目录。

#### _tests_ 目录

我们在项目目录的顶层创建 _tests_ 目录，在 _src_ 旁边。Cargo 知道在此目录中查找集成测试文件。然后，我们可以创建任意数量的测试文件，Cargo 将每个文件编译为单独的 crate。

让我们创建一个集成测试。在代码清单 11-12 中的代码仍在 _src/lib.rs_ 文件中的情况下，创建一个 _tests_ 目录，并创建一个名为 _tests/integration_test.rs_ 的新文件。你的目录结构应该看起来像这样：

```text
adder
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs
```

将代码清单 11-13 中的代码输入到 _tests/integration_test.rs_ 文件中。

<Listing number="11-13" file-name="tests/integration_test.rs" caption="`adder` crate 中函数的集成测试">

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-13/tests/integration_test.rs}}
```

</Listing>

_tests_ 目录中的每个文件都是单独的 crate，所以我们需要将库引入每个测试 crate 的作用域。因此，我们在代码顶部添加 `use adder::add_two;`，这在单元测试中不需要。

我们不需要用 `#[cfg(test)]` 注释 _tests/integration_test.rs_ 中的任何代码。Cargo 特殊处理 _tests_ 目录，并且仅在我们运行 `cargo test` 时编译此目录中的文件。现在运行 `cargo test`：

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-13/output.txt}}
```

输出的三个部分包括单元测试、集成测试和文档测试。注意，如果某个部分中的任何测试失败，以下部分将不会运行。例如，如果单元测试失败，集成测试和文档测试将没有任何输出，因为只有在所有单元测试通过时才会运行这些测试。

单元测试的第一部分与我们一直看到的一样：每个单元测试一行（一个名为 `internal` 的测试，我们在代码清单 11-12 中添加了它），然后是单元测试的摘要行。

集成测试部分以行 `Running tests/integration_test.rs` 开始。接下来，该集成测试中的每个测试函数有一行，在 `Doc-tests adder` 部分开始之前，集成测试结果的摘要行。

每个集成测试文件都有自己的部分，所以如果我们在 _tests_ 目录中添加更多文件，将会有更多集成测试部分。

我们仍然可以通过将测试函数的名称指定为 `cargo test` 的参数来运行特定的集成测试函数。要运行特定集成测试文件中的所有测试，使用 `cargo test` 的 `--test` 参数后跟文件名：

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-05-single-integration/output.txt}}
```

此命令仅运行 _tests/integration_test.rs_ 文件中的测试。

#### 集成测试中的子模块

当你添加更多集成测试时，你可能想在 _tests_ 目录中创建更多文件以帮助组织它们；例如，你可以按它们正在测试的功能对测试函数进行分组。如前所述，_tests_ 目录中的每个文件都编译为自己的单独 crate，这对于创建单独的作用域以更紧密地模仿最终用户将使用你的 crate 的方式很有用。但是，这意味着 _tests_ 目录中的文件不会与 _src_ 中的文件共享相同的行为，正如你在第 7 章中了解的关于如何将代码分离到模块和文件中。

_tests_ 目录文件的不同行为在你有一组要在多个集成测试文件中使用的辅助函数，并且你尝试遵循第 7 章的["将模块分离到不同文件"][separating-modules-into-files]<!-- ignore -->部分中的步骤将它们提取到公共模块时最为明显。例如，如果我们创建 _tests/common.rs_ 并在其中放置一个名为 `setup` 的函数，我们可以向 `setup` 添加一些代码，我们希望从多个测试文件中的多个测试函数调用这些代码：

<span class="filename">Filename: tests/common.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/tests/common.rs}}
```

当我们再次运行测试时，我们将在测试输出中看到 _common.rs_ 文件的新部分，即使此文件不包含任何测试函数，我们也没有从任何地方调用 `setup` 函数：

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/output.txt}}
```

让 `common` 出现在测试结果中，并为其显示 `running 0 tests`，这不是我们想要的。我们只是想与其他集成测试文件共享一些代码。为了避免 `common` 出现在测试输出中，而不是创建 _tests/common.rs_，我们将创建 _tests/common/mod.rs_。项目目录现在看起来像这样：

```text
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    ├── common
    │   └── mod.rs
    └── integration_test.rs
```

这是 Rust 也理解的较旧命名约定，我们在第 7 章的["备用文件路径"][alt-paths]<!-- ignore -->中提到了这一点。以这种方式命名文件告诉 Rust 不要将 `common` 模块视为集成测试文件。当我们将 `setup` 函数代码移动到 _tests/common/mod.rs_ 并删除 _tests/common.rs_ 文件时，测试输出中的部分将不再出现。_tests_ 目录的子目录中的文件不会编译为单独的 crate 或在测试输出中有部分。

在我们创建 _tests/common/mod.rs_ 之后，我们可以从任何集成测试文件中将其作为模块使用。以下是从 _tests/integration_test.rs_ 中的 `it_adds_two` 测试调用 `setup` 函数的示例：

<span class="filename">Filename: tests/integration_test.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-13-fix-shared-test-code-problem/tests/integration_test.rs}}
```

注意，`mod common;` 声明与我们演示的代码清单 7-21 中的模块声明相同。然后，在测试函数中，我们可以调用 `common::setup()` 函数。

#### 二进制 Crate 的集成测试

如果我们的项目是一个只包含 _src/main.rs_ 文件而没有 _src/lib.rs_ 文件的二进制 crate，我们无法在 _tests_ 目录中创建集成测试，并使用 `use` 语句将 _src/main.rs_ 文件中定义的函数引入作用域。只有库 crate 暴露其他 crate 可以使用的函数；二进制 crate 旨在独立运行。

这就是 Rust 项目提供二进制文件的原因之一，它们有一个简单的 _src/main.rs_ 文件，该文件调用存在于 _src/lib.rs_ 文件中的逻辑。使用该结构，集成测试 _可以_ 使用 `use` 测试库 crate，使重要功能可用。如果重要功能工作正常，_src/main.rs_ 文件中的少量代码也会工作，并且不需要测试那少量代码。

## 总结

Rust 的测试功能提供了一种指定代码应该如何工作的方法，以确保即使在你进行更改时它也能继续按预期工作。单元测试分别测试库的不同部分，可以测试私有实现细节。集成测试检查库的许多部分是否正确地协同工作，它们使用库的公共 API 来测试代码，就像外部代码会使用它一样。尽管 Rust 的类型系统和所有权规则有助于防止某些类型的 bug，但测试对于减少与代码预期行为相关的逻辑 bug 仍然很重要。

让我们结合你在本章和前面章节中学到的知识来做一个项目！

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
[separating-modules-into-files]: ch07-05-separating-modules-into-different-files.html
[alt-paths]: ch07-05-separating-modules-into-different-files.html#alternate-file-paths
