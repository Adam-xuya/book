## 实现面向对象设计模式

_状态模式_是一种面向对象的设计模式。该模式的核心是我们定义一组值可以在内部具有的状态。状态由一组_状态对象_表示，值的行为根据其状态而改变。我们将通过一个博客文章结构体的示例进行工作，该结构体有一个字段来保存其状态，该状态将是来自集合“草稿”、“审查”或“已发布”的状态对象。

状态对象共享功能：在 Rust 中，当然，我们使用结构体和 traits 而不是对象和继承。每个状态对象负责自己的行为并管理何时应该更改为另一个状态。持有状态对象的值对不同状态的行为或何时在状态之间转换一无所知。

使用状态模式的优点是，当程序的业务需求发生变化时，我们不需要更改持有状态的值的代码或使用该值的代码。我们只需要更新其中一个状态对象内部的代码以更改其规则，或者可能添加更多状态对象。

首先，我们将以更传统的面向对象方式实现状态模式。然后，我们将使用在 Rust 中更自然的方法。让我们深入研究逐步使用状态模式实现博客文章工作流。

最终功能将如下所示：

1. 博客文章以空草稿开始。
1. 当草稿完成时，请求对文章的审查。
1. 当文章被批准时，它会被发布。
1. 只有已发布的博客文章返回内容以打印，以便未批准的文章不能意外发布。

在文章上尝试的任何其他更改应该没有效果。例如，如果我们在请求审查之前尝试批准草稿博客文章，文章应该保持未发布的草稿状态。

<!-- Old headings. Do not remove or links may break. -->

<a id="a-traditional-object-oriented-attempt"></a>

### 尝试传统面向对象风格

构建代码以解决相同问题的方法有无限种，每种都有不同的权衡。本节的实现更像是传统的面向对象风格，这在 Rust 中是可能的，但没有利用 Rust 的一些优势。稍后，我们将演示一个不同的解决方案，它仍然使用面向对象的设计模式，但结构化的方式可能对具有面向对象经验的程序员来说不太熟悉。我们将比较这两种解决方案，以体验设计 Rust 代码与设计其他语言代码不同的权衡。

代码清单18-11以代码形式显示了此工作流：这是我们将在一个名为 `blog` 的库 crate 中实现的 API 的使用示例。这还不能编译，因为我们还没有实现 `blog` crate。

<Listing number="18-11" file-name="src/main.rs" caption="演示我们希望 `blog` crate 具有的所需行为的代码">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:all}}
```

</Listing>

我们希望允许用户使用 `Post::new` 创建新的草稿博客文章。我们希望允许向博客文章添加文本。如果我们立即尝试获取文章的内容，在批准之前，我们不应该获得任何文本，因为文章仍然是草稿。我们在代码中添加了 `assert_eq!` 用于演示目的。对此的优秀单元测试将是断言草稿博客文章从 `content` 方法返回空字符串，但我们不会为此示例编写测试。

接下来，我们希望启用对文章的审查请求，并且我们希望 `content` 在等待审查时返回空字符串。当文章获得批准时，它应该被发布，这意味着当调用 `content` 时将返回文章的文本。

请注意，我们从 crate 交互的唯一类型是 `Post` 类型。该类型将使用状态模式，并将持有一个值，该值将是三个状态对象之一，表示文章可以处于的各种状态——草稿、审查或已发布。从一个状态到另一个状态的更改将在 `Post` 类型内部管理。状态响应我们库用户在 `Post` 实例上调用的方法而改变，但它们不必直接管理状态更改。此外，用户不能在状态上犯错误，例如在审查之前发布文章。

<!-- Old headings. Do not remove or links may break. -->

<a id="defining-post-and-creating-a-new-instance-in-the-draft-state"></a>

#### 定义 `Post` 并创建新实例

让我们开始实现库！我们知道我们需要一个公共 `Post` 结构体，它保存一些内容，所以我们将从结构体的定义和关联的公共 `new` 函数开始，以创建 `Post` 的实例，如代码清单18-12所示。我们还将创建一个私有 `State` trait，它将定义 `Post` 的所有状态对象必须具有的行为。

然后，`Post` 将在 `Option<T>` 内的私有字段 `state` 中保存 `Box<dyn State>` 的 trait 对象以保存状态对象。你稍后会看到为什么 `Option<T>` 是必要的。

<Listing number="18-12" file-name="src/lib.rs" caption="`Post` 结构体的定义和创建新 `Post` 实例的 `new` 函数、`State` trait 和 `Draft` 结构体">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-12/src/lib.rs}}
```

</Listing>

`State` trait 定义不同文章状态共享的行为。状态对象是 `Draft`、`PendingReview` 和 `Published`，它们都将实现 `State` trait。目前，trait 没有任何方法，我们将从定义 `Draft` 状态开始，因为这是我们希望文章开始的状态。

当我们创建一个新的 `Post` 时，我们将其 `state` 字段设置为一个 `Some` 值，该值保存一个 `Box`。这个 `Box` 指向 `Draft` 结构体的新实例。这确保每当我们创建 `Post` 的新实例时，它将作为草稿开始。因为 `Post` 的 `state` 字段是私有的，无法以任何其他状态创建 `Post`！在 `Post::new` 函数中，我们将 `content` 字段设置为新的空 `String`。

#### 存储文章内容的文本

我们在代码清单18-11中看到，我们希望能够调用名为 `add_text` 的方法并向其传递 `&str`，然后将其添加为博客文章的文本内容。我们将其实现为方法，而不是将 `content` 字段公开为 `pub`，以便稍后我们可以实现一个方法来控制如何读取 `content` 字段的数据。`add_text` 方法非常简单，所以让我们将代码清单18-13中的实现添加到 `impl Post` 块中。

<Listing number="18-13" file-name="src/lib.rs" caption="实现 `add_text` 方法以向文章的 `content` 添加文本">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-13/src/lib.rs:here}}
```

</Listing>

`add_text` 方法接受对 `self` 的可变引用，因为我们正在更改调用 `add_text` 的 `Post` 实例。然后我们在 `content` 中的 `String` 上调用 `push_str`，并传递 `text` 参数以添加到保存的 `content`。此行为不依赖于文章所处的状态，因此它不是状态模式的一部分。`add_text` 方法根本不与 `state` 字段交互，但它是我们希望支持的行为的一部分。

<!-- Old headings. Do not remove or links may break. -->

<a id="ensuring-the-content-of-a-draft-post-is-empty"></a>

#### 确保草稿文章的内容为空

即使在我们调用 `add_text` 并向我们的文章添加一些内容之后，我们仍然希望 `content` 方法返回空字符串切片，因为文章仍在草稿状态，如代码清单18-11中的第一个 `assert_eq!` 所示。现在，让我们用将满足此要求的最简单的事情来实现 `content` 方法：始终返回空字符串切片。一旦我们实现了更改文章状态的能力，以便它可以被发布，我们将更改此方法。到目前为止，文章只能处于草稿状态，所以文章内容应该始终为空。代码清单18-14显示了此占位符实现。

<Listing number="18-14" file-name="src/lib.rs" caption="为 `Post` 上的 `content` 方法添加占位符实现，始终返回空字符串切片">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-14/src/lib.rs:here}}
```

</Listing>

通过添加这个 `content` 方法，代码清单18-11中到第一个 `assert_eq!` 的所有内容都按预期工作。

<!-- Old headings. Do not remove or links may break. -->

<a id="requesting-a-review-of-the-post-changes-its-state"></a>
<a id="requesting-a-review-changes-the-posts-state"></a>

#### 请求审查，这会更改文章的状态

接下来，我们需要添加功能来请求对文章的审查，这应该将其状态从 `Draft` 更改为 `PendingReview`。代码清单18-15显示了此代码。

<Listing number="18-15" file-name="src/lib.rs" caption="在 `Post` 和 `State` trait 上实现 `request_review` 方法">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-15/src/lib.rs:here}}
```

</Listing>

我们给 `Post` 一个名为 `request_review` 的公共方法，它将接受对 `self` 的可变引用。然后，我们在 `Post` 的当前状态上调用内部 `request_review` 方法，这第二个 `request_review` 方法消费当前状态并返回新状态。

我们将 `request_review` 方法添加到 `State` trait；所有实现该 trait 的类型现在都需要实现 `request_review` 方法。请注意，方法的第一个参数不是 `self`、`&self` 或 `&mut self`，而是 `self: Box<Self>`。这个语法意味着该方法仅在持有类型的 `Box` 上调用时有效。此语法取得 `Box<Self>` 的所有权，使旧状态无效，以便 `Post` 的状态值可以转换为新状态。

为了消费旧状态，`request_review` 方法需要取得状态值的所有权。这就是 `Post` 的 `state` 字段中的 `Option` 的来源：我们调用 `take` 方法从 `state` 字段中取出 `Some` 值，并在其位置留下 `None`，因为 Rust 不允许我们在结构体中有未填充的字段。这让我们可以将 `state` 值移出 `Post` 而不是借用它。然后，我们将把文章的 `state` 值设置为此操作的结果。

我们需要将 `state` 临时设置为 `None`，而不是使用像 `self.state = self.state.request_review();` 这样的代码直接设置它，以取得 `state` 值的所有权。这确保 `Post` 在我们将其转换为新状态后不能使用旧的 `state` 值。

`Draft` 上的 `request_review` 方法返回新的 `PendingReview` 结构体的新装箱实例，它表示文章等待审查时的状态。`PendingReview` 结构体也实现 `request_review` 方法但不进行任何转换。相反，它返回自身，因为当我们在已经处于 `PendingReview` 状态的文章上请求审查时，它应该保持在 `PendingReview` 状态。

现在我们可以开始看到状态模式的优点：`Post` 上的 `request_review` 方法无论其 `state` 值是什么都是相同的。每个状态负责自己的规则。

我们将把 `Post` 上的 `content` 方法保持原样，返回空字符串切片。我们现在可以在 `PendingReview` 状态以及 `Draft` 状态下有一个 `Post`，但我们希望在 `PendingReview` 状态下有相同的行为。代码清单18-11现在可以工作到第二个 `assert_eq!` 调用！

<!-- Old headings. Do not remove or links may break. -->

<a id="adding-the-approve-method-that-changes-the-behavior-of-content"></a>
<a id="adding-approve-to-change-the-behavior-of-content"></a>

#### 添加 `approve` 以更改 `content` 的行为

`approve` 方法将类似于 `request_review` 方法：它将把 `state` 设置为当前状态说当该状态被批准时应该具有的值，如代码清单18-16所示。

<Listing number="18-16" file-name="src/lib.rs" caption="在 `Post` 和 `State` trait 上实现 `approve` 方法">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-16/src/lib.rs:here}}
```

</Listing>

我们将 `approve` 方法添加到 `State` trait，并添加一个实现 `State` 的新结构体，`Published` 状态。

与 `PendingReview` 上的 `request_review` 类似的方式，如果我们在 `Draft` 上调用 `approve` 方法，它将没有效果，因为 `approve` 将返回 `self`。当我们在 `PendingReview` 上调用 `approve` 时，它返回 `Published` 结构体的新装箱实例。`Published` 结构体实现 `State` trait，对于 `request_review` 方法和 `approve` 方法，它返回自身，因为文章在这些情况下应该保持在 `Published` 状态。

现在我们需要更新 `Post` 上的 `content` 方法。我们希望从 `content` 返回的值取决于 `Post` 的当前状态，所以我们将让 `Post` 委托给在其 `state` 上定义的 `content` 方法，如代码清单18-17所示。

<Listing number="18-17" file-name="src/lib.rs" caption="更新 `Post` 上的 `content` 方法以委托给 `State` 上的 `content` 方法">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-17/src/lib.rs:here}}
```

</Listing>

因为目标是保持所有这些规则在实现 `State` 的结构体内部，我们在 `state` 中的值上调用 `content` 方法，并将文章实例（即 `self`）作为参数传递。然后，我们返回从使用 `state` 值上的 `content` 方法返回的值。

我们在 `Option` 上调用 `as_ref` 方法，因为我们想要 `Option` 内部值的引用而不是值的所有权。因为 `state` 是 `Option<Box<dyn State>>`，当我们调用 `as_ref` 时，返回 `Option<&Box<dyn State>>`。如果我们不调用 `as_ref`，我们会得到一个错误，因为我们无法从函数参数的借用 `&self` 中移出 `state`。

然后我们调用 `unwrap` 方法，我们知道它永远不会 panic，因为我们知道 `Post` 上的方法确保 `state` 在那些方法完成时将始终包含 `Some` 值。这是我们在第9章的[“当你拥有比编译器更多信息时”][more-info-than-rustc]<!-- ignore -->部分中讨论的情况之一，当我们知道 `None` 值永远不可能时，即使编译器无法理解这一点。

此时，当我们在 `&Box<dyn State>` 上调用 `content` 时，解引用强制转换将在 `&` 和 `Box` 上生效，以便 `content` 方法最终将在实现 `State` trait 的类型上调用。这意味着我们需要将 `content` 添加到 `State` trait 定义中，这就是我们将根据我们拥有的状态放置返回什么内容的逻辑的地方，如代码清单18-18所示。

<Listing number="18-18" file-name="src/lib.rs" caption="将 `content` 方法添加到 `State` trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-18/src/lib.rs:here}}
```

</Listing>

我们为 `content` 方法添加一个默认实现，返回空字符串切片。这意味着我们不需要在 `Draft` 和 `PendingReview` 结构体上实现 `content`。`Published` 结构体将覆盖 `content` 方法并返回 `post.content` 中的值。虽然方便，但在 `State` 上使用 `content` 方法来确定 `Post` 的内容模糊了 `State` 的责任和 `Post` 的责任之间的界限。

请注意，我们需要在此方法上使用生命周期注解，正如我们在第10章中讨论的。我们将 `post` 的引用作为参数，并返回该 `post` 的一部分的引用，因此返回引用的生命周期与 `post` 参数的生命周期相关。

我们完成了——代码清单18-11中的所有内容现在都可以工作！我们已经使用博客文章工作流的规则实现了状态模式。与规则相关的逻辑位于状态对象中，而不是分散在 `Post` 中。

> ### 为什么不使用枚举？
>
> 你可能想知道为什么我们不使用枚举，将不同的可能文章状态作为变体。这当然是一个可能的解决方案；试试它并比较最终结果，看看你更喜欢哪个！使用枚举的一个缺点是检查枚举值的每个地方都需要 `match` 表达式或类似的东西来处理每个可能的变体。这可能比这个 trait 对象解决方案更重复。

<!-- Old headings. Do not remove or links may break. -->

<a id="trade-offs-of-the-state-pattern"></a>

#### 评估状态模式

我们已经表明 Rust 能够实现面向对象的状态模式，以封装文章在每个状态中应该具有的不同类型的行为。`Post` 上的方法对各种行为一无所知。由于我们组织代码的方式，我们只需要在一个地方查看就能知道已发布文章的不同行为方式：`Published` 结构体上 `State` trait 的实现。

如果我们要创建一个不使用状态模式的替代实现，我们可能会在 `Post` 上的方法中使用 `match` 表达式，甚至在使用 `main` 代码中检查文章的状态并在这些地方更改行为。这意味着我们将不得不在几个地方查看以了解文章处于已发布状态的所有含义。

使用状态模式，`Post` 方法和我们使用 `Post` 的地方不需要 `match` 表达式，并且要添加新状态，我们只需要添加新结构体并在一个位置在该一个结构体上实现 trait 方法。

使用状态模式的实现易于扩展以添加更多功能。要查看使用状态模式维护代码的简单性，请尝试以下一些建议：

- 添加一个 `reject` 方法，将文章的状态从 `PendingReview` 更改回 `Draft`。
- 要求在状态可以更改为 `Published` 之前两次调用 `approve`。
- 只允许用户在文章处于 `Draft` 状态时添加文本内容。提示：让状态对象负责内容的可能更改，但不负责修改 `Post`。

状态模式的一个缺点是，由于状态实现状态之间的转换，一些状态彼此耦合。如果我们在 `PendingReview` 和 `Published` 之间添加另一个状态，例如 `Scheduled`，我们将不得不更改 `PendingReview` 中的代码以转换为 `Scheduled`。如果 `PendingReview` 不需要随着新状态的添加而更改，工作量会减少，但这意味着切换到另一种设计模式。

另一个缺点是我们复制了一些逻辑。为了消除一些重复，我们可能会尝试为 `State` trait 上的 `request_review` 和 `approve` 方法创建默认实现，返回 `self`。但是，这不会工作：当使用 `State` 作为 trait 对象时，trait 不知道具体的 `self` 究竟是什么，所以返回类型在编译时未知。（这是前面提到的 dyn 兼容性规则之一。）

其他重复包括 `Post` 上 `request_review` 和 `approve` 方法的类似实现。两种方法都使用 `Option::take` 与 `Post` 的 `state` 字段，如果 `state` 是 `Some`，它们委托给包装值的相同方法的实现，并将 `state` 字段的新值设置为结果。如果我们在 `Post` 上有许多遵循此模式的方法，我们可能会考虑定义一个宏来消除重复（参见第20章的[“宏”][macros]<!-- ignore -->部分）。

通过完全按照为面向对象语言定义的方式实现状态模式，我们没有充分利用 Rust 的优势。让我们看看我们可以对 `blog` crate 进行的一些更改，这些更改可以使无效状态和转换成为编译时错误。

### 将状态和行为编码为类型

我们将向你展示如何重新思考状态模式以获得不同的权衡。我们不是完全封装状态和转换，以便外部代码对它们一无所知，而是将状态编码为不同的类型。因此，Rust 的类型检查系统将通过发出编译器错误来防止在只允许已发布文章的地方使用草稿文章的尝试。

让我们考虑代码清单18-11中 `main` 的第一部分：

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:here}}
```

</Listing>

我们仍然允许使用 `Post::new` 在草稿状态中创建新文章，并允许向文章的内容添加文本。但是，不是在草稿文章上有一个返回空字符串的 `content` 方法，我们将使草稿文章根本没有 `content` 方法。这样，如果我们尝试获取草稿文章的内容，我们将得到一个编译器错误，告诉我们该方法不存在。因此，我们不可能意外地在生产环境中显示草稿文章内容，因为该代码甚至不会编译。代码清单18-19显示了 `Post` 结构体和 `DraftPost` 结构体的定义，以及每个结构体的方法。

<Listing number="18-19" file-name="src/lib.rs" caption="具有 `content` 方法的 `Post` 和不具有 `content` 方法的 `DraftPost`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-19/src/lib.rs}}
```

</Listing>

`Post` 和 `DraftPost` 结构体都有一个私有的 `content` 字段，用于存储博客文章文本。结构体不再有 `state` 字段，因为我们将状态的编码移动到结构体的类型中。`Post` 结构体将表示已发布的文章，它有一个 `content` 方法，返回 `content`。

我们仍然有一个 `Post::new` 函数，但它不返回 `Post` 的实例，而是返回 `DraftPost` 的实例。因为 `content` 是私有的，并且没有任何返回 `Post` 的函数，所以现在不可能创建 `Post` 的实例。

`DraftPost` 结构体有一个 `add_text` 方法，所以我们可以像以前一样向 `content` 添加文本，但请注意 `DraftPost` 没有定义 `content` 方法！所以现在程序确保所有文章都作为草稿文章开始，并且草稿文章没有可用于显示的内容。任何试图绕过这些约束的尝试都会导致编译器错误。

<!-- Old headings. Do not remove or links may break. -->

<a id="implementing-transitions-as-transformations-into-different-types"></a>

那么，我们如何获得已发布的文章？我们希望强制执行规则，即草稿文章必须在发布之前进行审查和批准。处于待审查状态的文章仍然不应显示任何内容。让我们通过添加另一个结构体 `PendingReviewPost` 来实现这些约束，在 `DraftPost` 上定义 `request_review` 方法以返回 `PendingReviewPost`，并在 `PendingReviewPost` 上定义 `approve` 方法以返回 `Post`，如代码清单18-20所示。

<Listing number="18-20" file-name="src/lib.rs" caption="通过调用 `DraftPost` 上的 `request_review` 创建的 `PendingReviewPost` 和将 `PendingReviewPost` 转换为已发布 `Post` 的 `approve` 方法">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-20/src/lib.rs:here}}
```

</Listing>

`request_review` 和 `approve` 方法取得 `self` 的所有权，从而消费 `DraftPost` 和 `PendingReviewPost` 实例，并将它们分别转换为 `PendingReviewPost` 和已发布的 `Post`。这样，在我们在它们上调用 `request_review` 之后，我们不会有任何残留的 `DraftPost` 实例，依此类推。`PendingReviewPost` 结构体没有在其上定义 `content` 方法，因此尝试读取其内容会导致编译器错误，就像 `DraftPost` 一样。因为获得具有定义的 `content` 方法的已发布 `Post` 实例的唯一方法是在 `PendingReviewPost` 上调用 `approve` 方法，而获得 `PendingReviewPost` 的唯一方法是在 `DraftPost` 上调用 `request_review` 方法，我们现在已将博客文章工作流编码到类型系统中。

但我们也必须对 `main` 做一些小的更改。`request_review` 和 `approve` 方法返回新实例而不是修改它们被调用的结构体，所以我们需要添加更多 `let post =` 遮蔽赋值来保存返回的实例。我们也不能有关于草稿和待审查文章内容为空字符串的断言，我们也不需要它们：我们不能再编译试图在这些状态下使用文章内容的代码。`main` 中的更新代码如代码清单18-21所示。

<Listing number="18-21" file-name="src/main.rs" caption="修改 `main` 以使用博客文章工作流的新实现">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-21/src/main.rs}}
```

</Listing>

我们需要对 `main` 进行的更改以重新分配 `post` 意味着此实现不再完全遵循面向对象的状态模式：状态之间的转换不再完全封装在 `Post` 实现中。但是，我们的收益是无效状态现在由于类型系统和在编译时发生的类型检查而变得不可能！这确保了某些错误，例如显示未发布文章的内容，将在它们进入生产之前被发现。

尝试本节开头建议的任务，在代码清单18-21之后的 `blog` crate 上，看看你对这个版本代码设计的看法。请注意，某些任务可能已经在此设计中完成。

我们已经看到，尽管 Rust 能够实现面向对象的设计模式，但其他模式，例如将状态编码到类型系统中，在 Rust 中也可用。这些模式有不同的权衡。虽然你可能非常熟悉面向对象模式，但重新思考问题以利用 Rust 的功能可以提供好处，例如在编译时防止某些错误。由于某些特性（如所有权），面向对象模式在 Rust 中不总是最佳解决方案，这些特性是面向对象语言所没有的。

## 总结

无论你在阅读本章后是否认为 Rust 是面向对象语言，你现在知道你可以使用 trait 对象在 Rust 中获得一些面向对象的特性。动态分发可以为你的代码提供一些灵活性，以换取一点运行时性能。你可以使用这种灵活性来实现可以帮助代码可维护性的面向对象模式。Rust 还具有其他特性，如所有权，这些是面向对象语言所没有的。面向对象模式不会总是利用 Rust 优势的最佳方式，但它是一个可用的选项。

接下来，我们将查看模式，这是 Rust 的另一个功能，它提供了很多灵活性。我们在整本书中简要地查看了它们，但还没有看到它们的全部能力。让我们开始吧！

[more-info-than-rustc]: ch09-03-to-panic-or-not-to-panic.html#cases-in-which-you-have-more-information-than-the-compiler
[macros]: ch20-05-macros.html#macros
