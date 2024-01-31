# 什么是 rustc?

欢迎来到 "The rustc book"! `rustc` 是 Rust 编程语言的编译器，它由 rust 编译构建。
编译器获取源代码并生成二进制代码，作为库 或可执行文件。

大多数 Rust 程序员不会 `rustc` 直接调用，而是通过 [Cargo](../cargo/index.html) 进行调用。
不过，这一切都是为了服务 `rustc`！如果你想看看货物是如何使用 `rustc` 的，你可以

```bash
$ cargo build --verbose
```

它会打印出每个 `rustc` 调用。这本书可以帮助你理解这些选项的每个选项。
此外，虽然大多数 Rust 开发都使用 Cargo，但并非所有人都是这样：有时他们将 `rustc` 集成到其他构建系统中。
这本书应该提供指导，帮助你了解所有需要的选项。

## 基本用法

假设在你的 `hello.rs` 文件中，有一个 Hello World 程序：

```rust
fn main() {
    println!("Hello, World!");
}
```

要将此源代码转换为可执行文件，您可以使用 `rustc`：

```bash
$ rustc hello.rs
$ ./hello # on a *NIX
$ .\hello.exe # on Windows
```

请注意，我们只向 `rustc` 传递 *crate root*，而不是我们希望编译的每个文件。
例如，如果我们有一个看起来像这样的 `main.rs`：

```rust,ignore (needs-multiple-files)
mod foo;

fn main() {
    foo::hello();
}
```

和一个包含以下内容的 `foo.rs`：

```rust,no_run
pub fn hello() {
    println!("Hello, world!");
}
```

要编译这个，我们可以运行这个命令：

```bash
$ rustc main.rs
```

无需告诉 `rustc` 关于 `foo.rs` 的信息；`mod` 语句提供了它所需的一切。这与您使用 C 编译器的方式不同，您在每个文件上调用编译器，然后链接所有内容。换句话说，*crate* 是翻译单元，而不是特定的模块。
