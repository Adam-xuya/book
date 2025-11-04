## 使用发布配置文件自定义构建

在 Rust 中，_发布配置文件_ 是具有不同配置的预定义、可自定义配置文件，允许程序员更好地控制编译代码的各种选项。每个配置文件都是独立配置的。

Cargo 有两个主要配置文件：`dev` 配置文件，Cargo 在你运行 `cargo build` 时使用，以及 `release` 配置文件，Cargo 在你运行 `cargo build --release` 时使用。`dev` 配置文件使用适合开发的良好默认值定义，`release` 配置文件具有适合发布构建的良好默认值。

这些配置文件名称可能从你的构建输出中熟悉：

<!-- manual-regeneration
anywhere, run:
cargo build
cargo build --release
and ensure output below is accurate
-->

```console
$ cargo build
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
$ cargo build --release
    Finished `release` profile [optimized] target(s) in 0.32s
```

`dev` 和 `release` 是编译器使用的这些不同配置文件。

Cargo 为每个配置文件设置了默认设置，当你在项目的 _Cargo.toml_ 文件中没有显式添加任何 `[profile.*]` 部分时应用这些设置。通过为你想要自定义的任何配置文件添加 `[profile.*]` 部分，你可以覆盖默认设置的任何子集。例如，以下是 `dev` 和 `release` 配置文件的 `opt-level` 设置的默认值：

<span class="filename">Filename: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

`opt-level` 设置控制 Rust 将应用于代码的优化数量，范围为 0 到 3。应用更多优化会延长编译时间，所以如果你在开发中并且经常编译代码，你会希望更少的优化以便更快地编译，即使生成的代码运行更慢。因此，`dev` 的默认 `opt-level` 是 `0`。当你准备发布代码时，最好花费更多时间编译。你只会以发布模式编译一次，但会多次运行编译的程序，所以发布模式以更长的编译时间换取运行更快的代码。这就是为什么 `release` 配置文件的默认 `opt-level` 是 `3`。

你可以通过在 _Cargo.toml_ 中为其添加不同的值来覆盖默认设置。例如，如果我们想在开发配置文件中使用优化级别 1，我们可以将这两行添加到项目的 _Cargo.toml_ 文件中：

<span class="filename">Filename: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 1
```

此代码覆盖了 `0` 的默认设置。现在当我们运行 `cargo build` 时，Cargo 将使用 `dev` 配置文件的默认值加上我们对 `opt-level` 的自定义。因为我们设置 `opt-level` 为 `1`，Cargo 将应用比默认值更多的优化，但不如发布构建中的那么多。

有关每个配置文件的配置选项和默认值的完整列表，请参阅 [Cargo 的文档](https://doc.rust-lang.org/cargo/reference/profiles.html)。
