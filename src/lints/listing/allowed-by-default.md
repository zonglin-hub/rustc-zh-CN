# Allowed-by-default Lints

默认情况下，这些 lint 都设置为 `allow` 级别。因此，除非您使用标志或属性将它们设置为更高的 lint 级别，否则它们将不会显示。

* [`absolute_paths_not_starting_with_crate`](#absolute-paths-not-starting-with-crate)
* [`box_pointers`](#box-pointers)
* [`elided_lifetimes_in_paths`](#elided-lifetimes-in-paths)
* [`explicit_outlives_requirements`](#explicit-outlives-requirements)
* [`ffi_unwind_calls`](#ffi-unwind-calls)
* [`fuzzy_provenance_casts`](#fuzzy-provenance-casts)
* [`keyword_idents`](#keyword-idents)
* [`let_underscore_drop`](#let-underscore-drop)
* [`lossy_provenance_casts`](#lossy-provenance-casts)
* [`macro_use_extern_crate`](#macro-use-extern-crate)
* [`meta_variable_misuse`](#meta-variable-misuse)
* [`missing_abi`](#missing-abi)
* [`missing_copy_implementations`](#missing-copy-implementations)
* [`missing_debug_implementations`](#missing-debug-implementations)
* [`missing_docs`](#missing-docs)
* [`multiple_supertrait_upcastable`](#multiple-supertrait-upcastable)
* [`must_not_suspend`](#must-not-suspend)
* [`non_ascii_idents`](#non-ascii-idents)
* [`non_exhaustive_omitted_patterns`](#non-exhaustive-omitted-patterns)
* [`pointer_structural_match`](#pointer-structural-match)
* [`rust_2021_incompatible_closure_captures`](#rust-2021-incompatible-closure-captures)
* [`rust_2021_incompatible_or_patterns`](#rust-2021-incompatible-or-patterns)
* [`rust_2021_prefixes_incompatible_syntax`](#rust-2021-prefixes-incompatible-syntax)
* [`rust_2021_prelude_collisions`](#rust-2021-prelude-collisions)
* [`single_use_lifetimes`](#single-use-lifetimes)
* [`trivial_casts`](#trivial-casts)
* [`trivial_numeric_casts`](#trivial-numeric-casts)
* [`unnameable_types`](#unnameable-types)
* [`unreachable_pub`](#unreachable-pub)
* [`unsafe_code`](#unsafe-code)
* [`unsafe_op_in_unsafe_fn`](#unsafe-op-in-unsafe-fn)
* [`unstable_features`](#unstable-features)
* [`unused_crate_dependencies`](#unused-crate-dependencies)
* [`unused_extern_crates`](#unused-extern-crates)
* [`unused_import_braces`](#unused-import-braces)
* [`unused_lifetimes`](#unused-lifetimes)
* [`unused_macro_rules`](#unused-macro-rules)
* [`unused_qualifications`](#unused-qualifications)
* [`unused_results`](#unused-results)
* [`unused_tuple_struct_fields`](#unused-tuple-struct-fields)
* [`variant_size_differences`](#variant-size-differences)

## absolute-paths-not-starting-with-crate

`absolute_paths_not_starting_with_crate` lint 检测以模块名而不是 `crate`、`self` 或外部 crate 名开始的完全限定路径。

### Example

```rust,edition2015,compile_fail
#![deny(absolute_paths_not_starting_with_crate)]

mod foo {
    pub fn bar() {}
}

fn main() {
    ::foo::bar();
}
```

This will produce:

```text
error: absolute paths must start with `self`, `super`, `crate`, or an external crate name in the 2018 edition
 --> lint_example.rs:8:5
  |
8 |     ::foo::bar();
  |     ^^^^^^^^^^ help: use `crate`: `crate::foo::bar`
  |
  = warning: this is accepted in the current edition (Rust 2015) but is a hard error in Rust 2018!
  = note: for more information, see issue #53130 <https://github.com/rust-lang/rust/issues/53130>
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(absolute_paths_not_starting_with_crate)]
  |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

```

### Explanation

Rust [editions] 允许在不破坏向后兼容性的情况下使语言不断发展。
此 lint 捕获使用 2015 年版风格的绝对路径的代码。
在 2015 年版中，绝对路径（以 `::` 开头的路径）指的是 crate 根目录或外部 crate。
在 2018 年版中，它被修改为只引用外部 crate。应该使用路径前缀 `crate::` 来引用 crate 根目录中的项。

如果您将编译器从 2015 版切换到 2018 版而没有更新代码，那么如果使用旧样式路径，将无法编译。
您可以手动更改路径以使用 `crate::` 前缀来过渡到 2018 版。

此 lint 可以自动解决此问题。默认情况下它被 `allow`，因为这段代码在 2015 年版中是完全有效的。
使用带有 `--edition` 标志的 [`cargo fix`] 工具将此 lint 切换为“警告”，并自动应用编译器建议的修复。
这为将旧代码更新到 2018 年版提供了完全自动化的方法。

[editions]: https://doc.rust-lang.org/edition-guide/
[`cargo fix`]: https://doc.rust-lang.org/cargo/commands/cargo-fix.html

## box-pointers

`box_pointers` lints 限制了 Box 类型使用

### Example

```rust,compile_fail
#![deny(box_pointers)]
struct Foo {
    x: Box<isize>,
}
```

This will produce:

```text
error: type uses owned (Box type) pointers: Box<isize>
 --> lint_example.rs:4:5
  |
4 |     x: Box<isize>,
  |     ^^^^^^^^^^^^^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(box_pointers)]
  |         ^^^^^^^^^^^^

```

### Explanation

这个 lint 主要是出于历史原因，并不是特别有用。`Box<T>` 曾经是语言内置的，是进行堆分配的唯一方式。今天的 Rust 可以调用其他分配器等。

## elided-lifetimes-in-paths

`elided_lifetimes_in_paths` lint 用于检测隐藏生命周期参数。

### Example

```rust,compile_fail
#![deny(elided_lifetimes_in_paths)]
#![deny(warnings)]
struct Foo<'a> {
    x: &'a u32
}

fn foo(x: &Foo) {
}
```

This will produce:

```text
error: hidden lifetime parameters in types are deprecated
 --> lint_example.rs:8:12
  |
8 | fn foo(x: &Foo) {
  |            ^^^ expected lifetime parameter
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(elided_lifetimes_in_paths)]
  |         ^^^^^^^^^^^^^^^^^^^^^^^^^
help: indicate the anonymous lifetime
  |
8 | fn foo(x: &Foo<'_>) {
  |               ++++

```

### Explanation

省略了生命周期参数会使人们难以一眼看出正在发生的借用。此 lint 确保始终明确声明生命周期参数，即使它是 `'_` [placeholder lifetime]。

此 lint 默认情况下为 `allow`，因为它存在一些已知问题，并且可能需要针对旧代码进行重大更改。

[placeholder lifetime]: https://doc.rust-lang.org/reference/lifetime-elision.html#lifetime-elision-in-functions

## explicit-outlives-requirements

`explicit_outlives_requirements` lint 检测指出不必要的生命周期约束（bounds）

### Example

```rust,compile_fail
# #![allow(unused)]
#![deny(explicit_outlives_requirements)]
#![deny(warnings)]

struct SharedRef<'a, T>
where
    T: 'a,
{
    data: &'a T,
}
```

This will produce:

```text
error: outlives requirements can be inferred
 --> lint_example.rs:6:24
  |
6 |   struct SharedRef<'a, T>
  |  ________________________^
7 | | where
8 | |     T: 'a,
  | |__________^ help: remove this bound
  |
note: the lint level is defined here
 --> lint_example.rs:2:9
  |
2 | #![deny(explicit_outlives_requirements)]
  |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

```

### Explanation

如果一个 `struct` 包含一个引用，例如 `&'a T`，编译器要求 `T` 的生命周期必须比 `'a` 长。
历史上，这要求编写一个显式的生命周期约束来表示此要求。但是，这可能过于明确，导致代码混乱和不必要的复杂性。
语言被修改为在未指定的情况下自动推断边界。
具体来说，如果结构体直接或间接地包含一个具有生命周期 `'x` 的 `T` 的引用，那么它将推断 `T: 'x` 是要求。

此 lint 默认情况下为 "allow"，因为对于已经满足这些要求的现有代码来说，它可能会很嘈杂。
这是一种风格选择，因为显式声明边界仍然是有效的。它还有一些可能造成混淆的误报。

参见 [RFC 2093] 了解更多。

[RFC 2093]: https://github.com/rust-lang/rfcs/blob/master/text/2093-infer-outlives.md

## ffi-unwind-calls

`ffi_unwind_calls` lint 检测使用 `C-unwind` 或其他 FFI-unwind ABI 调用外部函数或函数指针的情况。

### Example

```rust
#![warn(ffi_unwind_calls)]

extern "C-unwind" {
    fn foo();
}

fn bar() {
    unsafe { foo(); }
    let ptr: unsafe extern "C-unwind" fn() = foo;
    unsafe { ptr(); }
}
```

This will produce:

```text
warning: call to foreign function with FFI-unwind ABI
 --> lint_example.rs:9:14
  |
9 |     unsafe { foo(); }
  |              ^^^^^ call to foreign function with FFI-unwind ABI
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![warn(ffi_unwind_calls)]
  |         ^^^^^^^^^^^^^^^^


warning: call to function pointer with FFI-unwind ABI
  --> lint_example.rs:11:14
   |
11 |     unsafe { ptr(); }
   |              ^^^^^ call to function pointer with FFI-unwind ABI

```

### Explanation

对于包含此类调用的 crate，如果它们使用 `-C panic=unwind` 编译，则产生的库无法与使用 `-C panic=abort` 编译的 crate 链接。
因此，对于需要这种能力的 crate，必须避免此类调用。

## fuzzy-provenance-casts

`fuzzy_provenance_casts` lint 检测整数和指针之间的 `as` 转换。

### Example

```rust
#![feature(strict_provenance)]
#![warn(fuzzy_provenance_casts)]

fn main() {
    let _dangling = 16_usize as *const u8;
}
```

This will produce:

```text
warning: strict provenance disallows casting integer `usize` to pointer `*const u8`
 --> lint_example.rs:5:21
  |
5 |     let _dangling = 16_usize as *const u8;
  |                     ^^^^^^^^^^^^^^^^^^^^^
  |
  = help: if you can't comply with strict provenance and don't have a pointer with the correct provenance you can use `std::ptr::from_exposed_addr()` instead
note: the lint level is defined here
 --> lint_example.rs:2:9
  |
2 | #![warn(fuzzy_provenance_casts)]
  |         ^^^^^^^^^^^^^^^^^^^^^^
help: use `.with_addr()` to adjust a valid pointer in the same allocation, to this address
  |
5 |     let _dangling = (...).with_addr(16_usize);
  |                     ++++++++++++++++        ~

```

### Explanation

该 lint 是严格 provenance 工作的一部分，请参考 [issue #95228]。
将整数转换为指针被认为是不良风格，因为指针除了包含地址外还包含 provenance，表明指针可以读取/写入的内存。
将没有 provenance 的整数转换为指针需要编译器进行分配（猜测）provenance。
编译器分配“所有暴露的有效”（有关更多信息，请参阅 [`ptr::from_exposed_addr`] 的文档）。
这不利于优化器，也不适合 动态分析/动态程序验证（例如 Miri 或 CHERI 平台）。

使用 [`ptr::with_addr`] 而不是指定您想要的 provenance 要好得多。
如果因为代码依赖于暴露出的 provenance 而无法使用此函数，那么可以使用 [`ptr::from_exposed_addr`] 作为替代方案。

[issue #95228]: https://github.com/rust-lang/rust/issues/95228
[`ptr::with_addr`]: https://doc.rust-lang.org/core/ptr/fn.with_addr
[`ptr::from_exposed_addr`]: https://doc.rust-lang.org/core/ptr/fn.from_exposed_addr

## keyword-idents

`keyword-idents` lint 检测被用作标识符的版本关键字。

### Example

```rust,edition2015,compile_fail
#![deny(keyword_idents)]
// edition 2015
fn dyn() {}
```

This will produce:

```text
error: `dyn` is a keyword in the 2018 edition
 --> lint_example.rs:4:4
  |
4 | fn dyn() {}
  |    ^^^ help: you can use a raw identifier to stay compatible: `r#dyn`
  |
  = warning: this is accepted in the current edition (Rust 2015) but is a hard error in Rust 2018!
  = note: for more information, see issue #49716 <https://github.com/rust-lang/rust/issues/49716>
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(keyword_idents)]
  |         ^^^^^^^^^^^^^^

```

### Explanation

Rust [editions] 允许语言向前发展而不破坏其向后兼容性。
此 lint 捕获代码中的被用作标识符（例如变量名、函数名等等）的新增关键。
如果你没有更新代码就切换编译器到一个新语义版本，就会在你将新关键字作为标识符的情况下编译失败。

可以手动将标识符改为非关键字，或者使用 [raw identifier]，例如`r#dyn`，来过渡到新版本。

该 lint 可自动解决该问题，其默认为 “allow”等级，因为该代码在旧版本中完全有效。
[`cargo fix`] 工具自带的 `--edition` 标签会将此 lint 的等级切换为 “warn” ，并自动应用编译器建议的修复（即使用原始标识符）。
这提供了一种完全自动化的方法来将旧代码更新到新版本。

[editions]: https://doc.rust-lang.org/edition-guide/
[raw identifier]: https://doc.rust-lang.org/reference/identifiers.html
[`cargo fix`]: https://doc.rust-lang.org/cargo/commands/cargo-fix.html

## let-underscore-drop

`let_underscore_drop` lint 检查不将具有非平凡 Drop 实现的表达式绑定到任何东西的语句，这会导致表达式立即被丢弃，而不是在作用域结束时被丢弃。

### Example

```rust
struct SomeStruct;
impl Drop for SomeStruct {
    fn drop(&mut self) {
        println!("Dropping SomeStruct");
    }
}

fn main() {
   #[warn(let_underscore_drop)]
    // SomeStruct is dropped immediately instead of at end of scope,
    // so "Dropping SomeStruct" is printed before "end of main".
    // The order of prints would be reversed if SomeStruct was bound to
    // a name (such as "_foo").
    let _ = SomeStruct;
    println!("end of main");
}
```

This will produce:

```text
warning: non-binding let on a type that implements `Drop`
  --> lint_example.rs:14:5
   |
14 |     let _ = SomeStruct;
   |     ^^^^^^^^^^^^^^^^^^^
   |
note: the lint level is defined here
  --> lint_example.rs:9:11
   |
9  |    #[warn(let_underscore_drop)]
   |           ^^^^^^^^^^^^^^^^^^^
help: consider binding to an unused variable to avoid immediately dropping the value
   |
14 |     let _unused = SomeStruct;
   |         ~~~~~~~
help: consider immediately dropping the value
   |
14 |     drop(SomeStruct);
   |     ~~~~~          +

```

### Explanation

将表达式分配给下划线会导致表达式立即丢弃，而不是将表达式的生命周期延长到作用域的末尾。
这通常是意料之外的，尤其是对于像 `MutexGuard` 这样的类型，它们通常被用来在整个作用域期间锁定互斥锁。

如果你想将表达式的生命周期延长到作用域的末尾，将下划线前缀的名称（例如 `_foo`）分配给表达式。
如果你确实想立即丢弃表达式，那么对表达式调用 `std::mem::drop` 是更清晰且有助于传达意图的做法。

## lossy-provenance-casts

`lossy_provenance_cast` 检查指针和整数之间的转换。

### Example

```rust
#![feature(strict_provenance)]
#![warn(lossy_provenance_casts)]

fn main() {
    let x: u8 = 37;
    let _addr: usize = &x as *const u8 as usize;
}
```

This will produce:

```text
warning: under strict provenance it is considered bad style to cast pointer `*const u8` to integer `usize`
 --> lint_example.rs:6:24
  |
6 |     let _addr: usize = &x as *const u8 as usize;
  |                        ^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = help: if you can't comply with strict provenance and need to expose the pointer provenance you can use `.expose_addr()` instead
note: the lint level is defined here
 --> lint_example.rs:2:9
  |
2 | #![warn(lossy_provenance_casts)]
  |         ^^^^^^^^^^^^^^^^^^^^^^
help: use `.addr()` to obtain the address of a pointer
  |
6 |     let _addr: usize = (&x as *const u8).addr();
  |                        +               ~~~~~~~~

```

### Explanation

这个 lint 是「严格的出处 strict_provenance」管理的一部分，详见 [issue #95228]。将指针转换为整数是有损失的，因为除了*地址*之外，指针还可能与特定的 *出处* 相关联。此信息由优化器用于动态分析/动态程序验证（例如 Miri 或 CHERI 平台）。

由于此转换是有损的，因此建议使用 [`ptr::addr`] 方法，该方法具有类似的效果，但不会 “暴露” 指针的出处。这提高了优化的可能性。有关暴露指针出处的更多信息，请参阅 [`ptr::addr`] 和 [`ptr::expose_addr`] 的文档。

如果你的代码无法遵守严格的出处规定并且需要暴露出处，那么可以使用 [`ptr::expose_addr`] 作为逃生舱，它保留了 `as usize` 转换的行为，同时明确了语义。

[issue #95228]: https://github.com/rust-lang/rust/issues/95228
[`ptr::addr`]: https://doc.rust-lang.org/core/ptr/fn.addr
[`ptr::expose_addr`]: https://doc.rust-lang.org/core/ptr/fn.expose_addr

## macro-use-extern-crate

`macro_use_extern_crate` lint 用于检测 [`macro_use` attribute] 属性的使用

### Example

```rust,ignore (needs extern crate)
#![deny(macro_use_extern_crate)]

#[macro_use]
extern crate serde_json;

fn main() {
    let _ = json!{{}};
}
```

This will produce:

```text
error: deprecated `#[macro_use]` attribute used to import macros should be replaced at use sites with a `use` item to import the macro instead
 --> src/main.rs:3:1
  |
3 | #[macro_use]
  | ^^^^^^^^^^^^
  |
note: the lint level is defined here
 --> src/main.rs:1:9
  |
1 | #![deny(macro_use_extern_crate)]
  |         ^^^^^^^^^^^^^^^^^^^^^^
```

### Explanation

[`macro_use`] 属性放在 [`extern crate`] 项上使其宏可被使用，而这个外部 crate 可能会被放进该 crate 的路径前缀，导致导入宏在作用域内无处不在。在 [2018 版本][2018 edition] 中致力于简化依赖项的处理，`extern crate` 的使用已经淘汰了。要将宏从外部 crate 导入作用域，建议使用 [`use` 导入][`use` import]。

这个 lint 程序的默认值是 "allow"，因为这是一个风格选择。这个问题还没有解决，请参阅[issue #52043]了解更多信息。

[`macro_use` attribute]: https://doc.rust-lang.org/reference/macros-by-example.html#the-macro_use-attribute
[`use` import]: https://doc.rust-lang.org/reference/items/use-declarations.html
[issue #52043]: https://github.com/rust-lang/rust/issues/52043

## meta-variable-misuse

`meta_variable_misuse` lint 检测宏定义中可能存在的元变量滥用。

### Example

```rust,compile_fail
#![deny(meta_variable_misuse)]

macro_rules! foo {
    () => {};
    ($( $i:ident = $($j:ident),+ );*) => { $( $( $i = $k; )+ )* };
}

fn main() {
    foo!();
}
```

This will produce:

```text
error: unknown macro variable `k`
 --> lint_example.rs:5:55
  |
5 |     ($( $i:ident = $($j:ident),+ );*) => { $( $( $i = $k; )+ )* };
  |                                                       ^^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(meta_variable_misuse)]
  |         ^^^^^^^^^^^^^^^^^^^^

```

### Explanation

[`macro_rules`] 宏有许多不恰当的定义方式，这些错误以前只有在宏被展开或根本不展开时才能才能检测得到。
当宏被 *定义* 时，这个 lint 试图捕获一些问题。

这个 lint 默认等级是 “allow” 的，因为其有误报或其他问题。更多细节请参阅 [issue #61053]

[`macro_rules`]: https://doc.rust-lang.org/reference/macros-by-example.html
[issue #61053]: https://github.com/rust-lang/rust/issues/61053

## missing-abi

`missing_abi`检测在外部声明中 ABI 被遗漏的情况。

### Example

```rust,compile_fail
#![deny(missing_abi)]

extern fn foo() {}
```

This will produce:

```text
error: extern declarations without an explicit ABI are deprecated
 --> lint_example.rs:4:1
  |
4 | extern fn foo() {}
  | ^^^^^^^^^^^^^^^ ABI should be specified here
  |
  = help: the default ABI is C
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(missing_abi)]
  |         ^^^^^^^^^^^

```

### Explanation

历史上，Rust 隐式地选择 C 作为 extern 的 ABI 声明。我们希望在未来添加新的 abi，比如 `C-unwind`，虽然这还没有发生，他们的加入可以使 ABI 将使代码评审变得更容易。

## missing-copy-implementations

`missing_copy_implementations` lint 用于检测可能为公共类型遗忘的 [`Copy`] 实现的情况。

[`Copy`]: https://doc.rust-lang.org/std/marker/trait.Copy.html

### Example

```rust,compile_fail
#![deny(missing_copy_implementations)]
pub struct Foo {
    pub field: i32
}
# fn main() {}
```

This will produce:

```text
error: type could implement `Copy`; consider adding `impl Copy`
 --> lint_example.rs:2:1
  |
2 | / pub struct Foo {
3 | |     pub field: i32
4 | | }
  | |_^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(missing_copy_implementations)]
  |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^

```

### Explanation

在历史上（1.0 版本之前），如果可能，类型会被自动标记为 `Copy`。后来这一行为发生了改变，需要明确选择加入 `Copy` trait 来实现。作为这一改变的一部分，增加了一个 lint，用于提醒可复制的类型未被标记为 Copy。

这个 lint 默认设置为 “allow”，是因为这样的代码并不糟糕；通常，人们会特意编写这样的新类型，以使得一个 `Copy` 类型不再具有 `Copy` 特性。`Copy` 类型可能导致大数据的无意复制，从而影响性能。

## missing-debug-implementations

`missing_debug_implementations` lint 检测公共类型 [`fmt::Debug`] 的缺失实现。

[`fmt::Debug`]: https://doc.rust-lang.org/std/fmt/trait.Debug.html

### Example

```rust,compile_fail
#![deny(missing_debug_implementations)]
pub struct Foo;
# fn main() {}
```

This will produce:

```text
error: type does not implement `Debug`; consider adding `#[derive(Debug)]` or a manual implementation
 --> lint_example.rs:2:1
  |
2 | pub struct Foo;
  | ^^^^^^^^^^^^^^^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(missing_debug_implementations)]
  |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

```

### Explanation

在所有类型上实现 `Debug` 有助于调试。因为它提供了个格式化和显示值的便捷方式。使用 `#[derive(Debug)]` 属性会自动生成一个典型实现，或者手动实现该 `Debug`「特质 trait」 添加自定义实现。

该 lint 默认等级为 “allow” ，因为对所有类型添加 `Debug` 都可能会对编译时长和代码体积产生负面作用。它还要求对每种类型都添加样板，这有时会是种（编码上的）阻碍。

## missing-docs

`missing_docs` lint 检测公共项目的缺失文档。

### Example

```rust,compile_fail
#![deny(missing_docs)]
pub fn foo() {}
```

This will produce:

```text
error: missing documentation for the crate
 --> lint_example.rs:1:1
  |
1 | / #![deny(missing_docs)]
2 | | fn main() {
3 | | pub fn foo() {}
4 | | }
  | |_^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(missing_docs)]
  |         ^^^^^^^^^^^^

```

### Explanation

该 lint 旨在确保一个库有良好的文档记录。没有文档的项目对于用户来说很难理解如何正确使用。

该 lint 默认等级是 “allow” 是因为其可能会造成干扰，并不是所有项目都需要强制将一切用文档记录。

## multiple-supertrait-upcastable

`multiple_supertrait_upcastable` lint 用于检测在一个对象安全的 `trait` 是否有多个 `supertraits`。

### Example

```rust
#![feature(multiple_supertrait_upcastable)]
trait A {}
trait B {}

#[warn(multiple_supertrait_upcastable)]
trait C: A + B {}
```

This will produce:

```text
warning: `C` is object-safe and has multiple supertraits
 --> lint_example.rs:7:1
  |
7 | trait C: A + B {}
  | ^^^^^^^^^^^^^^
  |
note: the lint level is defined here
 --> lint_example.rs:6:8
  |
6 | #[warn(multiple_supertrait_upcastable)]
  |        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

```

### Explanation

为了支持具有多个 `supertraits` 的向上转型，我们需要存储多个虚函数表（vtable），这可能导致额外的空间开销，即使实际上没有代码使用向上转型。这个 lint 允许用户识别何时出现此类情况，并决定额外的开销是否合理。

## must-not-suspend

`must_not_suspend` lint 用于防止那些不应跨越挂起点 `.await` 持有的值。

### Example

```rust
#![feature(must_not_suspend)]
#![warn(must_not_suspend)]

#[must_not_suspend]
struct SyncThing {}

async fn yield_now() {}

pub async fn uhoh() {
    let guard = SyncThing {};
    yield_now().await;
    let _guard = guard;
}
```

This will produce:

```text
warning: `SyncThing` held across a suspend point, but should not be
  --> lint_example.rs:11:9
   |
11 |     let guard = SyncThing {};
   |         ^^^^^
12 |     yield_now().await;
   |                 ----- the value is held across this suspend point
   |
help: consider using a block (`{ ... }`) to shrink the value's scope, ending before the suspend point
  --> lint_example.rs:11:9
   |
11 |     let guard = SyncThing {};
   |         ^^^^^
note: the lint level is defined here
  --> lint_example.rs:2:9
   |
2  | #![warn(must_not_suspend)]
   |         ^^^^^^^^^^^^^^^^

```

### Explanation

`must_not_suspend` lint 用于检测被标记为 `#[must_not_suspend]` 属性的值在挂起点之间被持有。挂起点通常是异步函数中的 `.await`。

这个属性可用于标记在挂起时语义上不正确的值（如某些类型的计时器）、有异步替代方案的值，以及经常导致异步函数返回的 `future` 的 `Send` 性问题的值（如`MutexGuard`）。


## non-ascii-idents

`non_ascii_idents` lint 检测非 ASCII 标识符。

### Example

```rust,compile_fail
# #![allow(unused)]
#![deny(non_ascii_idents)]
fn main() {non
    let föö = 1;
}
```

This will produce:

```text
error: identifier contains non-ASCII characters
 --> lint_example.rs:4:9
  |
4 |     let föö = 1;
  |         ^^^
  |
note: the lint level is defined here
 --> lint_example.rs:2:9
  |
2 | #![deny(non_ascii_idents)]
  |         ^^^^^^^^^^^^^^^^

```

### Explanation

该 lint 允许那些只希望使用 ASCII 字符的项目将此 lint 设置为 “禁止”（例如，为了便于协作或出于安全原因）。更多详细信息请参见 [RFC 2457]。

[RFC 2457]: https://github.com/rust-lang/rfcs/blob/master/text/2457-non-ascii-idents.md

## non-exhaustive-omitted-patterns

The `non_exhaustive_omitted_patterns` lint aims to help consumers of a `#[non_exhaustive]`
struct or enum who want to match all of its fields/variants explicitly.

The `#[non_exhaustive]` annotation forces matches to use wildcards, so exhaustiveness
checking cannot be used to ensure that all fields/variants are matched explicitly. To remedy
this, this allow-by-default lint warns the user when a match mentions some but not all of
the fields/variants of a `#[non_exhaustive]` struct or enum.

### Example

```rust,ignore (needs separate crate)
// crate A
#[non_exhaustive]
pub enum Bar {
    A,
    B, // added variant in non breaking change
}

// in crate B
#![feature(non_exhaustive_omitted_patterns_lint)]
#[warn(non_exhaustive_omitted_patterns)]
match Bar::A {
    Bar::A => {},
    _ => {},
}
```

This will produce:

```text
warning: some variants are not matched explicitly
   --> $DIR/reachable-patterns.rs:70:9
   |
LL |         match Bar::A {
   |               ^ pattern `Bar::B` not covered
   |
 note: the lint level is defined here
  --> $DIR/reachable-patterns.rs:69:16
   |
LL |         #[warn(non_exhaustive_omitted_patterns)]
   |                ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   = help: ensure that all variants are matched explicitly by adding the suggested match arms
   = note: the matched value is of type `Bar` and the `non_exhaustive_omitted_patterns` attribute was found
```

Warning: setting this to `deny` will make upstream non-breaking changes (adding fields or
variants to a `#[non_exhaustive]` struct or enum) break your crate. This goes against
expected semver behavior.

### Explanation

Structs and enums tagged with `#[non_exhaustive]` force the user to add a (potentially
redundant) wildcard when pattern-matching, to allow for future addition of fields or
variants. The `non_exhaustive_omitted_patterns` lint detects when such a wildcard happens to
actually catch some fields/variants. In other words, when the match without the wildcard
would not be exhaustive. This lets the user be informed if new fields/variants were added.

## pointer-structural-match

`pointer_structural_match` lint 用于检测在模式中使用指针的情况，这些指针的行为在不同的编译器版本和优化级别上都无法依赖

### Example

```rust,compile_fail
#![deny(pointer_structural_match)]
fn foo(a: usize, b: usize) -> usize { a + b }
const FOO: fn(usize, usize) -> usize = foo;
fn main() {
    match FOO {
        FOO => {},
        _ => {},
    }
}
```

This will produce:

```text
error: function pointers and unsized pointers in patterns behave unpredictably and should not be relied upon. See https://github.com/rust-lang/rust/issues/70861 for details.
 --> lint_example.rs:6:9
  |
6 |         FOO => {},
  |         ^^^
  |
  = warning: this was previously accepted by the compiler but is being phased out; it will become a hard error in a future release!
  = note: for more information, see issue #62411 <https://github.com/rust-lang/rust/issues/70861>
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(pointer_structural_match)]
  |         ^^^^^^^^^^^^^^^^^^^^^^^^

```

### Explanation

早期 Rust 版本允许在模式中使用函数指针和泛（wide）原始指针。尽管许多情况下可以按用户期望的方式运行，但由于编译器进行优化，在运行时，指针可能已经 “不等于自身” 或者是指向不同函数的函数指针相等。这是因为如果函数体相等， LLVM 会优化掉重复函数（译者注：即保留一个），因此也会使得这些指向他们的函数指针指向同一位置。另外，如果重复的函数在不同 crate 中，且又没有通过 LTO 进行优化（删除相同代码数据），那么就会造成重复。

## rust-2021-incompatible-closure-captures

The `rust_2021_incompatible_closure_captures` lint detects variables that aren't completely
captured in Rust 2021, such that the `Drop` order of their fields may differ between
Rust 2018 and 2021.

It can also detect when a variable implements a trait like `Send`, but one of its fields does not,
and the field is captured by a closure and used with the assumption that said field implements
the same trait as the root variable.

### Example of drop reorder

```rust,edition2018,compile_fail
#![deny(rust_2021_incompatible_closure_captures)]
# #![allow(unused)]

struct FancyInteger(i32);

impl Drop for FancyInteger {
    fn drop(&mut self) {
        println!("Just dropped {}", self.0);
    }
}

struct Point { x: FancyInteger, y: FancyInteger }

fn main() {
  let p = Point { x: FancyInteger(10), y: FancyInteger(20) };

  let c = || {
     let x = p.x;
  };

  c();

  // ... More code ...
}
```

This will produce:

```text
error: changes to closure capture in Rust 2021 will affect drop order
  --> lint_example.rs:17:11
   |
17 |   let c = || {
   |           ^^
18 |      let x = p.x;
   |              --- in Rust 2018, this closure captures all of `p`, but in Rust 2021, it will only capture `p.x`
...
24 | }
   | - in Rust 2018, `p` is dropped here, but in Rust 2021, only `p.x` will be dropped here as part of the closure
   |
   = note: for more information, see <https://doc.rust-lang.org/nightly/edition-guide/rust-2021/disjoint-capture-in-closures.html>
note: the lint level is defined here
  --> lint_example.rs:1:9
   |
1  | #![deny(rust_2021_incompatible_closure_captures)]
   |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
help: add a dummy let to cause `p` to be fully captured
   |
17 ~   let c = || {
18 +      let _ = &p;
   |

```

### Explanation

In the above example, `p.y` will be dropped at the end of `f` instead of
with `c` in Rust 2021.

### Example of auto-trait

```rust,edition2018,compile_fail
#![deny(rust_2021_incompatible_closure_captures)]
use std::thread;

struct Pointer(*mut i32);
unsafe impl Send for Pointer {}

fn main() {
    let mut f = 10;
    let fptr = Pointer(&mut f as *mut i32);
    thread::spawn(move || unsafe {
        *fptr.0 = 20;
    });
}
```

This will produce:

```text
error: changes to closure capture in Rust 2021 will affect which traits the closure implements
  --> lint_example.rs:10:19
   |
10 |     thread::spawn(move || unsafe {
   |                   ^^^^^^^ in Rust 2018, this closure implements `Send` as `fptr` implements `Send`, but in Rust 2021, this closure will no longer implement `Send` because `fptr` is not fully captured and `fptr.0` does not implement `Send`
11 |         *fptr.0 = 20;
   |         ------- in Rust 2018, this closure captures all of `fptr`, but in Rust 2021, it will only capture `fptr.0`
   |
   = note: for more information, see <https://doc.rust-lang.org/nightly/edition-guide/rust-2021/disjoint-capture-in-closures.html>
note: the lint level is defined here
  --> lint_example.rs:1:9
   |
1  | #![deny(rust_2021_incompatible_closure_captures)]
   |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
help: add a dummy let to cause `fptr` to be fully captured
   |
10 ~     thread::spawn(move || { let _ = &fptr; unsafe {
11 |         *fptr.0 = 20;
12 ~     } });
   |

```

### Explanation

In the above example, only `fptr.0` is captured in Rust 2021.
The field is of type `*mut i32`, which doesn't implement `Send`,
making the code invalid as the field cannot be sent between threads safely.

## rust-2021-incompatible-or-patterns

The `rust_2021_incompatible_or_patterns` lint detects usage of old versions of or-patterns.

### Example

```rust,compile_fail
#![deny(rust_2021_incompatible_or_patterns)]

macro_rules! match_any {
    ( $expr:expr , $( $( $pat:pat )|+ => $expr_arm:expr ),+ ) => {
        match $expr {
            $(
                $( $pat => $expr_arm, )+
            )+
        }
    };
}

fn main() {
    let result: Result<i64, i32> = Err(42);
    let int: i64 = match_any!(result, Ok(i) | Err(i) => i.into());
    assert_eq!(int, 42);
}
```

This will produce:

```text
error: the meaning of the `pat` fragment specifier is changing in Rust 2021, which may affect this macro
 --> lint_example.rs:4:26
  |
4 |     ( $expr:expr , $( $( $pat:pat )|+ => $expr_arm:expr ),+ ) => {
  |                          ^^^^^^^^ help: use pat_param to preserve semantics: `$pat:pat_param`
  |
  = warning: this is accepted in the current edition (Rust 2018) but is a hard error in Rust 2021!
  = note: for more information, see <https://doc.rust-lang.org/nightly/edition-guide/rust-2021/or-patterns-macro-rules.html>
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(rust_2021_incompatible_or_patterns)]
  |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

```

### Explanation

In Rust 2021, the `pat` matcher will match additional patterns, which include the `|` character.

## rust-2021-prefixes-incompatible-syntax

The `rust_2021_prefixes_incompatible_syntax` lint detects identifiers that will be parsed as a
prefix instead in Rust 2021.

### Example

```rust,edition2018,compile_fail
#![deny(rust_2021_prefixes_incompatible_syntax)]

macro_rules! m {
    (z $x:expr) => ();
}

m!(z"hey");
```

This will produce:

```text
error: prefix `z` is unknown
 --> lint_example.rs:8:4
  |
8 | m!(z"hey");
  |    ^ unknown prefix
  |
  = warning: this is accepted in the current edition (Rust 2018) but is a hard error in Rust 2021!
  = note: for more information, see <https://doc.rust-lang.org/nightly/edition-guide/rust-2021/reserving-syntax.html>
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(rust_2021_prefixes_incompatible_syntax)]
  |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
help: insert whitespace here to avoid this being parsed as a prefix in Rust 2021
  |
8 | m!(z "hey");
  |     +

```

### Explanation

In Rust 2015 and 2018, `z"hey"` is two tokens: the identifier `z`
followed by the string literal `"hey"`. In Rust 2021, the `z` is
considered a prefix for `"hey"`.

This lint suggests to add whitespace between the `z` and `"hey"` tokens
to keep them separated in Rust 2021.

## rust-2021-prelude-collisions

The `rust_2021_prelude_collisions` lint detects the usage of trait methods which are ambiguous
with traits added to the prelude in future editions.

### Example

```rust,compile_fail
#![deny(rust_2021_prelude_collisions)]

trait Foo {
    fn try_into(self) -> Result<String, !>;
}

impl Foo for &str {
    fn try_into(self) -> Result<String, !> {
        Ok(String::from(self))
    }
}

fn main() {
    let x: String = "3".try_into().unwrap();
    //                  ^^^^^^^^
    // This call to try_into matches both Foo::try_into and TryInto::try_into as
    // `TryInto` has been added to the Rust prelude in 2021 edition.
    println!("{x}");
}
```

This will produce:

```text
error: trait method `try_into` will become ambiguous in Rust 2021
  --> lint_example.rs:14:21
   |
14 |     let x: String = "3".try_into().unwrap();
   |                     ^^^^^^^^^^^^^^ help: disambiguate the associated function: `Foo::try_into(&*"3")`
   |
   = warning: this is accepted in the current edition (Rust 2018) but is a hard error in Rust 2021!
   = note: for more information, see <https://doc.rust-lang.org/nightly/edition-guide/rust-2021/prelude.html>
note: the lint level is defined here
  --> lint_example.rs:1:9
   |
1  | #![deny(rust_2021_prelude_collisions)]
   |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^

```

### Explanation

In Rust 2021, one of the important introductions is the [prelude changes], which add
`TryFrom`, `TryInto`, and `FromIterator` into the standard library's prelude. Since this
results in an ambiguity as to which method/function to call when an existing `try_into`
method is called via dot-call syntax or a `try_from`/`from_iter` associated function
is called directly on a type.

[prelude changes]: https://blog.rust-lang.org/inside-rust/2021/03/04/planning-rust-2021.html#prelude-changes

## single-use-lifetimes

`single_use_lifetimes` lint 检测只使用一次的生命期。

### Example

```rust,compile_fail
#![deny(single_use_lifetimes)]

fn foo<'a>(x: &'a u32) {}
```

This will produce:

```text
error: lifetime parameter `'a` only used once
 --> lint_example.rs:4:8
  |
4 | fn foo<'a>(x: &'a u32) {}
  |        ^^      -- ...is used only here
  |        |
  |        this lifetime...
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(single_use_lifetimes)]
  |         ^^^^^^^^^^^^^^^^^^^^
help: elide the single-use lifetime
  |
4 - fn foo<'a>(x: &'a u32) {}
4 + fn foo(x: &u32) {}
  |

```

### Explanation

显式指定一个生命周期，例如在函数或 `impl` 中的 `'a`应该用来链接这两者。否则，应该使用 `'_` 表明生命周期并未链接到两者，或者如果有可能的话干脆直接省略生命周期。

该 lint 默认等级为 “allow” ，因为它是在 `'_` 和省略生命周期第一次被引入的时候引入的，而且这个 lint 可能会有很多干扰（too noisy）。 此外，它还会产生一些已知的误报，了解历史背景请参阅 [RFC 2115]，更多细节请参阅 [issue #44752]。

[RFC 2115]: https://github.com/rust-lang/rfcs/blob/master/text/2115-argument-lifetimes.md
[issue #44752]: https://github.com/rust-lang/rust/issues/44752

## trivial-casts

`trivial_casts` lint 检测可以被强制类型转换替代的平凡类型转换，这可能需要一个临时变量。

### Example

```rust,compile_fail
#![deny(trivial_casts)]
let x: &u32 = &42;
let y = x as *const u32;
```

This will produce:

```text
error: trivial cast: `&u32` as `*const u32`
 --> lint_example.rs:4:9
  |
4 | let y = x as *const u32;
  |         ^^^^^^^^^^^^^^^
  |
  = help: cast can be replaced by coercion; this might require a temporary variable
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(trivial_casts)]
  |         ^^^^^^^^^^^^^

```

### Explanation

一个简单的类型转换是将`e`转换为` T`，其中`e`的类型是`U`，`U`是
`T`的子类型。这种类型转换通常是不必要的，其通常可以被推断出来。

这个lint程序默认是"allow"，因为在某些情况下，例如在FFI接口或复杂类型别名中触发不正确，或者在更困难的情况下明确表达意图。
这可能会成为一个警告，可能会使用强制语法提供了一种方便的方法来解决当前的问题。
参见[RFC 401(强制类型)][RFC -401]，[RFC 803(类型归属)][RFC -803]和 [RFC 3307(删除类型归属)][RFC -3307]用于历史上下文。

[rfc-401]: https://github.com/rust-lang/rfcs/blob/master/text/0401-coercions.md
[rfc-803]: https://github.com/rust-lang/rfcs/blob/master/text/0803-type-ascription.md
[rfc-3307]: https://github.com/rust-lang/rfcs/blob/master/text/3307-de-rfc-type-ascription.md

## trivial-numeric-casts

`trivial_numeric_casts` lint 用于检测可以移除的琐碎的数值类型转换。

### Example

```rust,compile_fail
#![deny(trivial_numeric_casts)]
let x = 42_i32 as i32;
```

This will produce:

```text
error: trivial numeric cast: `i32` as `i32`
 --> lint_example.rs:3:9
  |
3 | let x = 42_i32 as i32;
  |         ^^^^^^^^^^^^^
  |
  = help: cast can be replaced by coercion; this might require a temporary variable
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(trivial_numeric_casts)]
  |         ^^^^^^^^^^^^^^^^^^^^^

```

### Explanation

琐碎的数值类型转换是指将一种数值类型转换为同一种数值类型。这种类型的转换通常是不必要的。

这个 lint 默认设置为 “allow”，因为在某些情况下，例如与 FFI 接口或复杂类型别名一起使用时，它可能错误地触发，或者在某些情况下，明确表达意图可能更加困难。在未来，这可能会成为一个警告，可能会通过一种显式的强制转换语法来方便地解决当前的问题。请参见[RFC 401（强制转换）][rfc-401]、[RFC 803（类型标注）][rfc-803]和[RFC 3307（移除类型标注）][rfc-3307]以了解历史背景。

[rfc-401]: https://github.com/rust-lang/rfcs/blob/master/text/0401-coercions.md
[rfc-803]: https://github.com/rust-lang/rfcs/blob/master/text/0803-type-ascription.md
[rfc-3307]: https://github.com/rust-lang/rfcs/blob/master/text/3307-de-rfc-type-ascription.md

## unnameable-types

The `unnameable_types` lint detects types for which you can get objects of that type,
but cannot name the type itself.

### Example

```rust,compile_fail
# #![feature(type_privacy_lints)]
# #![allow(unused)]
#![deny(unnameable_types)]
mod m {
    pub struct S;
}

pub fn get_unnameable() -> m::S { m::S }
# fn main() {}
```

This will produce:

```text
error: struct `S` is reachable but cannot be named
 --> lint_example.rs:5:5
  |
5 |     pub struct S;
  |     ^^^^^^^^^^^^ reachable at visibility `pub`, but can only be named at visibility `pub(crate)`
  |
note: the lint level is defined here
 --> lint_example.rs:3:9
  |
3 | #![deny(unnameable_types)]
  |         ^^^^^^^^^^^^^^^^

```

### Explanation

It is often expected that if you can obtain an object of type `T`, then
you can name the type `T` as well, this lint attempts to enforce this rule.

## unreachable-pub

`unreachable_pub` lint 会在从 crate 根部无法访问的 pub 项上触发。

### Example

```rust,compile_fail
#![deny(unreachable_pub)]
mod foo {
    pub mod bar {

    }
}
```

This will produce:

```text
error: unreachable `pub` item
 --> lint_example.rs:4:5
  |
4 |     pub mod bar {
  |     ---^^^^^^^^
  |     |
  |     help: consider restricting its visibility: `pub(crate)`
  |
  = help: or consider exporting it for use by other crates
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(unreachable_pub)]
  |         ^^^^^^^^^^^^^^^

```

### Explanation

`pub` 关键字既表达了将项目公开可用的意图，也向编译器发出使项目可公开访问的信号。然而，只有当包含此项目的所有项目也都公开可访问时，这种意图才能实现。因此，这个 lint 用于识别意图与现实不匹配的情况。

如果你希望项目在 crate 内的其他地方可访问，但不在外部可访问，建议使用 `pub(crate)` 可见性。这更清楚地表达了项目只在自己的 crate 内可见的意图。

这个 lint 默认设置为 “allow”，因为它将为大量现有的 Rust 代码触发，并且有一些误报。最终，我们希望 lint 成为默认警告。

## unsafe-code

`unsafe_code` lint 用于捕捉 `unsafe` 代码的使用以及其他可能不健全的结构，如 `no_mangle`、`export_name` 和 `link_section`。

### Example

```rust,compile_fail
#![deny(unsafe_code)]
fn main() {
    unsafe {

    }
}

#[no_mangle]
fn func_0() { }

#[export_name = "exported_symbol_name"]
pub fn name_in_rust() { }

#[no_mangle]
#[link_section = ".example_section"]
pub static VAR1: u32 = 1;
```

This will produce:

```text
error: usage of an `unsafe` block
 --> lint_example.rs:3:5
  |
3 | /     unsafe {
4 | |
5 | |     }
  | |_____^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(unsafe_code)]
  |         ^^^^^^^^^^^


error: declaration of a `no_mangle` function
 --> lint_example.rs:8:1
  |
8 | #[no_mangle]
  | ^^^^^^^^^^^^
  |
  = note: the linker's behavior with multiple libraries exporting duplicate symbol names is undefined and Rust cannot provide guarantees when you manually override them


error: declaration of a function with `export_name`
  --> lint_example.rs:11:1
   |
11 | #[export_name = "exported_symbol_name"]
   | ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   |
   = note: the linker's behavior with multiple libraries exporting duplicate symbol names is undefined and Rust cannot provide guarantees when you manually override them


error: declaration of a `no_mangle` static
  --> lint_example.rs:14:1
   |
14 | #[no_mangle]
   | ^^^^^^^^^^^^
   |
   = note: the linker's behavior with multiple libraries exporting duplicate symbol names is undefined and Rust cannot provide guarantees when you manually override them


error: declaration of a static with `link_section`
  --> lint_example.rs:15:1
   |
15 | #[link_section = ".example_section"]
   | ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   |
   = note: the program's behavior with overridden link sections on items is unpredictable and Rust cannot provide guarantees when you manually override them

```

### Explanation

该 lint 旨在限制 `unsafe` 代码块和其他结构（包括但不限于`no_mangle`、`link_section` 和 `export_name`属性）的使用，这些结构的错误使用会导致未定义的行为。

## unsafe-op-in-unsafe-fn

`unsafe_op_in_unsafe_fn` lint 用于检测在不安全的函数中执行不安全操作，但没有显式的不安全代码块。

### Example

```rust,compile_fail
#![deny(unsafe_op_in_unsafe_fn)]

unsafe fn foo() {}

unsafe fn bar() {
    foo();
}

fn main() {}
```

This will produce:

```text
error: call to unsafe function is unsafe and requires unsafe block (error E0133)
 --> lint_example.rs:6:5
  |
6 |     foo();
  |     ^^^^^ call to unsafe function
  |
  = note: consult the function's documentation for information on how to avoid undefined behavior
note: an unsafe function restricts its caller, but its body is safe by default
 --> lint_example.rs:5:1
  |
5 | unsafe fn bar() {
  | ^^^^^^^^^^^^^^^
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(unsafe_op_in_unsafe_fn)]
  |         ^^^^^^^^^^^^^^^^^^^^^^

```

### Explanation

目前，一个 [`unsafe fn`] 允许其体内执行任何 [unsafe] 的操作。然而，这会增加需要仔细检查以确保正确行为的代码的表面区域。[`unsafe` block]提供了一种方便的方式来清楚地表明代码的哪些部分正在执行不安全操作。在未来，我们希望改变这种状况，使得在 `unsafe fn` 中不能在没有 `unsafe` 块的情况下执行不安全操作。

解决这个问题的方法是将不安全的代码包装在一个 `unsafe` 块中。

这个 lint 在 2021 年之前的版本中默认是 “allow” ，从 2024 年开始默认是 “warn”；进一步提高严重性的计划仍在考虑中。更多详情见[RFC #2585]和[issue #71668]。

[`unsafe fn`]: https://doc.rust-lang.org/reference/unsafe-functions.html
[`unsafe` block]: https://doc.rust-lang.org/reference/expressions/block-expr.html#unsafe-blocks
[unsafe]: https://doc.rust-lang.org/reference/unsafety.html
[RFC #2585]: https://github.com/rust-lang/rfcs/blob/master/text/2585-unsafe-block-in-unsafe-fn.md
[issue #71668]: https://github.com/rust-lang/rust/issues/71668

## unstable-features

`unstable_features` lint 已被废弃，不应再使用。

## unused-crate-dependencies

`unused_crate_dependencies` lint 用于检测未被使用的 crate 依赖项。

### Example

```rust,ignore (needs extern crate)
#![deny(unused_crate_dependencies)]
```

This will produce:

```text
error: external crate `regex` unused in `lint_example`: remove the dependency or add `use regex as _;`
  |
note: the lint level is defined here
 --> src/lib.rs:1:9
  |
1 | #![deny(unused_crate_dependencies)]
  |         ^^^^^^^^^^^^^^^^^^^^^^^^^
```

### Explanation

将使用了依赖项的代码移除之后，通常需要从构建配置中删除依赖。然而，有时可能会忘记这一步，导致浪费时间来构建不再使用的依赖项。该 lint 可以被用来检测从未使用的依赖项（更具体地说，那些从未被 [`use`], [`extern crate`],或者任何[path]指向的，通过 `--extern`命令行标签指定的依赖）

该 lint 默认等级为 “allow” ，因为根据构建系统的配置不同可能会产生误报。例如，当使用 Cargo 时，一个 “package” 包含了多个 crate （例如一个库 crate 和一个二进制 crate ），但是这个包的依赖是为整体而定义的，如果有一个依赖仅在二进制 crate 中使用，在库 crate 中未使用，那么该 lint 将库 crate 运行的时候错误地被发出

[path]: https://doc.rust-lang.org/reference/paths.html
[`use`]: https://doc.rust-lang.org/reference/items/use-declarations.html
[`extern crate`]: https://doc.rust-lang.org/reference/items/extern-crates.html

## unused-extern-crates

`unused_extern_crates` lint 防止从未被使用的 `extern crate` 项。

### Example

```rust,compile_fail
#![deny(unused_extern_crates)]
#![deny(warnings)]
extern crate proc_macro;
```

This will produce:

```text
error: unused extern crate
 --> lint_example.rs:4:1
  |
4 | extern crate proc_macro;
  | ^^^^^^^^^^^^^^^^^^^^^^^^ help: remove it
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(unused_extern_crates)]
  |         ^^^^^^^^^^^^^^^^^^^^

```

### Explanation

未使用的 `extern crate` 项是无效的应该被删除。请注意，在某些情况下，需要指定 `extern crate` 以确保他们被 crate 所链接，即使没有直接引用它。可以通过为 crate 取一个下划线别名来消除检测，例如 `extern crate foo as _`。还要注意的是 `extern crate` 在 [2018 edition]中已经不常用，因为现在外部 crate（译者注：被指定的）会被自动添加到域中。

该 lint 默认等级为 “allow” ，因为其可能会造成干扰和误报。如果要从项目中移除依赖，推荐在构建配置中将其删除（例如 Cargo.toml）确保编译时不会留下陈旧的构建条目。

[2018 edition]: https://doc.rust-lang.org/edition-guide/rust-2018/module-system/path-clarity.html#no-more-extern-crate

## unused-import-braces

`unused_import_braces` lint 用于捕捉导入项周围不必要的大括号。

### Example

```rust,compile_fail
#![deny(unused_import_braces)]
use test::{A};

pub mod test {
    pub struct A;
}
# fn main() {}
```

This will produce:

```text
error: braces around A is unnecessary
 --> lint_example.rs:2:1
  |
2 | use test::{A};
  | ^^^^^^^^^^^^^^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(unused_import_braces)]
  |         ^^^^^^^^^^^^^^^^^^^^

```

### Explanation

如果仅有单个项，应该移除大括号（例如 `use test::A;`）。

该 lint 默认等级为 “allow” ，因为其只是强制执行样式选择。

## unused-lifetimes

`unused_lifetimes` lint 检测未使用的生命周期参数。

### Example

```rust,compile_fail
#[deny(unused_lifetimes)]

pub fn foo<'a>() {}
```

This will produce:

```text
error: lifetime parameter `'a` never used
 --> lint_example.rs:4:12
  |
4 | pub fn foo<'a>() {}
  |           -^^- help: elide the unused lifetime
  |
note: the lint level is defined here
 --> lint_example.rs:2:8
  |
2 | #[deny(unused_lifetimes)]
  |        ^^^^^^^^^^^^^^^^

```

### Explanation

未使用的生命周期参数可能有错误或是代码未完成。应该考虑删除该参数

## unused-macro-rules

The `unused_macro_rules` lint detects macro rules that were not used.

Note that the lint is distinct from the `unused_macros` lint, which
fires if the entire macro is never called, while this lint fires for
single unused rules of the macro that is otherwise used.
`unused_macro_rules` fires only if `unused_macros` wouldn't fire.

### Example

```rust
#[warn(unused_macro_rules)]
macro_rules! unused_empty {
    (hello) => { println!("Hello, world!") }; // This rule is unused
    () => { println!("empty") }; // This rule is used
}

fn main() {
    unused_empty!(hello);
}
```

This will produce:

```text
warning: 2nd rule of macro `unused_empty` is never used
 --> lint_example.rs:4:5
  |
4 |     () => { println!("empty") }; // This rule is used
  |     ^^
  |
note: the lint level is defined here
 --> lint_example.rs:1:8
  |
1 | #[warn(unused_macro_rules)]
  |        ^^^^^^^^^^^^^^^^^^

```

### Explanation

Unused macro rules may signal a mistake or unfinished code. Furthermore,
they slow down compilation. Right now, silencing the warning is not
supported on a single rule level, so you have to add an allow to the
entire macro definition.

If you intended to export the macro to make it
available outside of the crate, use the [`macro_export` attribute].

[`macro_export` attribute]: https://doc.rust-lang.org/reference/macros-by-example.html#path-based-scope

## unused-qualifications

`unused_qualifications` lint 检测不必要的限定名。

### Example

```rust,compile_fail
#![deny(unused_qualifications)]
mod foo {
    pub fn bar() {}
}

fn main() {
    use foo::bar;
    foo::bar();
}
```

This will produce:

```text
error: unnecessary qualification
 --> lint_example.rs:8:5
  |
8 |     foo::bar();
  |     ^^^^^^^^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(unused_qualifications)]
  |         ^^^^^^^^^^^^^^^^^^^^^
help: remove the unnecessary path segments
  |
8 -     foo::bar();
8 +     bar();
  |

```

### Explanation

如果来自另一个模块的项已经被导入了这个域，在这种情况下无需为该项加限定名，你可以不加 `foo::` 直接调用 `bar()`。

该 lint 默认等级为 “allow”，因为这有些花哨（pedantic），并不表示实际问题，而且是一种风格选择，并且当重构或移动代码的时候可能会带来干扰。

## unused-results

`unused_results` lint 检查语句中表达式未使用的 result 。

### Example

```rust,compile_fail
#![deny(unused_results)]
fn foo<T>() -> T { panic!() }

fn main() {
    foo::<usize>();
}
```

This will produce:

```text
error: unused result of type `usize`
 --> lint_example.rs:5:5
  |
5 |     foo::<usize>();
  |     ^^^^^^^^^^^^^^^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(unused_results)]
  |         ^^^^^^^^^^^^^^

```

### Explanation

忽略函数的返回值可能表明存在错误。在几乎可以肯定 result 应该被使用的情况下，建议使用[`must_use` attribute]来注释函数。未使用这样的返回值将触发默认警告的[`unused_must_use` lint]。`unused_results` lint 本质上是一样的，但它会触发*所有*返回值。

该 lint 默认等级为 “allow” ，因为其可能会带来干扰，且可能并不是一个真正的问题。例如，调用 `Vec` 或 `HashMap` 的  `remove` 方法会返回先前的值，你可能并不关心这个值，使用这个 lint 将会要求显式地忽略或丢弃这些值。

[`must_use` attribute]: https://doc.rust-lang.org/reference/attributes/diagnostics.html#the-must_use-attribute
[`unused_must_use` lint]: warn-by-default.html#unused-must-use

## unused-tuple-struct-fields

The `unused_tuple_struct_fields` lint detects fields of tuple structs
that are never read.

### Example

```rust
#[warn(unused_tuple_struct_fields)]
struct S(i32, i32, i32);
let s = S(1, 2, 3);
let _ = (s.0, s.2);
```

This will produce:

```text
warning: field `1` is never read
 --> lint_example.rs:3:15
  |
3 | struct S(i32, i32, i32);
  |        -      ^^^
  |        |
  |        field in this struct
  |
note: the lint level is defined here
 --> lint_example.rs:2:8
  |
2 | #[warn(unused_tuple_struct_fields)]
  |        ^^^^^^^^^^^^^^^^^^^^^^^^^^
help: consider changing the field to be of unit type to suppress this warning while preserving the field numbering, or remove the field
  |
3 | struct S(i32, (), i32);
  |               ~~

```

### Explanation

Tuple struct fields that are never read anywhere may indicate a
mistake or unfinished code. To silence this warning, consider
removing the unused field(s) or, to preserve the numbering of the
remaining fields, change the unused field(s) to have unit type.

## variant-size-differences

`variant_size_differences` lint 检测具有不同变量大小的枚举。

### Example

```rust,compile_fail
#![deny(variant_size_differences)]
enum En {
    V0(u8),
    VBig([u8; 1024]),
}
```

This will produce:

```text
error: enum variant is more than three times larger (1024 bytes) than the next largest
 --> lint_example.rs:5:5
  |
5 |     VBig([u8; 1024]),
  |     ^^^^^^^^^^^^^^^^
  |
note: the lint level is defined here
 --> lint_example.rs:1:9
  |
1 | #![deny(variant_size_differences)]
  |         ^^^^^^^^^^^^^^^^^^^^^^^^

```

### Explanation

向枚举中添加一个比其他变量大得多的变量可能是个错误，这会增加所有变量所需空间的总大小。这可能会影响性能和内存使用。如果第一大的变量比第二大的变量所需空间大三倍以上，就会触发这个 lint。

可以考虑将较大变量的内容放在堆上（例如通过 [`Box`]），以保持枚举体自身大小处于较小量值 。

该 lint 默认等级为 “allow” ，因为其可能会造成干扰，且可能并不是一个真正的问题。应通过基准测试和分析指导来考虑这个问题。

[`Box`]: https://doc.rust-lang.org/std/boxed/index.html

