# Linker-plugin-based LTO

`-C linker-plugin-lto` 标签允许将 LTO 优化推迟到实际链接步骤中，如果要链接的所有「目标文件 object files」都是基于 LLVM 工具链优化的，则此标签又允许跨编程语言边界执行过程间优化，本文示例主要展示如何将 Rust 代码与使用 Clang 编译的 C/C++ 代码链接在一起。

## Usage

链接器插件基于 LTO 的使用主要有两种情况：

 - 编译 Rust `staticlib` 用于 C ABI 的依赖项。
 - 在 `rustc` 调用链接器的地方编译 Rust 二进制文件。

在这两种情况下，Rust 代码需要用 `-C linker-plugin-lto` 编译，并且 c/c++ 代码使用 `-flto` 或 `-flto=thin` 以便将目标文件生成 LLVM 位码。

### Rust `staticlib` as dependency in C/C++ program

在这种情况下，Rust 编译器只需要确保 `staticlib` 中的目标文件格式正确即可。要想进行链接，必须使用带有 LLVM 插件的链接器（例如 LLD）。

直接使用 `rustc` ：

```bash
# Compile the Rust staticlib
rustc --crate-type=staticlib -Clinker-plugin-lto -Copt-level=2 ./lib.rs
# Compile the C code with `-flto=thin`
clang -c -O2 -flto=thin -o cmain.o ./cmain.c
# Link everything, making sure that we use an appropriate linker
clang -flto=thin -fuse-ld=lld -L . -l"name-of-your-rust-lib" -o main -O2 ./cmain.o
```

使用 `cargo`:

```bash
# Compile the Rust staticlib
RUSTFLAGS="-Clinker-plugin-lto" cargo build --release
# Compile the C code with `-flto=thin`
clang -c -O2 -flto=thin -o cmain.o ./cmain.c
# Link everything, making sure that we use an appropriate linker
clang -flto=thin -fuse-ld=lld -L . -l"name-of-your-rust-lib" -o main -O2 ./cmain.o
```

### C/C++ code as a dependency in Rust

在这种情况下，将由 `rustc` 调用链接器。我们再次必须确保使用合适的链接器。

直接使用 `rustc` ：

```bash
# Compile C code with `-flto`
clang ./clib.c -flto=thin -c -o ./clib.o -O2
# Create a static library from the C code
ar crus ./libxyz.a ./clib.o

# Invoke `rustc` with the additional arguments
rustc -Clinker-plugin-lto -L. -Copt-level=2 -Clinker=clang -Clink-arg=-fuse-ld=lld ./main.rs
```

直接使用 `cargo`：

```bash
# Compile C code with `-flto`
clang ./clib.c -flto=thin -c -o ./clib.o -O2
# Create a static library from the C code
ar crus ./libxyz.a ./clib.o

# Set the linking arguments via RUSTFLAGS
RUSTFLAGS="-Clinker-plugin-lto -Clinker=clang -Clink-arg=-fuse-ld=lld" cargo build --release
```

### Explicitly specifying the linker plugin to be used by `rustc`

如果想要使用 LLD 以外的链接器，需要明确指定使用的 LLVM 链接器插件。否则链接器无法读取目标文件。插件的路径通过作为 `-Clinker-plugin-lto` 选项的参数传递：

```bash
rustc -Clinker-plugin-lto="/path/to/LLVMgold.so" -L. -Copt-level=2 ./main.rs
```

### Usage with clang-cl and x86_64-pc-windows-msvc

Cross language LTO can be used with the x86_64-pc-windows-msvc target, but this requires using the
clang-cl compiler instead of the MSVC cl.exe included with Visual Studio Build Tools, and linking
with lld-link. Both clang-cl and lld-link can be downloaded from [LLVM's download page](https://releases.llvm.org/download.html).
Note that most crates in the ecosystem are likely to assume you are using cl.exe if using this target
and that some things, like for example vcpkg, [don't work very well with clang-cl](https://github.com/microsoft/vcpkg/issues/2087).

You will want to make sure your rust major LLVM version matches your installed LLVM tooling version,
otherwise it is likely you will get linker errors:

```bat
rustc -V --verbose
clang-cl --version
```

If you are compiling any proc-macros, you will get this error:

```bash
error: Linker plugin based LTO is not supported together with `-C prefer-dynamic` when
targeting Windows-like targets
```

This is fixed if you explicitly set the target, for example
`cargo build --target x86_64-pc-windows-msvc`
Without an explicit --target the flags will be passed to all compiler invocations (including build
scripts and proc macros), see [cargo docs on rustflags](../cargo/reference/config.html#buildrustflags)

If you have dependencies using the `cc` crate, you will need to set these
environment variables:
```bat
set CC=clang-cl
set CXX=clang-cl
set CFLAGS=/clang:-flto=thin /clang:-fuse-ld=lld-link
set CXXFLAGS=/clang:-flto=thin /clang:-fuse-ld=lld-link
REM Needed because msvc's lib.exe crashes on LLVM LTO .obj files
set AR=llvm-lib
```

If you are specifying lld-link as your linker by setting `linker = "lld-link.exe"` in your cargo config,
you may run into issues with some crates that compile code with separate cargo invocations. You should be
able to get around this problem by setting `-Clinker=lld-link` in RUSTFLAGS

## Toolchain Compatibility

<!-- NOTE: to update the below table, you can use this Python script:

```python
from collections import defaultdict
import subprocess

def minor_version(version):
    return int(version.split('.')[1])

INSTALL_TOOLCHAIN = ["rustup", "toolchain", "install", "--profile", "minimal"]
subprocess.run(INSTALL_TOOLCHAIN + ["nightly"])

LOWER_BOUND = 65
NIGHTLY_VERSION = minor_version(subprocess.run(
    ["rustc", "+nightly", "--version"],
    capture_output=True,
    text=True).stdout)

def llvm_version(toolchain):
    version_text = subprocess.run(
        ["rustc", "+{}".format(toolchain), "-Vv"],
        capture_output=True,
        text=True).stdout
    return int(version_text.split("LLVM")[1].split(':')[1].split('.')[0])

version_map = defaultdict(lambda: [])
for version in range(LOWER_BOUND, NIGHTLY_VERSION - 1):
    toolchain = "1.{}.0".format(version)
    subprocess.run(
        INSTALL_TOOLCHAIN + ["--no-self-update", toolchain],
        capture_output=True)
    version_map[llvm_version(toolchain)].append(version)

print("| Rust Version | Clang Version |")
print("|--------------|---------------|")
for clang, rust in sorted(version_map.items()):
    if len(rust) > 1:
        rust_range = "1.{} - 1.{}".format(rust[0], rust[-1])
    else:
        rust_range = "1.{}       ".format(rust[0])
    print("| {}  |      {}       |".format(rust_range, clang))
```

-->

为了使这种 LTO 生效，LLVM 链接器插件必须能够处理由 `rustc` 和 `clang` 产生的 LLVM 位码。

通过使用基于相同版本的 LLVM 的 `rustc` 和 `clang` 实现可获得最佳结果。可以使用 `rustc -vV` 来查看给定 `rustc` 版本所使用的 LLVM。注意因为Rust有时会使用 LLVM 的不稳定版本，所以此处所给出的版本号只是一个近似值，但是这种近似通常是可靠的。

下表展示了已知工具链的良好组合。

| Rust Version | Clang Version |
|--------------|---------------|
| 1.34 - 1.37  |       8       |
| 1.38 - 1.44  |       9       |
| 1.45 - 1.46  |      10       |
| 1.47 - 1.51  |      11       |
| 1.52 - 1.55  |      12       |
| 1.56 - 1.59  |      13       |
| 1.60 - 1.64  |      14       |
| 1.65         |      15       |

注意，此处的兼容性策略将来可能会更改。
