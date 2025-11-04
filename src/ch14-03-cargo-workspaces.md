## Cargo 工作空间

在第 12 章中，我们构建了一个包含二进制 crate 和库 crate 的包。随着项目的发展，你可能会发现库 crate 继续变大，并且你想进一步将包拆分为多个库 crate。Cargo 提供了一个称为 _工作空间_ 的功能，可以帮助管理多个相关包，这些包是同时开发的。

### 创建工作空间

_工作空间_ 是一组共享相同 _Cargo.lock_ 和输出目录的包。让我们使用工作空间创建一个项目——我们将使用简单的代码，以便我们可以专注于工作空间的结构。有多种方法来构建工作空间，所以我们只展示一种常见的方法。我们将有一个包含二进制文件和两个库的工作空间。二进制文件将提供主要功能，将依赖于两个库。一个库将提供 `add_one` 函数，另一个库将提供 `add_two` 函数。这三个 crate 将成为同一工作空间的一部分。我们将首先为工作空间创建一个新目录：

```console
$ mkdir add
$ cd add
```

接下来，在 _add_ 目录中，我们创建将配置整个工作空间的 _Cargo.toml_ 文件。此文件不会有 `[package]` 部分。相反，它将以 `[workspace]` 部分开始，这将允许我们将成员添加到工作空间。我们还通过将 `resolver` 值设置为 `"3"` 来确保在工作空间中使用 Cargo 解析器算法的最新和最佳版本：

<span class="filename">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-01-workspace/add/Cargo.toml}}
```

接下来，我们通过在 _add_ 目录内运行 `cargo new` 来创建 `adder` 二进制 crate：

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-01-adder-crate/add
remove `members = ["adder"]` from Cargo.toml
rm -rf adder
cargo new adder
copy output below
-->

```console
$ cargo new adder
     Created binary (application) `adder` package
      Adding `adder` as member of workspace at `file:///projects/add`
```

在工作空间内运行 `cargo new` 也会自动将新创建的包添加到工作空间 _Cargo.toml_ 中 `[workspace]` 定义的 `members` 键中，如下所示：

```toml
{{#include ../listings/ch14-more-about-cargo/output-only-01-adder-crate/add/Cargo.toml}}
```

此时，我们可以通过运行 `cargo build` 来构建工作空间。你的 _add_ 目录中的文件应该看起来像这样：

```text
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

工作空间在顶层有一个 _target_ 目录，编译工件将放置在其中；`adder` 包没有自己的 _target_ 目录。即使我们从 _adder_ 目录内运行 `cargo build`，编译工件仍将最终出现在 _add/target_ 中，而不是 _add/adder/target_。Cargo 以这种方式构建工作空间中的 _target_ 目录，因为工作空间中的 crate 旨在相互依赖。如果每个 crate 都有自己的 _target_ 目录，每个 crate 都必须重新编译工作空间中的其他 crate，以便将工件放在自己的 _target_ 目录中。通过共享一个 _target_ 目录，crate 可以避免不必要的重建。

### 在工作空间中创建第二个包

接下来，让我们在工作空间中创建另一个成员包，并将其称为 `add_one`。生成一个名为 `add_one` 的新库 crate：

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-02-add-one/add
remove `"add_one"` from `members` list in Cargo.toml
rm -rf add_one
cargo new add_one --lib
copy output below
-->

```console
$ cargo new add_one --lib
     Created library `add_one` package
      Adding `add_one` as member of workspace at `file:///projects/add`
```

顶层 _Cargo.toml_ 现在将在 `members` 列表中包含 _add_one_ 路径：

<span class="filename">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/Cargo.toml}}
```

你的 _add_ 目录现在应该有这些目录和文件：

```text
├── Cargo.lock
├── Cargo.toml
├── add_one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

在 _add_one/src/lib.rs_ 文件中，让我们添加一个 `add_one` 函数：

<span class="filename">Filename: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/add_one/src/lib.rs}}
```

现在我们可以让带有二进制文件的 `adder` 包依赖于具有库的 `add_one` 包。首先，我们需要在 _adder/Cargo.toml_ 中添加对 `add_one` 的路径依赖。

<span class="filename">Filename: adder/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/adder/Cargo.toml:6:7}}
```

Cargo 不假设工作空间中的 crate 会相互依赖，所以我们需要明确指定依赖关系。

接下来，让我们在 `adder` crate 中使用 `add_one` 函数（来自 `add_one` crate）。打开 _adder/src/main.rs_ 文件，并将 `main` 函数更改为调用 `add_one` 函数，如代码清单 14-7 所示。

<Listing number="14-7" file-name="adder/src/main.rs" caption="从 `adder` crate 使用 `add_one` 库 crate">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-07/add/adder/src/main.rs}}
```

</Listing>

让我们通过在顶层 _add_ 目录中运行 `cargo build` 来构建工作空间！

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.22s
```

要从 _add_ 目录运行二进制 crate，我们可以通过使用 `-p` 参数和包名称与 `cargo run` 来指定我们想要运行的工作空间中的哪个包：

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo run -p adder
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo run -p adder
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/adder`
Hello, world! 10 plus one is 11!
```

这运行 _adder/src/main.rs_ 中的代码，它依赖于 `add_one` crate。

<!-- Old headings. Do not remove or links may break. -->

<a id="depending-on-an-external-package-in-a-workspace"></a>

### 依赖外部包

注意，工作空间在顶层只有一个 _Cargo.lock_ 文件，而不是在每个 crate 的目录中都有一个 _Cargo.lock_。这确保所有 crate 使用所有依赖项的相同版本。如果我们将 `rand` 包添加到 _adder/Cargo.toml_ 和 _add_one/Cargo.toml_ 文件中，Cargo 将把这两个解析为 `rand` 的一个版本，并在一个 _Cargo.lock_ 中记录它。使工作空间中的所有 crate 使用相同的依赖项意味着 crate 将始终彼此兼容。让我们将 `rand` crate 添加到 _add_one/Cargo.toml_ 文件中的 `[dependencies]` 部分，以便我们可以在 `add_one` crate 中使用 `rand` crate：

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch02-00-guessing-game-tutorial.md
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
-->

<span class="filename">Filename: add_one/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add/add_one/Cargo.toml:6:7}}
```

我们现在可以将 `use rand;` 添加到 _add_one/src/lib.rs_ 文件中，通过在 _add_ 目录中运行 `cargo build` 来构建整个工作空间将引入并编译 `rand` crate。我们会收到一个警告，因为我们没有引用我们引入作用域的 `rand`：

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
    Updating crates.io index
  Downloaded rand v0.8.5
   --snip--
   Compiling rand v0.8.5
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
warning: unused import: `rand`
 --> add_one/src/lib.rs:1:5
  |
1 | use rand;
  |     ^^^^
  |
  = note: `#[warn(unused_imports)]` on by default

warning: `add_one` (lib) generated 1 warning (run `cargo fix --lib -p add_one` to apply 1 suggestion)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.95s
```

顶层 _Cargo.lock_ 现在包含 `add_one` 对 `rand` 的依赖信息。但是，即使 `rand` 在工作空间的某个地方使用，我们也无法在工作空间中的其他 crate 中使用它，除非我们也将其添加到它们的 _Cargo.toml_ 文件中。例如，如果我们将 `use rand;` 添加到 `adder` 包的 _adder/src/main.rs_ 文件中，我们将收到错误：

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-03-use-rand/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
  --snip--
   Compiling adder v0.1.0 (file:///projects/add/adder)
error[E0432]: unresolved import `rand`
 --> adder/src/main.rs:2:5
  |
2 | use rand;
  |     ^^^^ no external crate `rand`
```

要修复此问题，请编辑 `adder` 包的 _Cargo.toml_ 文件，并指示 `rand` 也是它的依赖项。构建 `adder` 包将把 `rand` 添加到 _Cargo.lock_ 中 `adder` 的依赖项列表中，但不会下载 `rand` 的额外副本。只要它们指定 `rand` 的兼容版本，Cargo 将确保工作空间中每个包中使用 `rand` 包的每个 crate 都使用相同版本，为我们节省空间并确保工作空间中的 crate 彼此兼容。

如果工作空间中的 crate 指定同一依赖项的不兼容版本，Cargo 将解析它们中的每一个，但仍将尝试解析尽可能少的版本。

### 向工作空间添加测试

为了另一个增强，让我们在 `add_one` crate 中添加 `add_one::add_one` 函数的测试：

<span class="filename">Filename: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add/add_one/src/lib.rs}}
```

现在在顶层 _add_ 目录中运行 `cargo test`。在像这样的工作空间中运行 `cargo test` 将为工作空间中的所有 crate 运行测试：

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test
copy output below; the output updating script doesn't handle subdirectories in
paths properly
-->

```console
$ cargo test
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.20s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/adder-3a47283c568d2b6a)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

输出的第一部分显示 `add_one` crate 中的 `it_works` 测试通过了。下一部分显示在 `adder` crate 中找到了零个测试，然后最后一部分显示在 `add_one` crate 中找到了零个文档测试。

我们还可以通过使用 `-p` 标志并从顶层目录指定我们想要测试的 crate 名称来运行工作空间中特定 crate 的测试：

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test -p add_one
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo test -p add_one
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

此输出显示 `cargo test` 只运行了 `add_one` crate 的测试，没有运行 `adder` crate 测试。

如果你将工作空间中的 crate 发布到 [crates.io](https://crates.io/)<!-- ignore -->，工作空间中的每个 crate 都需要单独发布。就像 `cargo test` 一样，我们可以通过使用 `-p` 标志并指定我们想要发布的 crate 名称来发布工作空间中的特定 crate。

为了额外的练习，以与 `add_one` crate 类似的方式向此工作空间添加 `add_two` crate！

随着项目增长，考虑使用工作空间：它使你能够使用比一大块代码更小、更容易理解的组件。此外，如果 crate 经常同时更改，将 crate 保留在工作空间中可以使 crate 之间的协调更容易。
