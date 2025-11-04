## 发布 Crate 到 Crates.io

我们已经使用 [crates.io](https://crates.io/)<!-- ignore --> 上的包作为我们项目的依赖项，但你也可以通过发布自己的包来与其他人员共享代码。在 [crates.io](https://crates.io/)<!-- ignore --> 的 crate 注册表分发包的源代码，所以它主要托管开源代码。

Rust 和 Cargo 有一些功能，使人们更容易找到和使用你发布的包。接下来我们将讨论其中一些功能，然后解释如何发布包。

### 编写有用的文档注释

准确记录你的包将帮助其他用户了解如何以及何时使用它们，所以值得投入时间编写文档。在第 3 章中，我们讨论了如何使用两个斜杠 `//` 注释 Rust 代码。Rust 还有一种特殊的文档注释，方便地称为 _文档注释_，它将生成 HTML 文档。HTML 显示公共 API 项的文档注释内容，这些内容面向有兴趣了解如何 _使用_ 你的 crate 而不是你的 crate 如何 _实现_ 的程序员。

文档注释使用三个斜杠 `///`，而不是两个，并支持 Markdown 符号来格式化文本。将文档注释放在它们正在记录的项之前。代码清单 14-1 显示了名为 `my_crate` 的 crate 中 `add_one` 函数的文档注释。

<Listing number="14-1" file-name="src/lib.rs" caption="函数的文档注释">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-01/src/lib.rs}}
```

</Listing>

这里，我们给出 `add_one` 函数功能的描述，开始一个带有标题 `Examples` 的部分，然后提供演示如何使用 `add_one` 函数的代码。我们可以通过运行 `cargo doc` 从该文档注释生成 HTML 文档。此命令运行随 Rust 一起分发的 `rustdoc` 工具，并将生成的 HTML 文档放在 _target/doc_ 目录中。

为了方便，运行 `cargo doc --open` 将为你当前 crate 的文档（以及所有 crate 依赖项的文档）构建 HTML，并在 Web 浏览器中打开结果。导航到 `add_one` 函数，你将看到文档注释中的文本如何呈现，如 Figure 14-1 所示。

<img alt="Rendered HTML documentation for the `add_one` function of `my_crate`" src="img/trpl14-01.png" class="center" />

<span class="caption">Figure 14-1: `add_one` 函数的 HTML 文档</span>

#### 常用部分

我们在代码清单 14-1 中使用了 `# Examples` Markdown 标题来在 HTML 中创建一个标题为"Examples"的部分。以下是一些 crate 作者在其文档中常用的其他部分：

- **Panics**：这些是被记录的函数可能 panic 的场景。不希望程序 panic 的函数调用者应确保他们不会在这些情况下调用函数。
- **Errors**：如果函数返回 `Result`，描述可能发生的错误类型以及可能导致这些错误返回的条件可能对调用者有帮助，以便他们可以编写代码以不同方式处理不同类型的错误。
- **Safety**：如果函数调用是 `unsafe`（我们在第 20 章讨论不安全），应该有一个部分解释为什么函数不安全，并涵盖函数期望调用者维护的不变量。

大多数文档注释不需要所有这些部分，但这是一个很好的清单，提醒你代码的用户会有兴趣了解的方面。

#### 文档注释作为测试

在文档注释中添加示例代码块可以帮助演示如何使用库，并有一个额外的好处：运行 `cargo test` 将在你的文档中运行代码示例作为测试！没有什么比带有示例的文档更好的了。但没有什么比示例不起作用更糟糕的了，因为代码自文档编写以来已经更改。如果我们使用代码清单 14-1 中 `add_one` 函数的文档运行 `cargo test`，我们将在测试结果中看到一个看起来像这样的部分：

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-01/
cargo test
copy just the doc-tests section below
-->

```text
   Doc-tests my_crate

running 1 test
test src/lib.rs - add_one (line 5) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.27s
```

现在，如果我们更改函数或示例，使示例中的 `assert_eq!` panic，并再次运行 `cargo test`，我们将看到文档测试捕获示例和代码彼此不同步！

<!-- Old headings. Do not remove or links may break. -->

<a id="commenting-contained-items"></a>

#### 包含项的注释

文档注释样式 `//!` 将文档添加到 _包含_ 注释的项，而不是 _跟随_ 注释的项。我们通常在 crate 根文件（按约定为 _src/lib.rs_）内或模块内使用这些文档注释，以将 crate 或模块作为一个整体进行文档化。

例如，要添加描述包含 `add_one` 函数的 `my_crate` crate 的目的的文档，我们将以 `//!` 开头的文档注释添加到 _src/lib.rs_ 文件的开头，如代码清单 14-2 所示。

<Listing number="14-2" file-name="src/lib.rs" caption="整个 `my_crate` crate 的文档">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-02/src/lib.rs:here}}
```

</Listing>

注意，在以 `//!` 开头的最后一行之后没有任何代码。因为我们用 `//!` 而不是 `///` 开始注释，我们正在记录包含此注释的项，而不是跟随此注释的项。在这种情况下，该项是 _src/lib.rs_ 文件，它是 crate 根。这些注释描述整个 crate。

当我们运行 `cargo doc --open` 时，这些注释将显示在 `my_crate` 文档的首页上，在 crate 中公共项列表的上方，如 Figure 14-2 所示。

项内的文档注释对于描述 crate 和模块特别有用。使用它们来解释容器的整体目的，以帮助用户理解 crate 的组织。

<img alt="Rendered HTML documentation with a comment for the crate as a whole" src="img/trpl14-02.png" class="center" />

<span class="caption">Figure 14-2: `my_crate` 的渲染文档，包括描述整个 crate 的注释</span>

<!-- Old headings. Do not remove or links may break. -->

<a id="exporting-a-convenient-public-api-with-pub-use"></a>

### 导出便捷的公共 API

公共 API 的结构是发布 crate 时的主要考虑因素。使用你的 crate 的人对你的结构不如你熟悉，如果你的 crate 具有大型模块层次结构，他们可能难以找到他们想要使用的部分。

在第 7 章中，我们涵盖了如何使用 `pub` 关键字使项公开，以及如何使用 `use` 关键字将项引入作用域。但是，你在开发 crate 时对你有意义的结构可能对你的用户来说不太方便。你可能想要在包含多个级别的层次结构中组织你的结构体，但然后想要使用你在层次结构深处定义的类型的人可能难以发现该类型存在。他们也可能对必须输入 `use my_crate::some_module::another_module::UsefulType;` 而不是 `use my_crate::UsefulType;` 感到烦恼。

好消息是，如果结构 _不_ 方便其他人从另一个库中使用，你不必重新排列内部组织：相反，你可以使用 `pub use` 重新导出项，以创建与私有结构不同的公共结构。*重新导出* 在一个位置获取公共项，并在另一个位置使其公开，就好像它是在另一个位置定义的一样。

例如，假设我们制作了一个名为 `art` 的库，用于建模艺术概念。在这个库中有两个模块：一个 `kinds` 模块，包含两个名为 `PrimaryColor` 和 `SecondaryColor` 的枚举，以及一个 `utils` 模块，包含一个名为 `mix` 的函数，如代码清单 14-3 所示。

<Listing number="14-3" file-name="src/lib.rs" caption="一个 `art` 库，项被组织到 `kinds` 和 `utils` 模块中">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-03/src/lib.rs:here}}
```

</Listing>

Figure 14-3 显示了 `cargo doc` 为此 crate 生成的文档首页的外观。

<img alt="Rendered documentation for the `art` crate that lists the `kinds` and `utils` modules" src="img/trpl14-03.png" class="center" />

<span class="caption">Figure 14-3: `art` 文档的首页，列出了 `kinds` 和 `utils` 模块</span>

注意，`PrimaryColor` 和 `SecondaryColor` 类型未列在首页上，`mix` 函数也未列出。我们必须单击 `kinds` 和 `utils` 才能看到它们。

依赖此库的另一个 crate 需要 `use` 语句，将项从 `art` 引入作用域，指定当前定义的模块结构。代码清单 14-4 显示了一个使用 `art` crate 的 `PrimaryColor` 和 `mix` 项的 crate 示例。

<Listing number="14-4" file-name="src/main.rs" caption="一个使用 `art` crate 的项的 crate，其内部结构已导出">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-04/src/main.rs}}
```

</Listing>

代码清单 14-4 中使用 `art` crate 的代码的作者必须弄清楚 `PrimaryColor` 在 `kinds` 模块中，`mix` 在 `utils` 模块中。`art` crate 的模块结构对在 `art` crate 上工作的开发人员比对使用它的开发人员更相关。内部结构不包含任何对试图理解如何使用 `art` crate 的人有用的信息，而是导致混淆，因为使用它的开发人员必须弄清楚在哪里查找，并且必须在 `use` 语句中指定模块名称。

为了从公共 API 中移除内部组织，我们可以修改代码清单 14-3 中的 `art` crate 代码，添加 `pub use` 语句以在顶层重新导出项，如代码清单 14-5 所示。

<Listing number="14-5" file-name="src/lib.rs" caption="添加 `pub use` 语句以重新导出项">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-05/src/lib.rs:here}}
```

</Listing>

`cargo doc` 为此 crate 生成的 API 文档现在将在首页列出并链接重新导出，如 Figure 14-4 所示，使 `PrimaryColor` 和 `SecondaryColor` 类型以及 `mix` 函数更容易找到。

<img alt="Rendered documentation for the `art` crate with the re-exports on the front page" src="img/trpl14-04.png" class="center" />

<span class="caption">Figure 14-4: `art` 文档的首页，列出了重新导出</span>

`art` crate 用户仍然可以看到并使用代码清单 14-3 中的内部结构，如代码清单 14-4 中所示，或者他们可以使用代码清单 14-5 中更方便的结构，如代码清单 14-6 所示。

<Listing number="14-6" file-name="src/main.rs" caption="一个使用从 `art` crate 重新导出项的程序">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-06/src/main.rs:here}}
```

</Listing>

在有许多嵌套模块的情况下，使用 `pub use` 在顶层重新导出类型可以对使用 crate 的人的体验产生重大影响。`pub use` 的另一个常见用途是在当前 crate 中重新导出依赖项的定义，以将该 crate 的定义作为你的 crate 公共 API 的一部分。

创建有用的公共 API 结构更像是一门艺术而不是科学，你可以迭代以找到最适合用户的 API。选择 `pub use` 让你在如何内部构建 crate 方面具有灵活性，并将该内部结构与向用户呈现的内容分离。查看你已安装的一些 crate 的代码，看看它们的内部结构是否与它们的公共 API 不同。

### 设置 Crates.io 账户

在你可以发布任何 crate 之前，你需要在 [crates.io](https://crates.io/)<!-- ignore --> 上创建一个账户并获取 API 令牌。为此，访问 [crates.io](https://crates.io/)<!-- ignore --> 的主页，并通过 GitHub 账户登录。（GitHub 账户目前是必需的，但该网站将来可能支持其他创建账户的方式。）登录后，访问你的账户设置 [https://crates.io/me/](https://crates.io/me/)<!-- ignore --> 并检索你的 API 密钥。然后，运行 `cargo login` 命令并在提示时粘贴你的 API 密钥，如下所示：

```console
$ cargo login
abcdefghijklmnopqrstuvwxyz012345
```

此命令将通知 Cargo 你的 API 令牌并将其本地存储在 _~/.cargo/credentials.toml_ 中。注意，此令牌是秘密的：不要与任何人共享。如果你出于任何原因与任何人共享它，你应该撤销它并在 [crates.io](https://crates.io/)<!-- ignore --> 上生成新令牌。

### 向新 Crate 添加元数据

假设你有一个想要发布的 crate。在发布之前，你需要在 crate 的 _Cargo.toml_ 文件的 `[package]` 部分添加一些元数据。

你的 crate 需要一个唯一的名称。当你在本地处理 crate 时，你可以随意命名 crate。但是，[crates.io](https://crates.io/)<!-- ignore --> 上的 crate 名称是按先到先得的方式分配的。一旦 crate 名称被占用，其他人就不能发布具有该名称的 crate。在尝试发布 crate 之前，搜索你想要使用的名称。如果该名称已被使用，你将需要找到另一个名称，并在 `[package]` 部分下的 _Cargo.toml_ 文件中编辑 `name` 字段以使用新名称进行发布，如下所示：

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
```

即使你选择了唯一的名称，当你此时运行 `cargo publish` 以发布 crate 时，你将收到警告然后错误：

<!-- manual-regeneration
Create a new package with an unregistered name, making no further modifications
  to the generated package, so it is missing the description and license fields.
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
warning: manifest has no description, license, license-file, documentation, homepage or repository.
See https://doc.rust-lang.org/cargo/reference/manifest.html#package-metadata for more info.
--snip--
error: failed to publish to registry at https://crates.io

Caused by:
  the remote server responded with an error (status 400 Bad Request): missing or empty metadata fields: description, license. Please see https://doc.rust-lang.org/cargo/reference/manifest.html for more information on configuring these fields
```

这会导致错误，因为你缺少一些关键信息：需要描述和许可证，以便人们知道你的 crate 做什么以及他们可以在什么条款下使用它。在 _Cargo.toml_ 中，添加一个只是一两句话的描述，因为它将出现在搜索结果中的 crate 旁边。对于 `license` 字段，你需要给出 _许可证标识符值_。[Linux Foundation 的 Software Package Data Exchange (SPDX)][spdx] 列出了你可以用于此值的标识符。例如，要指定你已使用 MIT 许可证许可你的 crate，请添加 `MIT` 标识符：

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
license = "MIT"
```

如果你想使用 SPDX 中未出现的许可证，你需要将该许可证的文本放在文件中，将该文件包含在你的项目中，然后使用 `license-file` 指定该文件的名称，而不是使用 `license` 键。

哪种许可证适合你的项目的指导超出了本书的范围。Rust 社区中的许多人通过与 Rust 相同的方式许可他们的项目，通过使用 `MIT OR Apache-2.0` 的双重许可证。这种做法表明你也可以指定多个由 `OR` 分隔的许可证标识符，以便为项目提供多个许可证。

有了唯一名称、版本、你的描述和许可证，准备发布的项目的 _Cargo.toml_ 文件可能看起来像这样：

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2024"
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0"

[dependencies]
```

[Cargo 的文档](https://doc.rust-lang.org/cargo/)描述了你可以指定的其他元数据，以确保其他人能够更容易地发现和使用你的 crate。

### 发布到 Crates.io

既然你已经创建了账户，保存了 API 令牌，为 crate 选择了名称，并指定了必需的元数据，你已经准备好发布了！发布 crate 将特定版本上传到 [crates.io](https://crates.io/)<!-- ignore --> 供其他人使用。

要小心，因为发布是 _永久性的_。版本永远不能被覆盖，代码不能删除，除非在某些情况下。Crates.io 的一个主要目标是作为代码的永久存档，以便依赖 [crates.io](https://crates.io/)<!-- ignore --> 上的 crate 的所有项目的构建将继续工作。允许版本删除将使实现该目标变得不可能。但是，你可以发布的 crate 版本数量没有限制。

再次运行 `cargo publish` 命令。它现在应该成功：

<!-- manual-regeneration
go to some valid crate, publish a new version
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
   Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
    Packaged 6 files, 1.2KiB (895.0B compressed)
   Verifying guessing_game v0.1.0 (file:///projects/guessing_game)
   Compiling guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.19s
   Uploading guessing_game v0.1.0 (file:///projects/guessing_game)
    Uploaded guessing_game v0.1.0 to registry `crates-io`
note: waiting for `guessing_game v0.1.0` to be available at registry
`crates-io`.
You may press ctrl-c to skip waiting; the crate should be available shortly.
   Published guessing_game v0.1.0 at registry `crates-io`
```

恭喜！你现在已经与 Rust 社区分享了你的代码，任何人都可以轻松地将你的 crate 添加为他们项目的依赖项。

### 发布现有 Crate 的新版本

当你对 crate 进行了更改并准备发布新版本时，你更改 _Cargo.toml_ 文件中指定的 `version` 值并重新发布。使用[语义版本规则][semver]根据你进行的更改类型决定合适的下一个版本号。然后，运行 `cargo publish` 上传新版本。

<!-- Old headings. Do not remove or links may break. -->

<a id="removing-versions-from-cratesio-with-cargo-yank"></a>
<a id="deprecating-versions-from-cratesio-with-cargo-yank"></a>

### 从 Crates.io 弃用版本

尽管你不能删除 crate 的先前版本，但你可以防止任何未来项目将它们添加为新依赖项。当 crate 版本因某种原因损坏时，这很有用。在这种情况下，Cargo 支持 yank crate 版本。

_Yanking_ 版本会阻止新项目依赖该版本，同时允许所有依赖它的现有项目继续。本质上，yank 意味着所有具有 _Cargo.lock_ 的项目都不会损坏，并且任何未来生成的 _Cargo.lock_ 文件都不会使用 yanked 版本。

要 yank crate 的版本，在你之前发布的 crate 目录中，运行 `cargo yank` 并指定你想要 yank 的版本。例如，如果我们发布了名为 `guessing_game` 版本 1.0.1 的 crate，并且我们想要 yank 它，那么我们将在 `guessing_game` 的项目目录中运行以下命令：

<!-- manual-regeneration:
cargo yank carol-test --version 2.1.0
cargo yank carol-test --version 2.1.0 --undo
-->

```console
$ cargo yank --vers 1.0.1
    Updating crates.io index
        Yank guessing_game@1.0.1
```

通过在命令中添加 `--undo`，你也可以撤销 yank 并允许项目再次开始依赖版本：

```console
$ cargo yank --vers 1.0.1 --undo
    Updating crates.io index
      Unyank guessing_game@1.0.1
```

Yank _不会_ 删除任何代码。例如，它不能删除意外上传的秘密。如果发生这种情况，你必须立即重置这些秘密。

[spdx]: https://spdx.org/licenses/
[semver]: https://semver.org/
