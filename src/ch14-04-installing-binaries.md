<!-- Old headings. Do not remove or links may break. -->

<a id="installing-binaries-from-cratesio-with-cargo-install"></a>

## 使用 `cargo install` 安装二进制文件

`cargo install` 命令允许你本地安装和使用二进制 crate。这不是为了替代系统包；它是为了方便 Rust 开发人员安装其他人已在 [crates.io](https://crates.io/)<!-- ignore --> 上共享的工具。注意，你只能安装具有二进制目标的包。_二进制目标_ 是如果 crate 有 _src/main.rs_ 文件或指定为二进制的另一个文件而创建的可运行程序，与库目标相反，库目标本身不可运行但适合包含在其他程序中。通常，crate 在 README 文件中有关于 crate 是库、有二进制目标还是两者兼有的信息。

所有使用 `cargo install` 安装的二进制文件都存储在安装根的 _bin_ 文件夹中。如果你使用 _rustup.rs_ 安装 Rust 并且没有任何自定义配置，此目录将是 *$HOME/.cargo/bin*。确保此目录在你的 `$PATH` 中，以便能够运行你使用 `cargo install` 安装的程序。

例如，在第 12 章中，我们提到了 `grep` 工具的 Rust 实现，称为 `ripgrep`，用于搜索文件。要安装 `ripgrep`，我们可以运行以下命令：

<!-- manual-regeneration
cargo install something you don't have, copy relevant output below
-->

```console
$ cargo install ripgrep
    Updating crates.io index
  Downloaded ripgrep v14.1.1
  Downloaded 1 crate (213.6 KB) in 0.40s
  Installing ripgrep v14.1.1
--snip--
   Compiling grep v0.3.2
    Finished `release` profile [optimized + debuginfo] target(s) in 6.73s
  Installing ~/.cargo/bin/rg
   Installed package `ripgrep v14.1.1` (executable `rg`)
```

输出的倒数第二行显示已安装二进制文件的位置和名称，对于 `ripgrep`，它是 `rg`。只要安装目录在你的 `$PATH` 中，如前所述，然后你可以运行 `rg --help` 并开始使用更快、更 Rust 的工具来搜索文件！
