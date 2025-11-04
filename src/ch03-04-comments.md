## 注释

所有程序员都努力使他们的代码易于理解，但有时需要额外的解释。在这些情况下，程序员在其源代码中留下_注释_，编译器将忽略这些注释，但阅读源代码的人可能会发现它们有用。

这是一个简单的注释：

```rust
// hello, world
```

在 Rust 中，惯用的注释样式以两个斜杠开始注释，注释一直持续到行尾。对于扩展到单行之外的注释，您需要在每一行包含 `//`，如下所示：

```rust
// 所以我们在这里做一些复杂的事情，足够长以至于我们需要
// 多行注释来完成它！哇！希望这个注释会
// 解释发生了什么。
```

注释也可以放在包含代码的行末尾：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-24-comments-end-of-line/src/main.rs}}
```

但是您会更经常看到它们以这种格式使用，注释在它注释的代码上方的单独行上：

<span class="filename">文件名：src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-25-comments-above-line/src/main.rs}}
```

Rust 还有另一种注释，文档注释，我们将在第 14 章的["发布 Crate 到 Crates.io"][publishing]<!-- ignore -->部分中讨论。

[publishing]: ch14-02-publishing-to-crates-io.html
