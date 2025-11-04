## 附录 B：运算符和符号

本附录包含 Rust 语法的词汇表，包括运算符和其他符号，它们单独出现或在路径、泛型、trait 限制、宏、属性、注释、元组和括号的上下文中出现。

### 运算符

表 B-1 包含 Rust 中的运算符、运算符在上下文中的示例、简短说明以及该运算符是否可重载。如果运算符可重载，则列出用于重载该运算符的相关 trait。

<span class="caption">表 B-1：运算符</span>

| Operator                  | Example                                                 | Explanation                                                           | Overloadable?  |
| ------------------------- | ------------------------------------------------------- | --------------------------------------------------------------------- | -------------- |
| `!`                       | `ident!(...)`, `ident!{...}`, `ident![...]`             | Macro expansion                                                       |                |
| `!`                       | `!expr`                                                 | Bitwise or logical complement                                         | `Not`          |
| `!=`                      | `expr != expr`                                          | Nonequality comparison                                                | `PartialEq`    |
| `%`                       | `expr % expr`                                           | Arithmetic remainder                                                  | `Rem`          |
| `%=`                      | `var %= expr`                                           | Arithmetic remainder and assignment                                   | `RemAssign`    |
| `&`                       | `&expr`, `&mut expr`                                    | Borrow                                                                |                |
| `&`                       | `&type`, `&mut type`, `&'a type`, `&'a mut type`        | Borrowed pointer type                                                 |                |
| `&`                       | `expr & expr`                                           | Bitwise AND                                                           | `BitAnd`       |
| `&=`                      | `var &= expr`                                           | Bitwise AND and assignment                                            | `BitAndAssign` |
| `&&`                      | `expr && expr`                                          | Short-circuiting logical AND                                          |                |
| `*`                       | `expr * expr`                                           | Arithmetic multiplication                                             | `Mul`          |
| `*=`                      | `var *= expr`                                           | Arithmetic multiplication and assignment                              | `MulAssign`    |
| `*`                       | `*expr`                                                 | Dereference                                                           | `Deref`        |
| `*`                       | `*const type`, `*mut type`                              | Raw pointer                                                           |                |
| `+`                       | `trait + trait`, `'a + trait`                           | Compound type constraint                                              |                |
| `+`                       | `expr + expr`                                           | Arithmetic addition                                                   | `Add`          |
| `+=`                      | `var += expr`                                           | Arithmetic addition and assignment                                    | `AddAssign`    |
| `,`                       | `expr, expr`                                            | Argument and element separator                                        |                |
| `-`                       | `- expr`                                                | Arithmetic negation                                                   | `Neg`          |
| `-`                       | `expr - expr`                                           | Arithmetic subtraction                                                | `Sub`          |
| `-=`                      | `var -= expr`                                           | Arithmetic subtraction and assignment                                 | `SubAssign`    |
| `->`                      | `fn(...) -> type`, <code>&vert;...&vert; -> type</code> | Function and closure return type                                      |                |
| `.`                       | `expr.ident`                                            | Field access                                                          |                |
| `.`                       | `expr.ident(expr, ...)`                                 | Method call                                                           |                |
| `.`                       | `expr.0`, `expr.1`, and so on                           | Tuple indexing                                                        |                |
| `..`                      | `..`, `expr..`, `..expr`, `expr..expr`                  | Right-exclusive range literal                                         | `PartialOrd`   |
| `..=`                     | `..=expr`, `expr..=expr`                                | Right-inclusive range literal                                         | `PartialOrd`   |
| `..`                      | `..expr`                                                | Struct literal update syntax                                          |                |
| `..`                      | `variant(x, ..)`, `struct_type { x, .. }`               | "And the rest" pattern binding                                        |                |
| `...`                     | `expr...expr`                                           | (Deprecated, use `..=` instead) In a pattern: inclusive range pattern |                |
| `/`                       | `expr / expr`                                           | Arithmetic division                                                   | `Div`          |
| `/=`                      | `var /= expr`                                           | Arithmetic division and assignment                                    | `DivAssign`    |
| `:`                       | `pat: type`, `ident: type`                              | Constraints                                                           |                |
| `:`                       | `ident: expr`                                           | Struct field initializer                                              |                |
| `:`                       | `'a: loop {...}`                                        | Loop label                                                            |                |
| `;`                       | `expr;`                                                 | Statement and item terminator                                         |                |
| `;`                       | `[...; len]`                                            | Part of fixed-size array syntax                                       |                |
| `<<`                      | `expr << expr`                                          | Left-shift                                                            | `Shl`          |
| `<<=`                     | `var <<= expr`                                          | Left-shift and assignment                                             | `ShlAssign`    |
| `<`                       | `expr < expr`                                           | Less than comparison                                                  | `PartialOrd`   |
| `<=`                      | `expr <= expr`                                          | Less than or equal to comparison                                      | `PartialOrd`   |
| `=`                       | `var = expr`, `ident = type`                            | Assignment/equivalence                                                |                |
| `==`                      | `expr == expr`                                          | Equality comparison                                                   | `PartialEq`    |
| `=>`                      | `pat => expr`                                           | Part of match arm syntax                                              |                |
| `>`                       | `expr > expr`                                           | Greater than comparison                                               | `PartialOrd`   |
| `>=`                      | `expr >= expr`                                          | Greater than or equal to comparison                                   | `PartialOrd`   |
| `>>`                      | `expr >> expr`                                          | Right-shift                                                           | `Shr`          |
| `>>=`                     | `var >>= expr`                                          | Right-shift and assignment                                            | `ShrAssign`    |
| `@`                       | `ident @ pat`                                           | Pattern binding                                                       |                |
| `^`                       | `expr ^ expr`                                           | Bitwise exclusive OR                                                  | `BitXor`       |
| `^=`                      | `var ^= expr`                                           | Bitwise exclusive OR and assignment                                   | `BitXorAssign` |
| <code>&vert;</code>       | <code>pat &vert; pat</code>                             | Pattern alternatives                                                  |                |
| <code>&vert;</code>       | <code>expr &vert; expr</code>                           | Bitwise OR                                                            | `BitOr`        |
| <code>&vert;=</code>      | <code>var &vert;= expr</code>                           | Bitwise OR and assignment                                             | `BitOrAssign`  |
| <code>&vert;&vert;</code> | <code>expr &vert;&vert; expr</code>                     | Short-circuiting logical OR                                           |                |
| `?`                       | `expr?`                                                 | Error propagation                                                     |                |

### 非运算符符号

下表包含不作为运算符功能的所有符号；也就是说，它们的行为不像函数或方法调用。

表 B-2 显示了单独出现并在各种位置有效的符号。

<span class="caption">表 B-2：独立语法</span>

| Symbol                                                                 | Explanation                                                            |
| ---------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| `'ident`                                                               | Named lifetime or loop label                                           |
| Digits immediately followed by `u8`, `i32`, `f64`, `usize`, and so on  | Numeric literal of specific type                                       |
| `"..."`                                                                | String literal                                                         |
| `r"..."`, `r#"..."#`, `r##"..."##`, and so on                          | Raw string literal; escape characters not processed                    |
| `b"..."`                                                               | Byte string literal; constructs an array of bytes instead of a string  |
| `br"..."`, `br#"..."#`, `br##"..."##`, and so on                       | Raw byte string literal; combination of raw and byte string literal    |
| `'...'`                                                                | Character literal                                                      |
| `b'...'`                                                               | ASCII byte literal                                                     |
| <code>&vert;...&vert; expr</code>                                      | Closure                                                                |
| `!`                                                                    | Always-empty bottom type for diverging functions                       |
| `_`                                                                    | "Ignored" pattern binding; also used to make integer literals readable |

表 B-3 显示了在通过模块层次结构到项的路径上下文中出现的符号。

<span class="caption">表 B-3：路径相关语法</span>

| Symbol                                  | Explanation                                                                                                  |
| --------------------------------------- | -------------------------------------------------------------------------------------------------------------|
| `ident::ident`                          | Namespace path                                                                                               |
| `::path`                                | Path relative to the crate root (that is, an explicitly absolute path)                                       |
| `self::path`                            | Path relative to the current module (that is, an explicitly relative path)                                   |
| `super::path`                           | Path relative to the parent of the current module                                                            |
| `type::ident`, `<type as trait>::ident` | Associated constants, functions, and types                                                                   |
| `<type>::...`                           | Associated item for a type that cannot be directly named (for example, `<&T>::...`, `<[T]>::...`, and so on) |
| `trait::method(...)`                    | Disambiguating a method call by naming the trait that defines it                                             |
| `type::method(...)`                     | Disambiguating a method call by naming the type for which it's defined                                       |
| `<type as trait>::method(...)`          | Disambiguating a method call by naming the trait and type                                                    |

表 B-4 显示了在使用泛型类型参数的上下文中出现的符号。

<span class="caption">表 B-4：泛型</span>

| Symbol                         | Explanation                                                                                                                                         |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| `path<...>`                    | Specifies parameters to a generic type in a type (for example, `Vec<u8>`)                                                                           |
| `path::<...>`, `method::<...>` | Specifies parameters to a generic type, function, or method in an expression; often referred to as _turbofish_ (for example, `"42".parse::<i32>()`) |
| `fn ident<...> ...`            | Define generic function                                                                                                                             |
| `struct ident<...> ...`        | Define generic structure                                                                                                                            |
| `enum ident<...> ...`          | Define generic enumeration                                                                                                                          |
| `impl<...> ...`                | Define generic implementation                                                                                                                       |
| `for<...> type`                | Higher ranked lifetime bounds                                                                                                                       |
| `type<ident=type>`             | A generic type where one or more associated types have specific assignments (for example, `Iterator<Item=T>`)                                       |

表 B-5 显示了在使用 trait 限制约束泛型类型参数的上下文中出现的符号。

<span class="caption">表 B-5：Trait 限制约束</span>

| Symbol                        | Explanation                                                                                                                                |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `T: U`                        | Generic parameter `T` constrained to types that implement `U`                                                                              |
| `T: 'a`                       | Generic type `T` must outlive lifetime `'a` (meaning the type cannot transitively contain any references with lifetimes shorter than `'a`) |
| `T: 'static`                  | Generic type `T` contains no borrowed references other than `'static` ones                                                                 |
| `'b: 'a`                      | Generic lifetime `'b` must outlive lifetime `'a`                                                                                           |
| `T: ?Sized`                   | Allow generic type parameter to be a dynamically sized type                                                                                |
| `'a + trait`, `trait + trait` | Compound type constraint                                                                                                                   |

表 B-6 显示了在调用或定义宏以及在项上指定属性的上下文中出现的符号。

<span class="caption">表 B-6：宏和属性</span>

| Symbol                                      | Explanation        |
| ------------------------------------------- | ------------------ |
| `#[meta]`                                   | Outer attribute    |
| `#![meta]`                                  | Inner attribute    |
| `$ident`                                    | Macro substitution |
| `$ident:kind`                               | Macro metavariable |
| `$(...)...`                                 | Macro repetition   |
| `ident!(...)`, `ident!{...}`, `ident![...]` | Macro invocation   |

表 B-7 显示了创建注释的符号。

<span class="caption">表 B-7：注释</span>

| Symbol     | Explanation             |
| ---------- | ----------------------- |
| `//`       | Line comment            |
| `//!`      | Inner line doc comment  |
| `///`      | Outer line doc comment  |
| `/*...*/`  | Block comment           |
| `/*!...*/` | Inner block doc comment |
| `/**...*/` | Outer block doc comment |

表 B-8 显示了使用括号的上下文。

<span class="caption">表 B-8：括号</span>

| Symbol                   | Explanation                                                                                 |
| ------------------------ | ------------------------------------------------------------------------------------------- |
| `()`                     | Empty tuple (aka unit), both literal and type                                               |
| `(expr)`                 | Parenthesized expression                                                                    |
| `(expr,)`                | Single-element tuple expression                                                             |
| `(type,)`                | Single-element tuple type                                                                   |
| `(expr, ...)`            | Tuple expression                                                                            |
| `(type, ...)`            | Tuple type                                                                                  |
| `expr(expr, ...)`        | Function call expression; also used to initialize tuple `struct`s and tuple `enum` variants |

表 B-9 显示了使用花括号的上下文。

<span class="caption">表 B-9：花括号</span>

| Context      | Explanation      |
| ------------ | ---------------- |
| `{...}`      | Block expression |
| `Type {...}` | Struct literal   |

表 B-10 显示了使用方括号的上下文。

<span class="caption">表 B-10：方括号</span>

| Context                                            | Explanation                                                                                                                   |
| -------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `[...]`                                            | Array literal                                                                                                                 |
| `[expr; len]`                                      | Array literal containing `len` copies of `expr`                                                                               |
| `[type; len]`                                      | Array type containing `len` instances of `type`                                                                               |
| `expr[expr]`                                       | Collection indexing; overloadable (`Index`, `IndexMut`)                                                                       |
| `expr[..]`, `expr[a..]`, `expr[..b]`, `expr[a..b]` | Collection indexing pretending to be collection slicing, using `Range`, `RangeFrom`, `RangeTo`, or `RangeFull` as the "index" |
