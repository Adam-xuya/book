## 引用与借用

清单 4-5 中元组代码的问题是我们必须将 `String` 返回到调用函数，以便我们仍然可以在调用 `calculate_length` 后使用 `String`，因为 `String` 被移动到 `calculate_length`。相反，我们可以提供对 `String` 值的引用。引用就像指针，它是一个地址，我们可以跟随该地址访问存储在该地址的数据；该数据由某个其他变量拥有。与指针不同，引用保证在该引用的生命周期内指向特定类型的有效值。

以下是您如何定义和使用 `calculate_length` 函数，该函数具有对对象的引用作为参数，而不是获取值的所有权：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:all}}
```

</Listing>

首先，注意变量声明和函数返回值中的所有元组代码都消失了。其次，注意我们将 `&s1` 传递给 `calculate_length`，在其定义中，我们接受 `&String` 而不是 `String`。这些与号表示引用，它们允许您引用某个值而不获取其所有权。图 4-6 描绘了这个概念。

<img alt="三个表：s 的表只包含指向 s1 表的指针。s1 的表包含 s1 的栈数据并指向堆上的字符串数据。" src="img/trpl04-06.svg" class="center" />

<span class="caption">图 4-6：指向 `String` `s1` 的 `&String` `s` 的图表</span>

> 注意：通过使用 `&` 进行引用的相反操作是_解引用_，这通过解引用运算符 `*` 完成。我们将在第 8 章看到解引用运算符的一些用途，并在第 15 章讨论解引用的详细信息。

让我们仔细看看这里的函数调用：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:here}}
```

`&s1` 语法让我们创建一个_引用_ `s1` 的值但不拥有它。因为引用不拥有它，当引用停止使用时，它指向的值不会被丢弃。

同样，函数的签名使用 `&` 表示参数 `s` 的类型是引用。让我们添加一些解释性注释：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-08-reference-with-annotations/src/main.rs:here}}
```

变量 `s` 有效的作用域与任何函数参数的作用域相同，但当 `s` 停止使用时，引用指向的值不会被丢弃，因为 `s` 没有所有权。当函数有引用作为参数而不是实际值时，我们不需要返回值以归还所有权，因为我们从未拥有所有权。

我们称创建引用的行为为_借用_。就像在现实生活中一样，如果一个人拥有某物，您可以从他们那里借用它。当您完成时，您必须归还它。您不拥有它。

那么，如果我们尝试修改我们正在借用的东西会发生什么？尝试清单 4-6 中的代码。剧透警告：它不起作用！

<Listing number="4-6" file-name="src/main.rs" caption="尝试修改借用的值">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-06/src/main.rs}}
```

</Listing>

这是错误：

```console
{{#include ../listings/ch04-understanding-ownership/listing-04-06/output.txt}}
```

就像变量默认是不可变的一样，引用也是不可变的。我们不允许修改我们有引用的东西。

### 可变引用

我们可以修复清单 4-6 中的代码，通过一些小的调整来允许我们修改借用的值，使用_可变引用_：

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-09-fixes-listing-04-06/src/main.rs}}
```

</Listing>

首先，我们将 `s` 更改为 `mut`。然后，我们在调用 `change` 函数的地方使用 `&mut s` 创建可变引用，并将函数签名更新为接受 `some_string: &mut String` 的可变引用。这非常清楚地表明 `change` 函数将改变它借用的值。

可变引用有一个很大的限制：如果您对值有可变引用，您就不能有对该值的任何其他引用。这段尝试创建两个对 `s` 的可变引用的代码将失败：

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/src/main.rs:here}}
```

</Listing>

这是错误：

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/output.txt}}
```

此错误说此代码无效，因为我们不能一次多次将 `s` 借用为可变。第一个可变借用位于 `r1` 中，必须持续到它在 `println!` 中使用为止，但在创建该可变引用和其使用之间，我们尝试在 `r2` 中创建另一个可变引用，它借用与 `r1` 相同的数据。

阻止同时有多个对同一数据的可变引用的限制允许改变，但以非常受控的方式进行。这是新的 Rustaceans 难以解决的问题，因为大多数语言允许您随时进行改变。拥有此限制的好处是 Rust 可以在编译时防止数据竞争。_数据竞争_类似于竞争条件，在发生以下三种行为时发生：

- 两个或多个指针同时访问相同的数据。
- 至少有一个指针用于写入数据。
- 没有用于同步数据访问的机制。

数据竞争会导致未定义行为，并且在运行时尝试追踪它们时可能难以诊断和修复；Rust 通过拒绝编译带有数据竞争的代码来防止此问题！

一如既往，我们可以使用花括号创建新作用域，允许多个可变引用，只是不能_同时_存在：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-11-muts-in-separate-scopes/src/main.rs:here}}
```

Rust 对组合可变和不可变引用强制执行类似的规则。此代码导致错误：

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/src/main.rs:here}}
```

这是错误：

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/output.txt}}
```

哇！我们_也不能_在对同一值有不可变引用的同时拥有可变引用。

不可变引用的用户不期望值突然从他们下面改变！但是，允许多个不可变引用，因为只是读取数据的人没有能力影响其他人对数据的读取。

请注意，引用的作用域从它被引入的地方开始，并持续到该引用最后一次使用。例如，此代码将编译，因为不可变引用的最后一次使用是在 `println!` 中，在引入可变引用之前：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-13-reference-scope-ends/src/main.rs:here}}
```

不可变引用 `r1` 和 `r2` 的作用域在它们最后一次使用的 `println!` 之后结束，这是在创建可变引用 `r3` 之前。这些作用域不重叠，所以允许此代码：编译器可以判断引用在作用域结束之前的某个点不再被使用。

尽管借用错误有时可能令人沮丧，但请记住，这是 Rust 编译器在早期（编译时而不是运行时）指出潜在错误并向您显示问题确切位置。然后，您不必追踪为什么您的数据不是您认为的那样。

### 悬垂引用

在具有指针的语言中，很容易错误地创建_悬垂指针_——引用可能已给予其他人的内存位置的指针——通过释放一些内存而保留指向该内存的指针。相比之下，在 Rust 中，编译器保证引用永远不会是悬垂引用：如果您有对某些数据的引用，编译器将确保数据不会在该数据的引用之前超出作用域。

让我们尝试创建一个悬垂引用来看看 Rust 如何通过编译时错误防止它们：

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/src/main.rs}}
```

</Listing>

这是错误：

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/output.txt}}
```

此错误消息指的是我们尚未涵盖的功能：生命周期。我们将在第 10 章详细讨论生命周期。但是，如果您忽略有关生命周期的部分，消息确实包含为什么此代码是问题的关键：

```text
此函数的返回类型包含借用的值，但没有值可以从中借用
```

让我们仔细看看我们的 `dangle` 代码的每个阶段究竟发生了什么：

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-15-dangling-reference-annotated/src/main.rs:here}}
```

</Listing>

因为 `s` 是在 `dangle` 内部创建的，当 `dangle` 的代码完成时，`s` 将被释放。但我们尝试返回对它的引用。这意味着此引用将指向无效的 `String`。那不好！Rust 不会让我们这样做。

这里的解决方案是直接返回 `String`：

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-16-no-dangle/src/main.rs:here}}
```

这工作起来没有任何问题。所有权被移出，没有任何东西被释放。

### 引用规则

让我们总结一下我们讨论的关于引用的内容：

- 在任何给定时间，您可以有_要么_一个可变引用_要么_任意数量的不可变引用。
- 引用必须始终有效。

接下来，我们将查看另一种引用：切片。
