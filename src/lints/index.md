# Lints

在软件开发中，"lint" 是一个帮助你改善源代码的工具。
Rust 编译器中包含了一些 lints，并且它在编译你的代码时会运行这些 lints。
这些 lints 可能会产生警告，错误或者什么都没有，这些依赖于你对它的配置。

这里有一个小例子:

```bash
$ cat main.rs
fn main() {
    let x = 5;
}
$ rustc main.rs
warning: unused variable: `x`
 --> main.rs:2:9
  |
2 |     let x = 5;
  |         ^
  |
  = note: `#[warn(unused_variables)]` on by default
  = note: to avoid this warning, consider using `_x` instead
```

这是 `unused_variables` lint，并且它告诉你在代码中引入了你没有使用过的变量。
这不是一种 *wrong* (错误)，但可能是一个Bug，因此你得到了一个警告。

## Future-incompatible lints

有时，为了修复一个问题，需要更改编译器，这个问题可能会导致现有代码停止编译。
在这些情况下，会发出`Future-incompatible`的 lint，以便为 Rust 的用户提供一个平稳的过渡到新的行为。
最初，编译器将继续接受有问题的代码并发出警告。
该警告有一个问题的描述，一个说明这在未来将成为错误的通知，以及一个链接到跟踪问题的机会，提供了详细的信息和反馈的机会。
这给用户一些时间来修复代码以适应更改。经过一段时间后，警告可能会变成错误。

以下是一个 `future-incompatible` 的例子：

```text
warning: borrow of packed field is unsafe and requires unsafe function or block (error E0133)
  --> lint_example.rs:11:13
   |
11 |     let y = &x.data.0;
   |             ^^^^^^^^^
   |
   = note: `#[warn(safe_packed_borrows)]` on by default
   = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
   = note: for more information, see issue #46043 <https://github.com/rust-lang/rust/issues/46043>
   = note: fields of packed structs might be misaligned: dereferencing a misaligned pointer or even just creating a misaligned reference is undefined behavior
```

有关更多 `future-incompatible` 流程和策略的变更，请参见 [RFC 1589]

[RFC 1589]: https://github.com/rust-lang/rfcs/blob/master/text/1589-rustc-bug-fix-procedure.md
