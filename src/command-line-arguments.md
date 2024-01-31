# 命令行参数

`rustc` 的命令行参数及其功能的列表。

<a id="option-help"></a>
## `-h`/`--help`: 获取帮助信息

该标志将打印出 `rustc` 的帮助信息。

<a id="option-cfg"></a>
## `--cfg`: 配置编译环境


此标志可以开启或关闭各种 `#[cfg]` 设置，用于[条件编译](../reference/conditional-compilation.md)。

该值可以是单个标识符，也可以是两个标识符，中间用等号 `=` 分隔。

例如，`--cfg 'verbose'` 或 `--cfg 'feature="serde"'`。 
它们分别对应 `#[cfg(verbose)]` 和 `#[cfg(feature = "serde")]`。

<a id="option-l-search-path"></a>
## `-L`: 将目录添加到库搜索路径中

`-L` 标志将路径添加到外部 crate 和库的搜索路径中。

可以通过 `-L KIND=PATH` 形式指定搜索路径的类型，其中 `KIND` 可以是以下之一：

- `dependency` - 在此目录中仅搜索传递性依赖项。
- `crate` - 在此目录中仅搜索此 crate 的直接依赖项。
- `native` - 在此目录中仅搜索本机库。
- `framework` - 在此目录中仅搜索 macOS 框架。
- `all` - 在此目录中搜索所有库类型。如果没有指定 `KIND`，则这是默认值。

<a id="option-l-link-lib"></a>
## `-l`: 将生成的 crate 链接到本机库

语法: `-l [KIND[:MODIFIERS]=]NAME[:RENAME]`.

这个标志允许你在构建crate时指定链接到特定的本机库。

库的类型可以使用 `-l KIND=lib` 的形式指定，其中 `KIND` 可以是以下之一：

- `dylib` - 本机动态库。
- `static` - 本机静态库（如 `.a` 档案）。
- `framework` - macOS 框架。

如果指定了库的类型，则可以附加链接修饰符。修饰符被指定为一个逗号分隔的字符串，每个修饰符都以前缀 `+` 或 `-` 表示修饰符是启用还是禁用。
目前不支持在单个 `link` 属性中指定多个 `modifiers` 参数，或在同一 `modifiers` 参数中指定多个相同的修饰符。
例如：`-l static:+whole-archive=mylib`。

库的类型和修饰符也可以在 `#[link]` 属性中指定。
如果在 `link` 属性或命令行中未指定库的类型，则除构建静态可执行文件外，默认情况下将链接动态库。
如果在命令行上指定了库的类型，它将覆盖在 `link` 属性中指定的类型。

可以使用 `-l ATTR_NAME:LINK_NAME` 的形式覆盖 `link` 属性中使用的名称，
其中 `ATTR_NAME` 是 `link` 属性中的名称，`LINK_NAME` 是将要链接的实际库的名称。

[link-attribute]: ../reference/items/external-blocks.html#the-link-attribute

### Linking modifiers: `whole-archive`

这个修饰符仅与 `static` 链接类型兼容。使用任何其他类型将导致编译器错误。

`+whole-archive` 表示静态库作为整个归档文件链接，而不丢弃任何目标文件。

该修饰符对于类似 `ld` 的链接器翻译为 `--whole-archive`，
对于 `link.exe` 翻译为 `/WHOLEARCHIVE`，
对于 `ld64` 翻译为 `-force_load`。
对于不支持它的链接器，该修饰符不会有任何作用。

此修饰符的默认值为 `-whole-archive`。
注意：由于向后兼容性的原因，在某些情况下，默认值可能会有所不同，但无法保证。
如果您需要整个归档语义，请显式使用 `+whole-archive`。

### Linking modifiers: `bundle`

该修饰符仅与 `static` 链接类型兼容。使用任何其他类型将导致编译器错误。

在构建 rlib 或 staticlib 时，`+bundle` 表示本机静态库将被打包到 rlib 或 staticlib 归档文件中，
并在最终二进制文件的链接过程中从中检索。

在构建 rlib 时，`-bundle` 表示本机静态库按名称注册为该 rlib 的依赖项，
并且仅在最终二进制文件的链接过程中包含来自它的目标文件，还将在最终链接过程中按该名称搜索文件。
当构建 staticlib 时，`-bundle` 表示本机静态库根本不包含在归档文件中，稍后需要在最终二进制文件的链接过程中添加它。

当构建可执行文件或动态库等其他目标时，此修饰符不会有任何效果。

该修饰符的默认值为 `+bundle`。

### Linking modifiers: `verbatim`

此修饰符与所有链接类型兼容。

`+verbatim` 表示 rustc 本身不会向库名称添加任何目标指定的库前缀或后缀（如 `lib` 或 `.a`），并会尽最大努力向链接器请求相同的东西。

对于支持 GNU 扩展的 `ld` 类链接器，rustc 将使用 `-l:filename` 语法（注意冒号）传递库文件，因此链接器不会向其添加任何前缀或后缀。
有关更多详细信息，请参阅 ld 文档中的 [`-l namespec`](https://sourceware.org/binutils/docs/ld/Options.html)。
对于不支持任何 verbatim 修饰符的链接器（例如 `link.exe` 或 `ld64`），库名称将按原样传递。
因此，此选项最可靠跨平台的使用场景是在不涉及链接器的情况下，例如将本机库打包到 rlib 中。

`-verbatim` 表示 rustc 将在将其传递给链接器之前，要么向库名称添加目标特定的前缀和后缀，要么不阻止链接器隐式添加前缀和后缀。
特别是在 `raw-dylib` 类型的情况下，Windows 上将向库名称添加 `.dll`。

该修饰符的默认值为 `-verbatim`。

注意：即使使用 `+verbatim` 和 `-l:filename` 语法，`ld` 类链接器通常不支持将绝对路径传递给库。
通常，此类路径需要通过不使用任何 `-l` 等选项，作为输入文件传递，例如 `ld /my/absolute/path`。
从稳定的 `rustc` 可以使用 `-Clink-arg=/my/absolute/path` 来实现这一点。

<a id="option-crate-type"></a>
## `--crate-type`: 编译器将要构建 crate 的类型列表

这将指示 `rustc` 以何种 crate type 去构建。
该标签接接收逗号分隔的值列表，也可以多次指定。
有效的 crate type 如下：

- `lib` — 编译器生成的首选库类型， 目前默认为 `rlib`。
- `rlib` — Rust 静态库。
- `staticlib` — 本地静态库。
- `dylib` — Rust 动态库。
- `cdylib` — 本地动态库。
- `bin` — 可执行程序。
- `proc-macro` — 生成格式化且编译器可加载的过程宏库。

可以使用 [`crate_type`][crate_type]属性来指定 crate 类型。
`--crate-type` 命令行的值会覆盖 `crate_type` 属性的值。

更多细节可以参阅 reference 中的 [链接章节][linkage chapter]

[linkage chapter]: ../reference/linkage.html
[crate_type]: ../reference/linkage.html

<a id="option-crate-name"></a>
## `--crate-name`: 指定正在构建的 crate 名称

这将告知 `rustc` 您的 crate 的名称。

<a id="option-edition"></a>
## `--edition`: 指定使用的语义版本

此标志接受的值是 `2015`、`2018` 或 `2021`。默认值为 `2015`。有关版本的更多信息，请参阅 [版本指南]。

[edition guide]: ../edition-guide/introduction.html

<a id="option-emit"></a>
## `--emit`: 指定要生成的输出文件类型

该标签控制编译器生成的输出文件的类型。其接收以逗号分隔的值列表，也可以多次指定。有效的生成类型有：

- `asm` - 生成包含 crate 汇编代码的文件。默认输出文件名为 `CRATE_NAME.s`。
- `dep-info` - 生成包含 Makefile 语法的文件，指示生成 crate 时加载的所有源文件。默认输出文件名为 `CRATE_NAME.d`。
- `link` - 生成 `--crate-type` 指定的 crate。默认输出文件名取决于 crate 类型和平台。如果没有指定 `--emit`，则此选项为默认选项。
- `llvm-bc` - 生成包含 LLVM bitcode 的二进制文件。默认输出文件名为 `CRATE_NAME.bc`。
- `llvm-ir` - 生成包含 LLVM IR 的文件。默认输出文件名为 `CRATE_NAME.ll`。
- `metadata` - 生成包含 crate 元数据的文件。默认输出文件名为 `libCRATE_NAME.rmeta`。
- `mir` - 生成包含 rustc 中级中间表示的文件。默认输出文件名为 `CRATE_NAME.mir`。
- `obj` - 生成本机目标文件。默认输出文件名为 `CRATE_NAME.o`。

输出文件名可以使用`-o`标志进行设置，而使用`-C extra-filename`标志可以为文件名添加后缀。
除非使用`--out-dir`标志，否则文件将写入当前目录。
每种排放类型还可以使用`KIND=PATH`的形式指定输出文件名，这优先于`-o`标志。
指定`-o -`或`--emit KIND=-`使rustc输出到标准输出。
尽管是与tty或非tty连接的文本输出类型(`asm`、`dep-info`、`llvm-ir`和`mir`)，但它们可以被写入到标准输出。
如果任何二进制输出类型被写入到标准输出，那么这将产生一个错误，因为标准输出是一个tty。
如果多个输出类型都被写入到标准输出，那么这将产生一个错误，因为它们会被混合在一起。

[LLVM bitcode]: https://llvm.org/docs/BitCodeFormat.html
[LLVM IR]: https://llvm.org/docs/LangRef.html

<a id="option-print"></a>
## `--print`: 打印编译器信息

这个标志会打印出有关编译器的各种信息。
这个标志可以指定多次，并且信息按照标志指定的顺序打印。
指定`--print`标志通常会禁用`--emit`步骤，并且只打印请求的信息。
有效的打印类型包括：

- `crate-name`：这是指编译的 crate 包的名称。
- `file-names`：这是由 `link` 发射类型创建的文件名的列表。
- `sysroot`：这是 sysroot 的路径。Sysroot 通常是指包含基本系统库和头文件的目录。
- `target-libdir`：这是目标 libdir 的路径。Libdir 是包含库文件的目录。
- `cfg`：cfg 值的列表。关于 cfg 的更多信息，可以参考 [conditional compilation] 的相关内容。
- `target-list`：已知目标的列表。目标可以使用 `--target` 标志进行选择。
- `target-cpus`：当前目标可用的 CPU 值的列表。目标 CPU 可以使用 [`-C target-cpu=val`
  flag](codegen-options/index.md#target-cpu) 进行选择。
- `target-features`：当前目标可用的目标功能的列表。目标功能可以使用 [`-C target-feature=val`
  flag](codegen-options/index.md#target-feature) 标志进行启用。此标志不安全，更多细节请参考[known issues](targets/known-issues.md)部分。
- `relocation-models`：可用的重定位模型的列表。重定位模型可以使用 [`-C relocation-model=val`
  flag](codegen-options/index.md#relocation-model) 进行选择。
- `code-models`：可用的代码模型的列表。代码模型可以使用 [`-C code-model=val` flag](codegen-options/index.md#code-model) 进行选择。
- `tls-models`：支持的线程本地存储模型的列表。模型可以使用 `-Z tls-model=val` 进行选择。
- `native-static-libs`：这个标志在创建`staticlib` crate类型时可以使用。
  如果这是唯一的标志，它将执行完整的编译，并包含一个诊断注释，指示链接静态库时使用的链接器标志。
  注释以`native-static-libs:`文本开始，以方便获取输出。
- `link-args`：此标志不会禁用`--emit`步骤。
  在链接时，此标志使`rustc`以人类可读的形式打印完整的链接器调用。
  这在调试链接器选项时可能是有用的。
  此调试输出的确切格式不是稳定的保证，但会包含链接器可执行文件和传递给链接器的每个命令行参数的文本。
- `deployment-target`：这是当前选择的Apple平台目标的 [deployment target]（或最低OS版本）。
  这值可以用于或传递给其他需要此信息的Rust构建组件，如C编译器。
  如果没有环境变量`*_DEPLOYMENT_TARGET`存在，这将返回 rustc 的最小支持部署目标；否则，将返回变量的解析值。

对于每个请求的信息类型，可以为其指定一个文件路径，格式为`--print KIND=PATH`，就像`--emit`一样。
当指定路径时，信息将被写入该路径而不是输出到标准输出。

[conditional compilation]: ../reference/conditional-compilation.html
[deployment target]: https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/cross_development/Configuring/configuring.html

<a id="option-g-debug"></a>
## `-g`: 包括调试信息

A synonym for [`-C debuginfo=2`](codegen-options/index.md#debuginfo).

<a id="option-o-optimize"></a>
## `-O`: 优化你的代码

A synonym for [`-C opt-level=2`](codegen-options/index.md#opt-level).

<a id="option-o-output"></a>
## `-o`: 输出文件的名称

此标志控制输出文件的名称。

<a id="option-out-dir"></a>
## `--out-dir`: 写入输出的目录

输出的 crate 将被写入到这个目录。如果使用了 [`-o` flag](#option-o-output)，此标志将被忽略。

<a id="option-explain"></a>
## `--explain`: 对错误信息进行详细解释

rustc 的每个错误都有一个错误代码，这将打印出给定错误的更详细解释。

<a id="option-test"></a>
## `--test`: 建立一个测试工具

在编译此 crate 时，`rustc` 将忽略您的 `main` 函数，而是生成一个测试工具。
有关测试的更多信息，请参阅 [Tests 章节](tests/index.md)。

<a id="option-target"></a>
## `--target`: 选择要构建的目标三元组

这控制了要生成哪个[target](targets/index.md)。

<a id="option-w-warn"></a>
## `-W`: 设置 lint 警告

此标志将设置哪些 lint 应设置为 [warn level](lints/levels.md#warn)。

_注意：_ 这些 lint 级别参数的顺序将被考虑，有关更多信息，请参见 [lint level via compiler flag](lints/levels.md#via-compiler-flag)。

<a id="option-force-warn"></a>
## `--force-warn`: 强制 lint 发出警告

此标志将给定的 lint 设置为 [forced warn level](lints/levels.md#force-warn)，该级别无法被覆盖，即使忽略 [lint caps](lints/levels.md#capping-lints)。

<a id="option-a-allow"></a>
## `-A`: 设置 lint 允许

此标志将设置哪些 lint 应设置为 [allow level](lints/levels.md#allow)。

_注意：_ 这些 lint 级别参数的顺序将被考虑，有关更多信息，请参见 [lint level via compiler flag](lints/levels.md#via-compiler-flag)。

<a id="option-d-deny"></a>
## `-D`: 设置 lint 拒绝

此标志将设置哪些 lint 应设置为 [deny level](lints/levels.md#deny)。

_注意：_ 这些 lint 级别参数的顺序将被考虑，有关更多信息，请参见 [lint level via compiler flag](lints/levels.md#via-compiler-flag)。

<a id="option-f-forbid"></a>
## `-F`: 设置 lint 禁止

此标志将设置哪些 lint 应设置为 [forbid level](lints/levels.md#forbid)。

_注意：_ 这些 lint 级别参数的顺序将被考虑，有关更多信息，请参见 [lint level via compiler flag](lints/levels.md#via-compiler-flag)。

<a id="option-z-unstable"></a>
## `-Z`: 设置不稳定的选项

此标志将允许您设置 rustc 的不稳定选项。为了设置多个选项，可以使用 -Z 标志多次。
例如：`rustc -Z verbose -Z time-passes`。
使用 -Z 指定选项仅适用于 nightly 版本。
要查看所有可用选项，请运行：`rustc -Z help`，或参见 [The Unstable Book](../unstable-book/index.html)。

<a id="option-cap-lints"></a>
## `--cap-lints`: 设置最严格的 lint 级别

此标志允许您对 lints 进行 'cap'，有关更多信息，请参见 [see here](lints/levels.md#capping-lints)。

<a id="option-codegen"></a>
## `-C`/`--codegen`: 代码生成选项

此标志允许您设置 [codegen options](codegen-options/index.md).

<a id="option-version"></a>
## `-V`/`--version`: 打印版本信息

此标志将打印 `rustc` 的版本信息。

<a id="option-verbose"></a>
## `-v`/`--verbose`: 使用详细输出

此标志与其他标志结合使用时，会使它们产生额外的输出。

<a id="option-extern"></a>
## `--extern`: 指定外部库的位置

此标志允许您为直接依赖项的外部 crate 传递名称和位置。
间接依赖项（依赖项的依赖项）的位置使用 [`-L` flag](#option-l-search-path) 指定。
给出的 crate 名称将添加到 [extern prelude] 中，类似于在根模块中指定 `extern crate`。
给出的 crate 名称不需要与库的名称匹配。

指定 `--extern` 与 `extern crate` 的行为有一个区别：`--extern` 只是将 crate 视为链接的候选；
除非主动使用，否则它不会实际链接它。
在很少的情况下，您可能希望确保即使您没有从代码中主动使用它，crate 也会被链接：
例如，如果它改变了全局分配器，或者如果它包含供其他编程语言使用的 `#[no_mangle]` 符号。
在这种情况下，您需要使用 `extern crate`。

此标志可以多次指定。此标志接受以下任一格式的参数：

* `CRATENAME=PATH` — 表示给定的 crate 在给定的路径中可以找到。
* `CRATENAME` — 表示给定的 crate 可能在搜索路径中找到，例如在 sysroot 中或通过 `-L` 标志。

同一个 crate 名称可以被不同 crate types 指定多次。
如果同时找到 `rlib` 和 `dylib` （同名）文件， 则使用一个内部算法来决定使用哪一个进行链接。
[`-C prefer-dynamic`][prefer-dynamic] 标签可以被用来影响使用谁。

如果指定了一个带有路径和一个不带路径的同名 crate，则带路径的那个被使用且不带路径的那个无效。

[extern prelude]: ../reference/names/preludes.html#extern-prelude
[prefer-dynamic]: codegen-options/index.md#prefer-dynamic

<a id="option-sysroot"></a>
## `--sysroot`: 覆盖系统根目录

"sysroot" 是 `rustc` 寻找 Rust 发行 crate 的位置；该标签允许被重写。

<a id="option-error-format"></a>
## `--error-format`: 控制如何产生错误

该标签允许你控制消息的格式。 消息被打印到 stderr 。有效的选项有：

- `human` — 人可读的输出格式，此为默认选项。
- `json` — 结构化Json输出。 更多细节请参阅 [the JSON chapter] for more detail。
- `short` — 简短，单行信息。

<a id="option-color"></a>
## `--color`: 配置输出的颜色

该标签允许你配置输出的颜色。有效选项包括：

- `auto` — 如果颜色输出到终端（tty）使用颜色，这是默认选项。
- `always` — 始终使用颜色。
- `never` — 始终不对输出进行着色。

<a id="option-diagnostic-width"></a>
## `--diagnostic-width`: 为诊断信息指定终端宽度

此标志接受一个数字，该数字指定终端的字符宽度。诊断信息的格式化将考虑宽度，以使其更好地适应屏幕。

<a id="option-remap-path-prefix"></a>
## `--remap-path-prefix`: 输出中重映射源名称

在所有输出中重新映射源路径前缀，包括编译器诊断、调试信息、宏扩展等。
它以  `FROM=TO` 形式接收一个值，其中 `FROM` 的路径前缀被重写为值 `TO` 。
`FROM` 本身可能包含 `=` 符号， 但是 `TO` 值不可能包含。该标签可以被多次指定。

这对于规范化构建产品是很有用的，例如，通过从发送到对象文件的路径名中删除当前目录。
该替换是纯文本的，不考虑当前系统的路径名语法。
例如 `--remap-path-prefix foo=bar` 将会匹配 `foo/lib.rs` 而不是 `./foo/lib.rs`。

当给定多个重映射并且它们有几个匹配时，将应用 **last** 匹配的映射。

<a id="option-json"></a>
## `--json`: 配置编译器打印json的消息

当 [`--error-format=json` option](#option-error-format) 传递到 rustc 时，编译器所有的 诊断输出都将以 JSON blobs 的形式发出。 `--json` 参数可以与 `--error-format=json` 参数一起使用以配置 JSON blobs 包含的内容以及发出的内容。

使用 `--error-format=json` 编译器 会将所有编译器错误以 JSON blob 格式发出，但  `--json` 标签也可以用以下选项来自定义输出：

- `diagnostic-short` -  用于诊断消息的json blobs应该使用“ short ”形式呈现，而不是平时的“human”默认值。
  这意味着 `--error-format=short` 的输出将被嵌入到JSON诊断中，而不是默认的 `--error-format=human` 。

- `diagnostic-rendered-ansi` - 默认情况下，JSON blob在其 `rendered` 字段中将包含诊断的纯文本呈现。
  相反，该选项指示诊断应该嵌入ANSI颜色代码用于为消息着色，就像rustc通常对终端输出所做的那样。
  注意通常与像 [`fwdansi`](https://crates.io/crates/fwdansi) 这类的 crate 结合以在 windows console 命令行转换 ANSI 码；
  或者如果你想在其后选择性地移除 ansi 颜色可以结合 [`strip-ansi-escapes`](https://crates.io/crates/strip-ansi-escapes)。

- `artifacts` - 这指示rustc为所发出的每个部件发出一个JSON blob对象。部件对应于来自 [`--emit` CLI](#option-emit) 参数的请求，一旦部件在文件系统上可用，就会发出相应通知。

- `future-incompat` - 包含一个 JSON 消息，其中包含有关 crate 是否包含可能在未来无法编译的任何代码的报告。

请注意，`--json` 参数与 [`--color`](#option-color) 参数组合是无效的，需要将 `--json` 与 `--error format=json` 组合。

更多细节请参阅 [JSON 章节][the JSON chapter]。

<a id="at-path"></a>
## `@path`: 从路径加载命令行标签

如果在命令行中指定 `@path`，则它将打开 `path` 并读取命令行选项。
这些选项每行一个；空行表示一个空选项。文件可以使用 `Unix` 或 `Windows` 样式的行尾，并且必须编码为 `UTF-8`。

[the JSON chapter]: json.md
