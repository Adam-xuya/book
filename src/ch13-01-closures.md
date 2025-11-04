<!-- Old headings. Do not remove or links may break. -->

<a id="closures-anonymous-functions-that-can-capture-their-environment"></a>
<a id="closures-anonymous-functions-that-capture-their-environment"></a>

## 闭包

Rust 的闭包是匿名函数，你可以保存在变量中或作为参数传递给其他函数。你可以在一个地方创建闭包，然后在其他地方调用闭包以在不同上下文中评估它。与函数不同，闭包可以从定义它们的作用域中捕获值。我们将演示这些闭包功能如何允许代码重用和行为自定义。

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-an-abstraction-of-behavior-with-closures"></a>
<a id="refactoring-using-functions"></a>
<a id="refactoring-with-closures-to-store-code"></a>
<a id="capturing-the-environment-with-closures"></a>

### 捕获环境

我们将首先检查如何使用闭包从定义它们的环境中捕获值以供稍后使用。这是场景：每隔一段时间，我们的 T 恤公司会向邮件列表中的人赠送一件独家限量版 T 恤作为促销。邮件列表中的人可以选择将他们的最喜欢的颜色添加到他们的个人资料中。如果被选中获得免费 T 恤的人设置了他们最喜欢的颜色，他们会得到该颜色的 T 恤。如果该人没有指定最喜欢的颜色，他们会得到公司目前库存最多的任何颜色。

有许多方法可以实现这一点。对于这个例子，我们将使用一个名为 `ShirtColor` 的枚举，它有变体 `Red` 和 `Blue`（限制可用颜色的数量以简化）。我们用 `Inventory` 结构体表示公司的库存，该结构体有一个名为 `shirts` 的字段，包含表示当前库存的 T 恤颜色的 `Vec<ShirtColor>`。在 `Inventory` 上定义的 `giveaway` 方法获取免费 T 恤获胜者的可选 T 恤颜色偏好，并返回该人将获得的 T 恤颜色。此设置如代码清单 13-1 所示。

<Listing number="13-1" file-name="src/main.rs" caption="T 恤公司赠送情况">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-01/src/main.rs}}
```

</Listing>

在 `main` 中定义的 `store` 有两件蓝色 T 恤和一件红色 T 恤剩余，用于这个限量版促销。我们为偏好红色 T 恤的用户和没有任何偏好的用户调用 `giveaway` 方法。

同样，这段代码可以用多种方式实现，在这里，为了专注于闭包，我们坚持你已经学到的概念，除了使用闭包的 `giveaway` 方法的主体。在 `giveaway` 方法中，我们将用户偏好作为类型 `Option<ShirtColor>` 的参数获取，并在 `user_preference` 上调用 `unwrap_or_else` 方法。[`Option<T>` 上的 `unwrap_or_else` 方法][unwrap-or-else]<!-- ignore -->由标准库定义。它接受一个参数：一个不带任何参数并返回值 `T`（与 `Option<T>` 的 `Some` 变体中存储的类型相同，在这种情况下是 `ShirtColor`）的闭包。如果 `Option<T>` 是 `Some` 变体，`unwrap_or_else` 返回 `Some` 内部的值。如果 `Option<T>` 是 `None` 变体，`unwrap_or_else` 调用闭包并返回闭包返回的值。

我们将闭包表达式 `|| self.most_stocked()` 指定为 `unwrap_or_else` 的参数。这是一个本身不带参数的闭包（如果闭包有参数，它们将出现在两个垂直管道之间）。闭包的主体调用 `self.most_stocked()`。我们在这里定义闭包，如果需要结果，`unwrap_or_else` 的实现稍后将评估闭包。

运行此代码会打印以下内容：

```console
{{#include ../listings/ch13-functional-features/listing-13-01/output.txt}}
```

这里一个有趣的方面是我们传递了一个在当前 `Inventory` 实例上调用 `self.most_stocked()` 的闭包。标准库不需要了解我们定义的 `Inventory` 或 `ShirtColor` 类型，或者我们想在此场景中使用的逻辑。闭包捕获对 `self` `Inventory` 实例的不可变引用，并将其与我们指定的代码一起传递给 `unwrap_or_else` 方法。另一方面，函数无法以这种方式捕获它们的环境。

<!-- Old headings. Do not remove or links may break. -->

<a id="closure-type-inference-and-annotation"></a>

### 推断和注释闭包类型

函数和闭包之间还有更多差异。闭包通常不需要你像 `fn` 函数那样注释参数或返回值的类型。函数需要类型注释，因为类型是暴露给用户的显式接口的一部分。严格定义此接口对于确保每个人都同意函数使用和返回的值类型很重要。另一方面，闭包不会在这样的暴露接口中使用：它们存储在变量中，并且在不命名它们和不将它们暴露给库用户的情况下使用。

闭包通常很短，并且只在狭窄的上下文中相关，而不是在任何任意场景中。在这些有限的上下文中，编译器可以推断参数和返回类型，类似于它能够推断大多数变量的类型（在极少数情况下，编译器也需要闭包类型注释）。

与变量一样，如果我们想增加显式性和清晰度，可以添加类型注释，代价是比严格必要的更冗长。为闭包注释类型将看起来像代码清单 13-2 中显示的定义。在这个例子中，我们定义一个闭包并将其存储在变量中，而不是在我们将其作为参数传递的地方定义闭包，就像我们在代码清单 13-1 中所做的那样。

<Listing number="13-2" file-name="src/main.rs" caption="在闭包中添加参数和返回值类型的可选类型注释">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-02/src/main.rs:here}}
```

</Listing>

添加类型注释后，闭包的语法看起来更像函数的语法。这里，我们定义一个函数，将其参数加 1，以及一个具有相同行为的闭包，用于比较。我们添加了一些空格以对齐相关部分。这说明了闭包语法与函数语法相似，除了使用管道和可选语法的数量：

```rust,ignore
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

第一行显示函数定义，第二行显示完全注释的闭包定义。在第三行中，我们从闭包定义中移除类型注释。在第四行中，我们移除括号，这是可选的，因为闭包主体只有一个表达式。这些都是有效的定义，在调用时会产生相同的行为。`add_one_v3` 和 `add_one_v4` 行需要评估闭包才能编译，因为类型将从它们的使用中推断。这类似于 `let v = Vec::new();` 需要类型注释或插入一些类型的值到 `Vec` 中，以便 Rust 能够推断类型。

对于闭包定义，编译器将为它们的每个参数和返回值推断一个具体类型。例如，代码清单 13-3 显示了一个短闭包的定义，它只是返回它作为参数接收的值。这个闭包除了这个例子的目的之外不是很有用。注意，我们没有在定义中添加任何类型注释。因为没有类型注释，我们可以用任何类型调用闭包，我们在这里第一次用 `String` 这样做了。如果我们然后尝试用整数调用 `example_closure`，我们将得到错误。

<Listing number="13-3" file-name="src/main.rs" caption="尝试用两种不同类型调用类型已推断的闭包">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-03/src/main.rs:here}}
```

</Listing>

编译器给我们这个错误：

```console
{{#include ../listings/ch13-functional-features/listing-13-03/output.txt}}
```

我们第一次用 `String` 值调用 `example_closure` 时，编译器推断 `x` 的类型和闭包的返回类型为 `String`。这些类型然后被锁定在 `example_closure` 中的闭包中，当我们下次尝试用不同的类型使用同一个闭包时，我们得到一个类型错误。

### 捕获引用或移动所有权

闭包可以通过三种方式从环境中捕获值，这三种方式直接映射到函数可以采用参数的三种方式：不可变借用、可变借用和获取所有权。闭包将根据函数主体如何处理捕获的值来决定使用哪种方式。

在代码清单 13-4 中，我们定义一个闭包，它捕获对名为 `list` 的向量的不可变引用，因为它只需要一个不可变引用来打印值。

<Listing number="13-4" file-name="src/main.rs" caption="定义并调用捕获不可变引用的闭包">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-04/src/main.rs}}
```

</Listing>

此示例还说明了变量可以绑定到闭包定义，并且我们稍后可以通过使用变量名和括号来调用闭包，就像变量名是函数名一样。

因为我们可以同时对 `list` 有多个不可变引用，`list` 仍然可以从闭包定义之前的代码、闭包定义之后但在闭包调用之前的代码以及闭包调用之后的代码访问。此代码编译、运行并打印：

```console
{{#include ../listings/ch13-functional-features/listing-13-04/output.txt}}
```

接下来，在代码清单 13-5 中，我们更改闭包主体，使其向 `list` 向量添加元素。闭包现在捕获可变引用。

<Listing number="13-5" file-name="src/main.rs" caption="定义并调用捕获可变引用的闭包">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-05/src/main.rs}}
```

</Listing>

此代码编译、运行并打印：

```console
{{#include ../listings/ch13-functional-features/listing-13-05/output.txt}}
```

注意，在 `borrows_mutably` 闭包的定义和调用之间不再有 `println!`：当定义 `borrows_mutably` 时，它捕获对 `list` 的可变引用。我们在调用闭包之后不再使用闭包，所以可变借用结束。在闭包定义和闭包调用之间，不允许不可变借用来打印，因为当有可变借用时不允许其他借用。尝试在那里添加 `println!` 看看你得到什么错误消息！

如果你想要强制闭包获取它在环境中使用的值的所有权，即使闭包的主体并不严格需要所有权，你可以在参数列表之前使用 `move` 关键字。

这种技术主要在将闭包传递给新线程以移动数据以便新线程拥有它时很有用。我们将在第 16 章讨论并发时详细讨论线程以及为什么你想使用它们，但现在，让我们简要探索使用需要 `move` 关键字的闭包生成新线程。代码清单 13-6 显示了对代码清单 13-4 的修改，在新线程中而不是在主线程中打印向量。

<Listing number="13-6" file-name="src/main.rs" caption="使用 `move` 强制线程的闭包获取 `list` 的所有权">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-06/src/main.rs}}
```

</Listing>

我们生成一个新线程，将闭包作为参数提供给线程运行。闭包主体打印出列表。在代码清单 13-4 中，闭包只使用不可变引用捕获 `list`，因为这是打印它所需的 `list` 的最少访问量。在这个例子中，即使闭包主体仍然只需要不可变引用，我们也需要通过将 `move` 关键字放在闭包定义的开头来指定应该将 `list` 移动到闭包中。如果主线程在调用新线程的 `join` 之前执行更多操作，新线程可能在主线程的其余部分完成之前完成，或者主线程可能首先完成。如果主线程保持 `list` 的所有权但在新线程之前结束并丢弃 `list`，线程中的不可变引用将无效。因此，编译器要求将 `list` 移动到给新线程的闭包中，以便引用有效。尝试移除 `move` 关键字或在主线程中定义闭包后使用 `list`，看看你得到什么编译器错误！

<!-- Old headings. Do not remove or links may break. -->

<a id="storing-closures-using-generic-parameters-and-the-fn-traits"></a>
<a id="limitations-of-the-cacher-implementation"></a>
<a id="moving-captured-values-out-of-the-closure-and-the-fn-traits"></a>
<a id="moving-captured-values-out-of-closures-and-the-fn-traits"></a>

### 将捕获的值移出闭包

一旦闭包从定义闭包的环境中捕获了引用或捕获了值的所有权（从而影响是否有任何内容被移动 _到_ 闭包中），闭包主体中的代码定义了在稍后评估闭包时对引用或值发生什么（从而影响是否有任何内容被移动 _出_ 闭包）。

闭包主体可以执行以下任何操作：将捕获的值移出闭包、改变捕获的值、既不移动也不改变值，或者根本不从环境中捕获任何内容。

闭包从环境中捕获和处理值的方式影响闭包实现哪些 trait，而 trait 是函数和结构体可以指定它们可以使用哪些类型的闭包的方式。闭包将根据闭包主体如何处理值，以累加方式自动实现这些 `Fn` trait 中的一个、两个或全部三个：

* `FnOnce` 适用于可以调用一次的闭包。所有闭包至少实现此 trait，因为所有闭包都可以调用。将捕获的值移出其主体的闭包将只实现 `FnOnce` 而不实现其他 `Fn` trait，因为它只能调用一次。
* `FnMut` 适用于不将捕获的值移出其主体但可能改变捕获值的闭包。这些闭包可以调用多次。
* `Fn` 适用于不将捕获的值移出其主体且不改变捕获值的闭包，以及不捕获其环境中任何内容的闭包。这些闭包可以多次调用而不改变其环境，这在例如多次并发调用闭包的情况下很重要。

让我们看看我们在代码清单 13-1 中使用的 `Option<T>` 上的 `unwrap_or_else` 方法的定义：

```rust,ignore
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

回想一下，`T` 是表示 `Option` 的 `Some` 变体中值类型的泛型类型。类型 `T` 也是 `unwrap_or_else` 函数的返回类型：例如，在 `Option<String>` 上调用 `unwrap_or_else` 的代码将获得 `String`。

接下来，注意 `unwrap_or_else` 函数有额外的泛型类型参数 `F`。`F` 类型是名为 `f` 的参数的类型，这是我们在调用 `unwrap_or_else` 时提供的闭包。

在泛型类型 `F` 上指定的 trait 约束是 `FnOnce() -> T`，这意味着 `F` 必须能够被调用一次，不接受参数，并返回 `T`。在 trait 约束中使用 `FnOnce` 表示 `unwrap_or_else` 不会多次调用 `f` 的约束。在 `unwrap_or_else` 的主体中，我们可以看到，如果 `Option` 是 `Some`，`f` 不会被调用。如果 `Option` 是 `None`，`f` 将被调用一次。因为所有闭包都实现 `FnOnce`，`unwrap_or_else` 接受所有三种类型的闭包，并且尽可能灵活。

> 注意：如果我们想做的事情不需要从环境中捕获值，我们可以在需要实现 `Fn` trait 之一的地方使用函数名而不是闭包。例如，在 `Option<Vec<T>>` 值上，如果值是 `None`，我们可以调用 `unwrap_or_else(Vec::new)` 来获得新的空向量。编译器自动实现适用于函数定义的任何 `Fn` trait。

现在让我们看看标准库方法 `sort_by_key`，它定义在切片上，看看它如何与 `unwrap_or_else` 不同，以及为什么 `sort_by_key` 使用 `FnMut` 而不是 `FnOnce` 作为 trait 约束。闭包以对被考虑的切片中当前项的引用的形式获得一个参数，并返回可以排序的类型 `K` 的值。当你想按每个项的特定属性对切片进行排序时，此函数很有用。在代码清单 13-7 中，我们有一个 `Rectangle` 实例列表，我们使用 `sort_by_key` 按它们的 `width` 属性从低到高排序。

<Listing number="13-7" file-name="src/main.rs" caption="使用 `sort_by_key` 按宽度对矩形排序">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-07/src/main.rs}}
```

</Listing>

此代码打印：

```console
{{#include ../listings/ch13-functional-features/listing-13-07/output.txt}}
```

`sort_by_key` 定义为接受 `FnMut` 闭包的原因是它多次调用闭包：对切片中的每个项一次。闭包 `|r| r.width` 不从其环境中捕获、改变或移动任何内容，所以它满足 trait 约束要求。

相比之下，代码清单 13-8 显示了一个只实现 `FnOnce` trait 的闭包示例，因为它从环境中移出一个值。编译器不会让我们将这个闭包与 `sort_by_key` 一起使用。

<Listing number="13-8" file-name="src/main.rs" caption="尝试将 `FnOnce` 闭包与 `sort_by_key` 一起使用">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-08/src/main.rs}}
```

</Listing>

这是一种人为的、复杂的方式（不起作用）来尝试计算 `sort_by_key` 在对 `list` 排序时调用闭包的次数。此代码尝试通过将 `value`（来自闭包环境的 `String`）推入 `sort_operations` 向量来进行此计数。闭包捕获 `value`，然后通过将 `value` 的所有权转移给 `sort_operations` 向量将 `value` 移出闭包。此闭包可以被调用一次；尝试第二次调用它将不起作用，因为 `value` 将不再在环境中被推入 `sort_operations` 再次！因此，此闭包只实现 `FnOnce`。当我们尝试编译此代码时，我们得到此错误，即 `value` 不能从闭包中移出，因为闭包必须实现 `FnMut`：

```console
{{#include ../listings/ch13-functional-features/listing-13-08/output.txt}}
```

错误指向闭包主体中将 `value` 移出环境的行。要修复此问题，我们需要更改闭包主体，使其不从环境中移出值。在环境中保持计数器并在闭包主体中增加其值是计算闭包被调用次数的更直接方法。代码清单 13-9 中的闭包与 `sort_by_key` 一起工作，因为它只捕获对 `num_sort_operations` 计数器的可变引用，因此可以多次调用。

<Listing number="13-9" file-name="src/main.rs" caption="允许使用 `FnMut` 闭包与 `sort_by_key`。">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-09/src/main.rs}}
```

</Listing>

`Fn` trait 在定义或使用利用闭包的函数或类型时很重要。在下一节中，我们将讨论迭代器。许多迭代器方法接受闭包参数，所以在继续时请记住这些闭包细节！

[unwrap-or-else]: ../std/option/enum.Option.html#method.unwrap_or_else
