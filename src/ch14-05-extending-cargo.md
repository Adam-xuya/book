## 使用自定义命令扩展 Cargo

Cargo 的设计允许你使用新的子命令扩展它，而无需修改它。如果你的 `$PATH` 中的二进制文件名为 `cargo-something`，你可以通过运行 `cargo something` 将其作为 Cargo 子命令运行。当你运行 `cargo --list` 时，也会列出这样的自定义命令。能够使用 `cargo install` 安装扩展然后像内置 Cargo 工具一样运行它们是 Cargo 设计的超级便利的好处！

## 总结

使用 Cargo 和 [crates.io](https://crates.io/)<!-- ignore --> 共享代码是使 Rust 生态系统对许多不同任务有用的部分原因。Rust 的标准库小而稳定，但 crate 很容易共享、使用和改进，时间线与语言不同。不要害羞在 [crates.io](https://crates.io/)<!-- ignore --> 上共享对你有用的代码；它可能对其他人也有用！
