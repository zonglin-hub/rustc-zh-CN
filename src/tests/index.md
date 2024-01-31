# Tests

`rustc` 有一个内置的用于为 crate 构建和运行测试的设备。
更多关于编写和运行测试的信息可以在 Rust 编程语言书的 [Testing Chapter] 中找到。

测试被编写为带有 `#[test]` 属性的自由函数。例如：

```rust
#[test]
fn it_works() {
    assert_eq!(2 + 2, 4);
}
```

测试“通过”的条件是它们无错误地返回。如果它们 “失败”，则它们要么 [panic]，要么返回一种类型，如[`Result`]，它实现了 [`Termination`] trait 并且值不为零。

通过将 [`--test` option] 传递给 `rustc`，编译器将以特殊模式构建 crate，以构造一个可执行文件，该文件将运行 crate 中的测试。`--test`标志将进行以下更改：

* crate将构建为 `bin` [crate type]，强制其为可执行文件。
* 将可执行文件与 [`libtest`] 链接，这是标准库中的测试工具，用于处理运行测试。
* 合成一个 [`main` function]，它将处理命令行参数并运行测试。
  这个新的 `main` 函数将替换现有的 `main` 函数作为可执行文件的入口点，尽管现有的 `main` 仍将被编译。
* 启用 [`test` cfg option]，它允许您的代码使用条件编译来检测其是否作为测试构建。
* 启用带有 [`test`][attribute-test] 和 [`bench`](#benchmarks) 属性的函数的构建，测试工具将运行这些函数。

创建可执行文件后，您可以运行它以执行测试并接收有关通过和失败的报告。
如果您使用 [Cargo] 来管理项目，它有一个内置的 [`cargo test`] 命令，可以自动处理这些。输出的示例如下所示：

```text
running 4 tests
test it_works ... ok
test check_valid_args ... ok
test invalid_characters ... ok
test walks_the_dog ... ok

test result: ok. 4 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

> **注意**：测试必须使用 [`unwind` panic strategy][panic-strategy] 构建。
> 这是因为所有测试都在同一进程中运行，并且旨在捕获 panics，而使用 `abort` 策略无法实现这一点。
> 可以使用不稳定的 [`-Z panic-abort-tests`] 选项在实验上通过分别在单独的进程中运行测试来支持 `abort` 策略。

## Test attributes

测试通过在自由函数上使用属性来表示。以下属性用于测试，更多详情请参阅链接的文档：

* [`#[test]`][attribute-test] - 表示一个函数是要运行的测试。
* `#[bench]` - 表示一个函数是要运行的基准测试。
  基准测试目前不稳定，仅在 nightly 频道可用，更多详情请参阅 [unstable docs][bench-docs]。
* [`#[should_panic]`][attribute-should_panic] - 表示测试函数只有在函数 [panics][panic] 时才会通过。
* [`#[ignore]`][attribute-ignore] - 表示测试函数将被编译，但默认不运行。
  请参阅 [`--ignored`](#--ignored) 和 [`--include-ignored`](#--include-ignored) 选项以运行这些测试。

## CLI arguments

libtest 测试工具具有几个命令行参数来控制其行为。

> 注意：在使用 [`cargo test`] 运行时，libtest CLI 参数必须在 `--` 参数之后传递，以区分 Cargo 的标志和测试工具的标志。例如：`cargo test -- --nocapture`

### Filters

位置参数（没有 `-` 前缀的参数）被视为过滤器，只会运行名称与这些字符串之一匹配的测试。
过滤器将匹配测试函数完整路径中找到的任何子字符串。
例如，如果测试函数 `it_works` 位于模块 `utils::paths::tests` 中，那么任何过滤器 `works`、`path`、`utils::`或 `utils::paths::tests::it_works` 都会匹配该测试。

有关更多选项来控制运行的测试，请参阅 [Selection options](#selection-options)。

### Action options

以下选项执行除运行测试之外的不同操作。

#### `--list`

打印所有测试和基准测试的列表。不运行任何测试。[Filters](#filters) 可用于仅列出匹配的测试。

#### `-h`, `--help`

显示使用信息和命令行选项。

### Selection options

以下选项更改测试的选择方式。

#### `--test`

这是默认模式，将运行所有测试以及仅进行一次迭代的所有基准测试（以确保基准测试有效，而无需花费实际进行基准测试的时间）。这可与 `--bench` 标志结合使用，以运行测试并执行完整的基准测试。

#### `--bench`

此模式将忽略测试，仅运行基准测试。这可与 `--test` 结合使用，以同时运行基准测试和测试。

#### `--exact`

这将强制 [filters](#filters) 与测试的完整路径完全匹配。例如，如果测试 `it_works` 在模块`utils::paths::tests`中，则只有字符串 `utils::paths::tests::it_works` 才会匹配该测试。

#### `--skip` _FILTER_

跳过名称包含给定 _FILTER_ 字符串的任何测试。此标志可以传递多次。

#### `--ignored`

仅运行标记为 [`ignore`
attribute][attribute-ignore] 的测试。

#### `--include-ignored`

运行[被忽略的](#--ignored)和未被忽略的测试。

#### `--exclude-should-panic`

排除标记为 [`should_panic`
attribute][attribute-should_panic] 的测试。

⚠️ 🚧此选项 [unstable](#unstable-options)，需要使用 `-Z unstable-options` 标志。
有关更多信息，请参阅 [tracking issue #82348](https://github.com/rust-lang/rust/issues/82348)。

### Execution options

以下选项会影响测试的执行方式。

#### `--test-threads` _NUM_THREADS_

设置用于并行运行测试的线程数。默认情况下，使用 [`available_parallelism`] 指示的硬件上的并发量。

也可以使用 `RUST_TEST_THREADS` 环境变量指定。

#### `--force-run-in-process`

在使用 [`abort` panic
strategy][panic-strategy] 时，强制测试在单个进程中运行。

⚠️ 🚧这仅适用于不稳定的 [`-Z panic-abort-tests`] 选项，并需要使用 `-Z unstable-options` 标志。有关更多信息，请参阅[tracking issue #67650](https://github.com/rust-lang/rust/issues/67650)。

#### `--ensure-time`

⚠️ 🚧此选项 [unstable](#unstable-options)，需要使用 `-Z unstable-options` 标志。有关更多信息，请参阅 [tracking issue #64888](https://github.com/rust-lang/rust/issues/64888) 和 [unstable docs](../../unstable-book/compiler-flags/report-time.html)。

#### `--shuffle`

以随机顺序运行测试，而不是默认的字母顺序。

也可以通过将 `RUST_TEST_SHUFFLE` 环境变量设置为除 `0` 以外的任何值来指定。

输出的随机数生成器种子可以传递给[`--shuffle-seed`](#--shuffle-seed-seed)，以再次以相同的顺序运行测试。

请注意，`--shuffle`不会影响测试是否并行运行。要以随机顺序依次运行测试，请使用`--shuffle --test-threads 1`。

⚠️ 🚧此选项 [unstable](#unstable-options)，需要使用 `-Z unstable-options` 标志。有关更多信息，请参阅 [tracking issue #89583](https://github.com/rust-lang/rust/issues/89583)。

#### `--shuffle-seed` _SEED_

与 [`--shuffle`](#--shuffle) 类似，但使用 _SEED_ 为随机数生成器种子。因此，使用 `--shuffle-seed` _SEED_ 两次调用测试工具，两次都以相同的顺序运行测试。

_SEED_ 是任何 64 位无符号整数，例如由 [`--shuffle`](#--shuffle) 生成的整数。

也可以使用 `RUST_TEST_SHUFFLE_SEED` 环境变量指定。

⚠️ 🚧此选项 [unstable](#unstable-options)，需要使用 `-Z unstable-options` 标志。有关更多信息，请参阅[tracking issue #89583](https://github.com/rust-lang/rust/issues/89583)。

### Output options

以下选项会影响输出行为。

#### `-q`, `--quiet`

每个测试显示一个字符，而不是每行一个测试。这是 [`--format=terse`](#--format-format) 的别名。

#### `--nocapture`

不捕获测试的 stdout 和 stderr，并允许测试打印到控制台。通常输出被捕获，并且只在测试失败时显示。

也可以通过将 `RUST_TEST_NOCAPTURE` 环境变量设置为除 `0` 以外的任何值来指定。

#### `--show-output`

在所有测试运行完毕后，显示成功测试的 stdout 和 stderr。

与 [`--nocapture`](#--nocapture) 不同，它允许测试 *在运行时* 打印，如果有多个测试并行运行，可能会导致交错输出，`--show-output` 确保输出是连续的，但需要等待所有测试完成。

#### `--color` _COLOR_

控制何时使用彩色终端输出。有效选项：

* `auto`：如果 stdout 是 tty 且未使用 [`--nocapture`](#--nocapture)，则进行彩色化。这是默认值。
* `always`：总是对输出进行彩色化。
* `never`：从不对输出进行彩色化。

#### `--format` _FORMAT_

控制输出格式。有效选项：

* `pretty`：这是默认格式，每个测试一行。
* `terse`：每个测试仅显示一个字符。[`--quiet`](#-q---quiet) 是此选项的别名。
* `json`：逐行输出JSON对象。⚠️ 🚧此选项 [unstable](#unstable-options)，需要使用 `-Z unstable-options` 标志。有关更多信息，请参阅 [tracking issue #49359](https://github.com/rust-lang/rust/issues/49359)。

#### `--logfile` _PATH_

将测试结果写入给定文件。

#### `--report-time`

⚠️ 🚧此选项 [unstable](#unstable-options)，需要使用 `-Z unstable-options` 标志。有关更多信息，请参阅 [tracking issue #64888](https://github.com/rust-lang/rust/issues/64888) 和 [unstable docs](../../unstable-book/compiler-flags/report-time.html)。

### Unstable options

一些 CLI 选项以 “不稳定” 状态添加，它们旨在进行实验和测试，以确定选项是否正常工作、设计正确且有用。该选项可能无法正常工作、中断或随时更改。为了表示您确认正在使用不稳定的选项，它们需要传递 `-Z unstable-options` 命令行标志。

## Benchmarks

libtest 工具支持运行用 `#[bench]` 属性注释的函数的基准测试。基准测试目前不稳定，只在 [nightly channel] 上可用。更多信息可以在 [unstable book][bench-docs] 中找到。

## Custom test frameworks

在 [nightly channel] 上可以使用自定义测试工具的实验性支持。有关更多信息，请参阅 [tracking issue #50297](https://github.com/rust-lang/rust/issues/50297) 和 [custom_test_frameworks documentation]。

[`--test` option]: ../command-line-arguments.md#option-test
[`-Z panic-abort-tests`]: https://github.com/rust-lang/rust/issues/67650
[`available_parallelism`]: ../../std/thread/fn.available_parallelism.html
[`cargo test`]: ../../cargo/commands/cargo-test.html
[`libtest`]: ../../test/index.html
[`main` function]: ../../reference/crates-and-source-files.html#main-functions
[`Result`]: ../../std/result/index.html
[`Termination`]: ../../std/process/trait.Termination.html
[`test` cfg option]: ../../reference/conditional-compilation.html#test
[attribute-ignore]: ../../reference/attributes/testing.html#the-ignore-attribute
[attribute-should_panic]: ../../reference/attributes/testing.html#the-should_panic-attribute
[attribute-test]: ../../reference/attributes/testing.html#the-test-attribute
[bench-docs]: ../../unstable-book/library-features/test.html
[Cargo]: ../../cargo/index.html
[crate type]: ../../reference/linkage.html
[custom_test_frameworks documentation]: ../../unstable-book/language-features/custom-test-frameworks.html
[nightly channel]: ../../book/appendix-07-nightly-rust.html
[panic-strategy]: ../../book/ch09-01-unrecoverable-errors-with-panic.html
[panic]: ../../book/ch09-01-unrecoverable-errors-with-panic.html
[Testing Chapter]: ../../book/ch11-00-testing.html
