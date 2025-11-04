## 使用迭代器处理一系列项

迭代器模式允许你依次对一系列项执行某些任务。迭代器负责迭代每个项的逻辑并确定序列何时完成。当你使用迭代器时，你不必自己重新实现该逻辑。

在 Rust 中，迭代器是 _惰性的_，这意味着它们在你调用消耗迭代器以使用它的方法之前没有任何效果。例如，代码清单 13-10 中的代码通过在 `Vec<T>` 上调用定义的 `iter` 方法在向量 `v1` 中的项上创建迭代器。这段代码本身不做任何有用的事情。

<Listing number="13-10" file-name="src/main.rs" caption="创建迭代器">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-10/src/main.rs:here}}
```

</Listing>

迭代器存储在 `v1_iter` 变量中。一旦我们创建了迭代器，我们可以以多种方式使用它。在代码清单 3-5 中，我们使用 `for` 循环迭代数组，对其每个项执行一些代码。在底层，这隐式创建然后消耗迭代器，但直到现在我们才详细说明它是如何工作的。

在代码清单 13-11 的示例中，我们将迭代器的创建与 `for` 循环中迭代器的使用分开。当使用 `v1_iter` 中的迭代器调用 `for` 循环时，迭代器中的每个元素在循环的一次迭代中使用，这会打印出每个值。

<Listing number="13-11" file-name="src/main.rs" caption="在 `for` 循环中使用迭代器">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-11/src/main.rs:here}}
```

</Listing>

在标准库不提供迭代器的语言中，你可能通过将变量从索引 0 开始，使用该变量索引到向量以获取值，并在循环中递增变量值直到它达到向量中项的总数来编写相同的功能。

迭代器为你处理所有这些逻辑，减少你可能弄乱的重复代码。迭代器为你提供更多灵活性，可以在许多不同类型的序列中使用相同的逻辑，不仅仅是你可以索引的数据结构，如向量。让我们检查迭代器如何做到这一点。

### `Iterator` Trait 和 `next` 方法

所有迭代器都实现一个名为 `Iterator` 的 trait，它在标准库中定义。trait 的定义如下：

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // methods with default implementations elided
}
```

注意，此定义使用一些新语法：`type Item` 和 `Self::Item`，它们正在定义与此 trait 关联的类型。我们将在第 20 章深入讨论关联类型。现在，你只需要知道这段代码说明实现 `Iterator` trait 需要你也定义 `Item` 类型，并且此 `Item` 类型在 `next` 方法的返回类型中使用。换句话说，`Item` 类型将是迭代器返回的类型。

`Iterator` trait 只需要实现者定义一个方法：`next` 方法，它一次返回迭代器的一个项，包装在 `Some` 中，当迭代结束时，返回 `None`。

我们可以直接在迭代器上调用 `next` 方法；代码清单 13-12 演示了从向量创建的迭代器上重复调用 `next` 返回的值。

<Listing number="13-12" file-name="src/lib.rs" caption="在迭代器上调用 `next` 方法">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-12/src/lib.rs:here}}
```

</Listing>

注意，我们需要使 `v1_iter` 可变：在迭代器上调用 `next` 方法会更改迭代器用来跟踪它在序列中的位置的内部状态。换句话说，这段代码 _消耗_ 或使用迭代器。每次调用 `next` 都会消耗迭代器中的一个项。当我们使用 `for` 循环时，我们不需要使 `v1_iter` 可变，因为循环获取 `v1_iter` 的所有权并在幕后使其可变。

还要注意，我们从 `next` 调用中得到的值是对向量中值的不可变引用。`iter` 方法产生不可变引用的迭代器。如果我们想创建一个获取 `v1` 所有权并返回拥有值的迭代器，我们可以调用 `into_iter` 而不是 `iter`。类似地，如果我们想迭代可变引用，我们可以调用 `iter_mut` 而不是 `iter`。

### 消耗迭代器的方法

`Iterator` trait 有许多不同的方法，这些方法具有标准库提供的默认实现；你可以通过在标准库 API 文档中查找 `Iterator` trait 来了解这些方法。这些方法中的一些在其定义中调用 `next` 方法，这就是为什么在实现 `Iterator` trait 时要求你实现 `next` 方法。

调用 `next` 的方法称为 _消耗适配器_，因为调用它们会使用迭代器。一个例子是 `sum` 方法，它获取迭代器的所有权并通过重复调用 `next` 迭代项，从而消耗迭代器。在迭代过程中，它将每个项添加到运行总计中，并在迭代完成时返回总计。代码清单 13-13 有一个说明 `sum` 方法使用的测试。

<Listing number="13-13" file-name="src/lib.rs" caption="调用 `sum` 方法以获取迭代器中所有项的总计">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-13/src/lib.rs:here}}
```

</Listing>

在调用 `sum` 之后，我们不允许使用 `v1_iter`，因为 `sum` 获取我们调用它的迭代器的所有权。

### 产生其他迭代器的方法

_迭代器适配器_ 是在 `Iterator` trait 上定义的不消耗迭代器的方法。相反，它们通过改变原始迭代器的某些方面来产生不同的迭代器。

代码清单 13-14 显示了一个调用迭代器适配器方法 `map` 的示例，它接受一个闭包，在迭代项时在每个项上调用。`map` 方法返回一个产生修改项的迭代器。这里的闭包创建一个新迭代器，其中向量中的每个项将增加 1。

<Listing number="13-14" file-name="src/main.rs" caption="调用迭代器适配器 `map` 以创建新迭代器">

```rust,not_desired_behavior
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-14/src/main.rs:here}}
```

</Listing>

但是，此代码产生警告：

```console
{{#include ../listings/ch13-functional-features/listing-13-14/output.txt}}
```

代码清单 13-14 中的代码不做任何事情；我们指定的闭包从未被调用。警告提醒我们原因：迭代器适配器是惰性的，我们需要在这里消耗迭代器。

为了修复此警告并消耗迭代器，我们将使用 `collect` 方法，我们在代码清单 12-1 中与 `env::args` 一起使用了它。此方法消耗迭代器并将结果值收集到集合数据类型中。

在代码清单 13-15 中，我们将迭代从 `map` 调用返回的迭代器的结果收集到向量中。此向量最终将包含原始向量中的每个项，增加 1。

<Listing number="13-15" file-name="src/main.rs" caption="调用 `map` 方法以创建新迭代器，然后调用 `collect` 方法以消耗新迭代器并创建向量">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-15/src/main.rs:here}}
```

</Listing>

因为 `map` 接受闭包，我们可以指定我们想对每个项执行的任何操作。这是闭包如何让你自定义某些行为同时重用 `Iterator` trait 提供的迭代行为的一个很好的例子。

你可以链接多个对迭代器适配器的调用，以可读的方式执行复杂的操作。但是因为所有迭代器都是惰性的，你必须调用消耗适配器方法之一来从迭代器适配器调用中获得结果。

<!-- Old headings. Do not remove or links may break. -->

<a id="using-closures-that-capture-their-environment"></a>

### 捕获其环境的闭包

许多迭代器适配器接受闭包作为参数，通常我们将指定为迭代器适配器参数的闭包将是捕获其环境的闭包。

对于这个例子，我们将使用接受闭包的 `filter` 方法。闭包从迭代器获取一个项并返回 `bool`。如果闭包返回 `true`，该值将包含在 `filter` 产生的迭代中。如果闭包返回 `false`，该值将不包含在内。

在代码清单 13-16 中，我们使用 `filter` 和一个从环境中捕获 `shoe_size` 变量的闭包来迭代 `Shoe` 结构体实例的集合。它将只返回指定大小的鞋子。

<Listing number="13-16" file-name="src/lib.rs" caption="使用 `filter` 方法和捕获 `shoe_size` 的闭包">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-16/src/lib.rs}}
```

</Listing>

`shoes_in_size` 函数获取鞋子向量和鞋子大小的所有权作为参数。它返回一个只包含指定大小的鞋子的向量。

在 `shoes_in_size` 的主体中，我们调用 `into_iter` 以创建获取向量所有权的迭代器。然后，我们调用 `filter` 以将该迭代器适配为只包含闭包返回 `true` 的元素的迭代器。

闭包从环境中捕获 `shoe_size` 参数并将该值与每只鞋子的大小进行比较，只保留指定大小的鞋子。最后，调用 `collect` 将适配迭代器返回的值收集到函数返回的向量中。

测试显示，当我们调用 `shoes_in_size` 时，我们只得到与我们指定的值大小相同的鞋子。
