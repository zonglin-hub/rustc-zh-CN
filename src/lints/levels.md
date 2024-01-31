# Lint Levels

在 `rustc` 里, lints 被分为 5 个 *levels*（级别）:

1. allow (允许) 
2. warn（警告）
3. force-warn（强制警告）
4. deny（拒绝）
5. forbid（禁止）

每一个 lint 都有一个默认级别 (会在本章后面lint列表中解释), 并且编译器有一个默认的警告级别。
首先, 让我们解释一下这些级别的含义, 然后再去讨论关于它们的配置.

## allow

这个 lints 存在, 但默认情况下不执行任何操作. 例如下面这个源码:

```rust
pub fn foo() {}
```

编译这个文件不会产生任何警告:

```bash
$ rustc lib.rs --crate-type=lib
$
```

但是此代码违反了 `missing_docs` lint.

这些 lints 主要是通过配置手动去开启的, 我们将在本章节后面再讨论.

## warn

如果你违反了 warn 这种 lint 的级别，将会在编译时产生一些警告. 例如下面的代码与 `unused_variables` lint 发生冲突:

```rust
pub fn foo() {
    let x = 5;
}
```

这将会产生如下所示的 warn:

```bash
$ rustc lib.rs --crate-type=lib
warning: unused variable: `x`
 --> lib.rs:2:9
  |
2 |     let x = 5;
  |         ^
  |
  = note: `#[warn(unused_variables)]` on by default
  = note: to avoid this warning, consider using `_x` instead
```

## force-warn

'force-warn' 是一种特殊的 lint 级别。它和 'warn' 一样，都会产生警告，但与 'warn' 不同的是，'force-warn' 级别无法被覆盖。
如果将 lint 设置为 'force-warn'，则一定会产生警告：不多也不少。即使通过 cap-lints 限制了整体 lint 级别，也是如此。

## deny

一个 'deny' lint 将会在你违反其规则时产生一个错误。例如下面的代码与 `exceeding_bitshifts` lint 相冲突。

```rust,no_run
fn main() {
    100u8 << 10;
}
```

```bash
$ rustc main.rs
error: bitshift exceeds the type's number of bits
 --> main.rs:2:13
  |
2 |     100u8 << 10;
  |     ^^^^^^^^^^^
  |
  = note: `#[deny(exceeding_bitshifts)]` on by default
```

 lint 和普通错误之间的区别是什么？
 可以通过级别配置 lints，因此，与 'allow' lints 类似，默认 'deny' 的警告可以允许。
 同样，你可能希望设置一个默认 `warn` 的 lint 来产生错误。这个 lint 级别为你提供了这种可能性。

## forbid

'forbid' 是一种特殊的 lint 级别，它比 'deny' 等级更高。 它与 'deny' 一样会发出一个错误，但是与 'deny' 级别不同的是, 'forbid' 级别不能被比错误更低的情况覆盖了。然而, lint 仍然被 `--cap-lints` (参阅下文) 限制, 因此 `rustc --cap-lints warn` 命令将 'forbid' 级别的lints 设置为仅有警告信息。

## Configuring warning levels

记得我们上面默认级别为 'allow' 的 lint `missing_docs` 的例子的吗?

```bash
$ cat lib.rs
pub fn foo() {}
$ rustc lib.rs --crate-type=lib
$
```

这样可以配置此 lint 在更高级别上运行,两者都使用编译器标志以及源代码中的属性。

你还可以 "cap" lints 以便编译器可以选择忽略某些 lint 级别. 我们将在最后讨论。

### Via compiler flag

比如 `-A`, `-W`, `--force-warn` `-D`, 和 `-F` 标志允许你将一个或多个 lint 转换为允许、警告、强制警告、拒绝或禁止级别，如下所示：

```bash
$ rustc lib.rs --crate-type=lib -W missing-docs
warning: missing documentation for crate
 --> lib.rs:1:1
  |
1 | pub fn foo() {}
  | ^^^^^^^^^^^^
  |
  = note: requested on the command line with `-W missing-docs`

warning: missing documentation for a function
 --> lib.rs:1:1
  |
1 | pub fn foo() {}
  | ^^^^^^^^^^^^
```

```bash
$ rustc lib.rs --crate-type=lib -D missing-docs
error: missing documentation for crate
 --> lib.rs:1:1
  |
1 | pub fn foo() {}
  | ^^^^^^^^^^^^
  |
  = note: requested on the command line with `-D missing-docs`

error: missing documentation for a function
 --> lib.rs:1:1
  |
1 | pub fn foo() {}
  | ^^^^^^^^^^^^

error: aborting due to 2 previous errors
```

你也可以多次传递每个标签，来修改多个 lints 的级别:

```bash
$ rustc lib.rs --crate-type=lib -D missing-docs -D unused-variables
```

当然，你可以将这 5 个 flags 混合在一起使用：

```bash
$ rustc lib.rs --crate-type=lib -D missing-docs -A unused-variables
```

这些命令行参数的顺序也考虑在内了。以下内容允许 `unused-variables` lint， 是因为它是该 lint 的最后一个参数：

```bash
$ rustc lib.rs --crate-type=lib -D unused-variables -A unused-variables
```

您可以通过此举在一组 lints 中覆盖某个特定的 lint 的级别来。
以下例子将在 `unused` 组中所有 lint 设置为 拒绝（ denies ）级别，但是将此组中的 `unused-variables` lint 设置为允许( allows )。(无论顺序如何，"forbid" 仍然胜过一切):

```bash
$ rustc lib.rs --crate-type=lib -D unused -A unused-variables
```

由于 `force-warn` and `forbid` 无法被覆盖，设置其中一个将防止相同 lint 的任何后续级别生效。

### Via an attribute

你也可以通过设置一个 `crate-wide` 属性来修改 lint 的级别:

```bash
$ cat lib.rs
#![warn(missing_docs)]

pub fn foo() {}
$ rustc lib.rs --crate-type=lib
warning: missing documentation for crate
 --> lib.rs:1:1
  |
1 | / #![warn(missing_docs)]
2 | |
3 | | pub fn foo() {}
  | |_______________^
  |
note: lint level defined here
 --> lib.rs:1:9
  |
1 | #![warn(missing_docs)]
  |         ^^^^^^^^^^^^

warning: missing documentation for a function
 --> lib.rs:3:1
  |
3 | pub fn foo() {}
  | ^^^^^^^^^^^^
```

`warn`, `allow`, `deny`, and `forbid`  都是以这种方式工作的。无法使用属性将 lint 设置为 `force-warn` 。

你还可以为每个属性传递多个 lint ：

```rust
#![warn(missing_docs, unused_variables)]

pub fn foo() {}
```

同时使用多个属性：

```rust
#![warn(missing_docs)]
#![deny(unused_variables)]

pub fn foo() {}
```

### Capping lints

`rustc` 支持一个标签, `--cap-lints LEVEL` 用来设置 "lint cap level"。即此设置所有的 lint 的最高级别。
如下所示， 如果我们采用上面 "deny" lint 级别代码示例:

```rust,no_run
fn main() {
    100u8 << 10;
}
```

然后我们编译它, 限制 lints 为 warn 级别:

```bash
$ rustc lib.rs --cap-lints warn
warning: bitshift exceeds the type's number of bits
 --> lib.rs:2:5
  |
2 |     100u8 << 10;
  |     ^^^^^^^^^^^
  |
  = note: `#[warn(exceeding_bitshifts)]` on by default

warning: this expression will panic at run-time
 --> lib.rs:2:5
  |
2 |     100u8 << 10;
  |     ^^^^^^^^^^^ attempt to shift left with overflow
```

It now only warns, rather than errors. We can go further and allow all lints:

现在它只发出 warns，而不是 errors。我们可以进一步允许所有 lint：

```bash
$ rustc lib.rs --cap-lints allow
```

在 Cargo 中这个特性被使用的很多; 它会在我们编译依赖时设置 `--cap-lints allow` 命令, 以便在有任何警告信息的时候, 不会影响到我们的构建输出。
