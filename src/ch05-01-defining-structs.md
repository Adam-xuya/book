## 定义和实例化结构体

结构体类似于在["元组类型"][tuples]<!--
ignore -->部分中讨论的元组，因为它们都包含多个相关值。与元组一样，结构体的各个部分可以是不同类型。与元组不同，在结构体中，您将为每个数据片段命名，以便清楚值的含义。添加这些名称意味着结构体比元组更灵活：您不必依赖数据的顺序来指定或访问实例的值。

要定义结构体，我们输入关键字 `struct` 并命名整个结构体。结构体的名称应该描述被分组在一起的数据片段的重要性。然后，在花括号内，我们定义数据片段的名称和类型，我们称之为_字段_。例如，清单 5-1 显示了一个存储用户帐户信息的结构体。

<Listing number="5-1" file-name="src/main.rs" caption="`User` 结构体定义">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-01/src/main.rs:here}}
```

</Listing>

在我们定义结构体后使用它，我们通过为每个字段指定具体值来创建该结构体的_实例_。我们通过说明结构体的名称来创建实例，然后添加包含_`key:
value`_ 对的花括号，其中键是字段的名称，值是我们想要存储在这些字段中的数据。我们不必按照在结构体中声明它们的相同顺序指定字段。换句话说，结构体定义就像类型的通用模板，实例用特定数据填充该模板以创建该类型的值。例如，我们可以声明一个特定用户，如清单 5-2 所示。

<Listing number="5-2" file-name="src/main.rs" caption="创建 `User` 结构体的实例">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-02/src/main.rs:here}}
```

</Listing>

要从结构体中获取特定值，我们使用点符号。例如，要访问此用户的电子邮件地址，我们使用 `user1.email`。如果实例是可变的，我们可以通过使用点符号并将值赋值给特定字段来更改值。清单 5-3 显示了如何更改可变 `User` 实例的 `email` 字段中的值。

<Listing number="5-3" file-name="src/main.rs" caption="更改 `User` 实例的 `email` 字段中的值">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-03/src/main.rs:here}}
```

</Listing>

请注意，整个实例必须是可变的；Rust 不允许我们只将某些字段标记为可变。与任何表达式一样，我们可以构造结构体的新实例作为函数体中的最后一个表达式，以隐式返回该新实例。

清单 5-4 显示了一个 `build_user` 函数，它返回一个具有给定电子邮件和用户名的 `User` 实例。`active` 字段获得值 `true`，`sign_in_count` 获得值 `1`。

<Listing number="5-4" file-name="src/main.rs" caption="一个接受电子邮件和用户名并返回 `User` 实例的 `build_user` 函数">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-04/src/main.rs:here}}
```

</Listing>

使用与结构体字段相同的名称命名函数参数是有意义的，但必须重复 `email` 和 `username` 字段名称和变量有点繁琐。如果结构体有更多字段，重复每个名称会更加烦人。幸运的是，有一个方便的简写！

<!-- Old headings. Do not remove or links may break. -->

<a id="using-the-field-init-shorthand-when-variables-and-fields-have-the-same-name"></a>

### 使用字段初始化简写

因为在清单 5-4 中参数名称和结构体字段名称完全相同，我们可以使用_字段初始化简写_语法重写 `build_user`，使其行为完全相同，但没有 `username` 和 `email` 的重复，如清单 5-5 所示。

<Listing number="5-5" file-name="src/main.rs" caption="一个使用字段初始化简写的 `build_user` 函数，因为 `username` 和 `email` 参数与结构体字段同名">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-05/src/main.rs:here}}
```

</Listing>

这里，我们创建 `User` 结构体的新实例，它有一个名为 `email` 的字段。我们希望将 `email` 字段的值设置为 `build_user` 函数的 `email` 参数中的值。因为 `email` 字段和 `email` 参数具有相同的名称，我们只需要编写 `email` 而不是 `email: email`。

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-instances-from-other-instances-with-struct-update-syntax"></a>

### 使用结构体更新语法创建实例

创建一个结构体的新实例通常很有用，该实例包含来自同一类型的另一个实例的大部分值，但更改其中一些值。您可以使用结构体更新语法来做到这一点。

首先，在清单 5-6 中，我们显示了如何以常规方式在 `user2` 中创建新 `User` 实例，不使用更新语法。我们为 `email` 设置新值，但否则使用我们在清单 5-2 中创建的 `user1` 的相同值。

<Listing number="5-6" file-name="src/main.rs" caption="使用来自 `user1` 的所有值（除了一个）创建新的 `User` 实例">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-06/src/main.rs:here}}
```

</Listing>

使用结构体更新语法，我们可以用更少的代码实现相同的效果，如清单 5-7 所示。语法 `..` 指定未显式设置的剩余字段应与给定实例中的字段具有相同的值。

<Listing number="5-7" file-name="src/main.rs" caption="使用结构体更新语法为 `User` 实例设置新的 `email` 值，但使用来自 `user1` 的其余值">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-07/src/main.rs:here}}
```

</Listing>

清单 5-7 中的代码也在 `user2` 中创建一个实例，该实例的 `email` 值不同，但 `username`、`active` 和 `sign_in_count` 字段的值与 `user1` 相同。`..user1` 必须放在最后，以指定任何剩余字段应从 `user1` 中的相应字段获取其值，但我们可以选择以任何顺序为我们想要的任意数量的字段指定值，而不管字段在结构体定义中的顺序。

请注意，结构体更新语法使用 `=` 像赋值一样；这是因为它移动数据，就像我们在["变量和数据交互：移动"][move]<!-- ignore -->部分中看到的那样。在此示例中，我们在创建 `user2` 后不能再使用 `user1`，因为 `user1` 的 `username` 字段中的 `String` 被移动到 `user2`。如果我们为 `user2` 的 `email` 和 `username` 都提供了新的 `String` 值，因此只使用了 `user1` 的 `active` 和 `sign_in_count` 值，那么 `user1` 在创建 `user2` 后仍然有效。`active` 和 `sign_in_count` 都是实现 `Copy` trait 的类型，所以我们在["仅栈数据：复制"][copy]<!-- ignore -->部分讨论的行为将适用。我们也可以在此示例中仍然使用 `user1.email`，因为它的值没有从 `user1` 移出。

<!-- Old headings. Do not remove or links may break. -->

<a id="using-tuple-structs-without-named-fields-to-create-different-types"></a>

### 使用元组结构体创建不同类型

Rust 还支持看起来类似于元组的结构体，称为_元组结构体_。元组结构体具有结构体名称提供的附加含义，但没有与其字段关联的名称；相反，它们只有字段的类型。当您想给整个元组一个名称并使元组成为与其他元组不同的类型，以及当像常规结构体中那样命名每个字段会很冗长或冗余时，元组结构体很有用。

要定义元组结构体，以 `struct` 关键字和结构体名称开头，后跟元组中的类型。例如，这里我们定义并使用两个名为 `Color` 和 `Point` 的元组结构体：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-01-tuple-structs/src/main.rs}}
```

</Listing>

请注意，`black` 和 `origin` 值是不同类型，因为它们是不同元组结构体的实例。您定义的每个结构体都是其自己的类型，即使结构体内的字段可能具有相同的类型。例如，接受 `Color` 类型参数的函数不能接受 `Point` 作为参数，即使两种类型都由三个 `i32` 值组成。否则，元组结构体实例类似于元组，因为您可以将它们解构为它们的各个部分，并且可以使用 `.` 后跟索引来访问单个值。与元组不同，元组结构体要求您在解构它们时命名结构体的类型。例如，我们会编写 `let Point(x, y, z) = origin;` 来将 `origin` 点中的值解构为名为 `x`、`y` 和 `z` 的变量。

<!-- Old headings. Do not remove or links may break. -->

<a id="unit-like-structs-without-any-fields"></a>

### 定义类单元结构体

您也可以定义没有任何字段的结构体！这些被称为_类单元结构体_，因为它们的行为类似于 `()`，即我们在["元组类型"][tuples]<!-- ignore -->部分中提到的单元类型。当您需要在某种类型上实现 trait 但没有任何要存储在类型本身中的数据时，类单元结构体可能很有用。我们将在第 10 章讨论 trait。以下是声明和实例化名为 `AlwaysEqual` 的单元结构体的示例：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-04-unit-like-structs/src/main.rs}}
```

</Listing>

要定义 `AlwaysEqual`，我们使用 `struct` 关键字、我们想要的名称，然后是分号。不需要花括号或括号！然后，我们可以以类似的方式在 `subject` 变量中获取 `AlwaysEqual` 的实例：使用我们定义的名称，没有任何花括号或括号。想象一下，稍后我们将为此类型实现行为，使得 `AlwaysEqual` 的每个实例始终等于任何其他类型的每个实例，也许是为了测试目的而具有已知结果。我们不需要任何数据来实现该行为！您将在第 10 章看到如何定义 trait 并在任何类型上实现它们，包括类单元结构体。

> ### 结构体数据的所有权
>
> 在清单 5-1 中的 `User` 结构体定义中，我们使用了拥有的 `String` 类型而不是 `&str` 字符串切片类型。这是一个有意的选择，因为我们希望此结构体的每个实例都拥有其所有数据，并且该数据在整个结构体有效期间都有效。
>
> 结构体也可以存储对由其他东西拥有的数据的引用，但要这样做需要使用_生命周期_，这是 Rust 的一个功能，我们将在第 10 章讨论。生命周期确保结构体引用的数据在结构体有效期间有效。假设您尝试在不指定生命周期的情况下在结构体中存储引用，如下面的 *src/main.rs* 中所示；这将不起作用：
>
> <Listing file-name="src/main.rs">
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore,does_not_compile
> struct User {
>     active: bool,
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
> }
>
> fn main() {
>     let user1 = User {
>         active: true,
>         username: "someusername123",
>         email: "someone@example.com",
>         sign_in_count: 1,
>     };
> }
> ```
>
> </Listing>
>
> 编译器会抱怨它需要生命周期说明符：
>
> ```console
> $ cargo run
>    Compiling structs v0.1.0 (file:///projects/structs)
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:3:15
>   |
> 3 |     username: &str,
>   |               ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 ~     username: &'a str,
>   |
>
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:4:12
>   |
> 4 |     email: &str,
>   |            ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 |     username: &str,
> 4 ~     email: &'a str,
>   |
>
> For more information about this error, try `rustc --explain E0106`.
> error: could not compile `structs` (bin "structs") due to 2 previous errors
> ```
>
> 在第 10 章中，我们将讨论如何修复这些错误，以便您可以在结构体中存储引用，但现在，我们将使用拥有的类型（如 `String`）而不是引用（如 `&str`）来修复此类错误。

<!-- manual-regeneration
for the error above
after running update-rustc.sh:
pbcopy < listings/ch05-using-structs-to-structure-related-data/no-listing-02-reference-in-struct/output.txt
paste above
add `> ` before every line -->

[tuples]: ch03-02-data-types.html#the-tuple-type
[move]: ch04-01-what-is-ownership.html#variables-and-data-interacting-with-move
[copy]: ch04-01-what-is-ownership.html#stack-only-data-copy
