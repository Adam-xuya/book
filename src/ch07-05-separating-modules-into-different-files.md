## 将模块分离到不同文件

到目前为止，本章中的所有示例都在一个文件中定义了多个模块。当模块变得很大时，你可能希望将它们的定义移动到单独的文件中，以使代码更容易导航。

例如，让我们从代码清单 7-17 中具有多个餐厅模块的代码开始。我们将模块提取到文件中，而不是在 crate 根文件中定义所有模块。在这种情况下，crate 根文件是 _src/lib.rs_，但此过程也适用于 crate 根文件是 _src/main.rs_ 的二进制 crate。

首先，我们将 `front_of_house` 模块提取到自己的文件中。删除 `front_of_house` 模块花括号内的代码，只保留 `mod front_of_house;` 声明，以便 _src/lib.rs_ 包含代码清单 7-21 中显示的代码。注意，这不会编译，直到我们在代码清单 7-22 中创建 _src/front_of_house.rs_ 文件。

<Listing number="7-21" file-name="src/lib.rs" caption="声明 `front_of_house` 模块，其主体将在 *src/front_of_house.rs* 中">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/lib.rs}}
```

</Listing>

接下来，将花括号中的代码放入名为 _src/front_of_house.rs_ 的新文件中，如代码清单 7-22 所示。编译器知道在此文件中查找，因为它在 crate 根中遇到了名为 `front_of_house` 的模块声明。

<Listing number="7-22" file-name="src/front_of_house.rs" caption="*src/front_of_house.rs* 中 `front_of_house` 模块内的定义">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/front_of_house.rs}}
```

</Listing>

注意，你只需要在模块树中使用 `mod` 声明 _一次_ 加载文件。一旦编译器知道文件是项目的一部分（并且知道代码在模块树中的位置，因为你放置了 `mod` 语句的位置），项目中的其他文件应该使用声明它的路径引用加载文件的代码，如["在模块树中引用项的路径"][paths]<!-- ignore -->部分所述。换句话说，`mod` _不是_ 你可能在其他编程语言中见过的"include"操作。

接下来，我们将 `hosting` 模块提取到自己的文件中。这个过程有点不同，因为 `hosting` 是 `front_of_house` 的子模块，而不是根模块的子模块。我们将 `hosting` 的文件放在一个新目录中，该目录将以模块树中其祖先命名，在这种情况下是 _src/front_of_house_。

要开始移动 `hosting`，我们将 _src/front_of_house.rs_ 更改为仅包含 `hosting` 模块的声明：

<Listing file-name="src/front_of_house.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house.rs}}
```

</Listing>

然后，我们创建一个 _src/front_of_house_ 目录和一个 _hosting.rs_ 文件来包含 `hosting` 模块中定义的定义：

<Listing file-name="src/front_of_house/hosting.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house/hosting.rs}}
```

</Listing>

如果我们将 _hosting.rs_ 放在 _src_ 目录中，编译器会期望 _hosting.rs_ 代码位于在 crate 根中声明的 `hosting` 模块中，而不是声明为 `front_of_house` 模块的子模块。编译器检查哪些文件的哪些模块代码的规则意味着目录和文件更紧密地匹配模块树。

> ### 备用文件路径
>
> 到目前为止，我们已经介绍了 Rust 编译器使用的最惯用的文件路径，但 Rust 也支持较旧的文件路径样式。对于在 crate 根中声明的名为 `front_of_house` 的模块，编译器将在以下位置查找模块的代码：
>
> - _src/front_of_house.rs_（我们介绍的）
> - _src/front_of_house/mod.rs_（较旧的样式，仍然支持的路径）
>
> 对于作为 `front_of_house` 子模块的名为 `hosting` 的模块，编译器将在以下位置查找模块的代码：
>
> - _src/front_of_house/hosting.rs_（我们介绍的）
> - _src/front_of_house/hosting/mod.rs_（较旧的样式，仍然支持的路径）
>
> 如果你对同一模块使用两种样式，你会得到编译器错误。在同一项目的不同模块中使用两种样式的混合是允许的，但可能会让导航你项目的人感到困惑。
>
> 使用名为 _mod.rs_ 的文件样式的缺点是，你的项目最终可能有很多名为 _mod.rs_ 的文件，当你在编辑器中同时打开它们时，这可能会令人困惑。

我们已经将每个模块的代码移动到单独的文件中，模块树保持不变。`eat_at_restaurant` 中的函数调用将在没有任何修改的情况下工作，即使定义位于不同的文件中。这种技术让你可以在模块增长时将模块移动到新文件。

注意，_src/lib.rs_ 中的 `pub use crate::front_of_house::hosting` 语句也没有更改，`use` 也不会对哪些文件作为 crate 的一部分编译产生影响。`mod` 关键字声明模块，Rust 在与模块同名的文件中查找进入该模块的代码。

## 总结

Rust 允许你将包拆分为多个 crate，将 crate 拆分为模块，以便你可以从另一个模块引用在一个模块中定义的项。你可以通过指定绝对路径或相对路径来实现这一点。这些路径可以使用 `use` 语句引入作用域，以便你可以在该作用域中多次使用该项时使用更短的路径。模块代码默认是私有的，但你可以通过添加 `pub` 关键字使定义公开。

在下一章中，我们将查看标准库中的一些集合数据结构，你可以在组织良好的代码中使用它们。

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
