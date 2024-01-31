# Targets

`rustc` 默认是一个交叉编译器。这意味着您可以使用任何编译器为任何架构进行构建。*targets* 列表是您可以为其构建的架构。有关目标的详细列表，请参阅 [Platform Support](../platform-support.md) 页面，或参阅 [Built-in Targets](built-in.md) 了解如何查看您的 `rustc` 版本可用目标的说明。

要查看可以设置的所有目标选项，请参阅文档
[here](https://doc.rust-lang.org/nightly/nightly-rustc/rustc_target/spec/struct.Target.html).

要编译到特定的「目标 target」，请使用 `--target` 标志：

```bash
$ rustc src/main.rs --target=wasm32-unknown-unknown
```
## Target Features

`x86` 和 `ARMv8` 是两种流行的CPU架构。它们的指令集形成了大多数CPU的通用基准。但是，某些CPU会通过自定义指令集来扩展这些指令集，例如向量（`AVX`）、位操作（`BMI`）或加密（`AES`）指令集。

知道其编译的代码将在哪些CPU上运行的开发人员可以通过 `-C target-feature=val` 标志选择添加（或删除）特定于CPU的指令集。

请注意，该标志通常被认为是不安全的。更多详细信息可以在 [本章](known-issues.md) 中找到。
