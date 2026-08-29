+++
title = 'Lito: C++ Module 构建系统和包管理'
date = 2026-08-26T23:06:46+08:00
draft = false
tags = ['编程']

+++

天下苦 cmake 久矣。  

写 C++ 代表我喜欢灵活，但不意味着构建系统是杂乱无章的。  
恰恰相反，C++ 的灵活需要配合做减法，而构建系统更应该做减法。

Lito，一个 module 优先的 C++ 包管理，同时也是构建系统。  
C++ 20 module 编写，整体参考 Cargo 设计，脚本部分由 lua 负责。  

Lito 的目标只有一个:   

**Make C++ simple and fun**.

Lito 的名字也是，lito/lite, 即我想更轻松的写 C++。  
项目地址：[litocpp/lito](https://github.com/litocpp/lito)，文档在 [lito.litocpp.org](https://lito.litocpp.org/)。

## CMake 的问题
cmake 除了语法杂糅，文档不说人话，接口黑盒外，最让我难受的地方是  

**cmake 几乎不对项目做假设**  

cmake 是有一套基于 target 的构建逻辑，但这是一个构建系统必须有的，除此之外它什么都不管。  
源码怎么组织，从哪里下载，放到哪里，如何编译，完全由项目自己负责，之后虽然添加了 FetchContent 支持，但也只是方便在 cmakelist 里做上述流程，cmake 自己并不做假设。  
支持 45 种编译器，[CMAKE_LANG_COMPILER_ID](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_COMPILER_ID.html)，这有点地狱笑话，想象一个支持 45 种编译器的库。  
没有自己的构建设施，反而通过 [cmake-generators](https://cmake.org/cmake/help/latest/manual/cmake-generators.7.html) 去支持 Makefile/Ninja/Visual Studio/...。  
没有假设包的概念，cmake 不是包管理  

**cmake 的设计逻辑是单个项目的构建系统**  

虽然 cmake 支撑了现有的大量 C/C++ 项目，但这些项目间都是割裂的。  
没有倾向，于是无差别地创死所有人。  
有着无与伦比的灵活性，但是配上了最烂的脚本语言。  

## Lito 的假设与设计

### Only Clang

这个决定是我最开始就定下的，这是 Lito 的基本假设。

### Module First

Lito 强制使用 C++20 以上的标准，并围绕 Modules 设计。

### Typed Compiler Flags

Only Clang 以后，Lito 会理解你的编译选项。  
常见的 Compiler Flags 会被解析成 typed option。  

Lito 默认不会偷偷继承当前 shell 里的 `CFLAGS`、`CXXFLAGS` 和 `LDFLAGS`。当然你也可以明确选择 `plain` profile，并通过 `--use-env-flags` 把环境变量变成本次构建的显式输入。  

### Builtin Frontend

源码本身带有大量构建信息，所以 Lito 会解析源码。

Lito 假设 library 和 runnable target 的默认入口分别是：

```text
src/lib.cppm
src/main.cppm
```

再从入口沿着实际生效的 `import` 发现 module closure。它不会因为某个 `.cppm` 恰好在 `src/` 下，就无条件编译整个目录。  
Lito 还提供 `LITO_PKG_VERSION`、`LITO_FEAT_XXX` 等宏。frontend 会记录哪些 source 真正查询了这些宏，再给对应的扫描和编译环境提供一致的值。  

### Manifest with Package

Lito 是包管理，而不只是构建工具。

### Lua Script
Lito 提供 `build.lua` 和 `install.lua`。  

- `build.lua` 用于声明生成源码和调用 build tool，例如 moc、rcc、protoc 或 esbuild
- `install.lua` 用于描述 target、runtime asset 和资源文件最终怎样落入 install prefix

Lito 也支持 script package，当前内置的 Qt script package 已经能够处理 moc、QML、resource、translation 和 protobuf 等常见流程。

### External Dependencies
Lito 支持 PkgConfig 和 CMake 外部依赖。  


## Module 和源码自发现

假设有一个根 module `geometry`：

```cpp
// src/lib.cppm
export module geometry;

export import :shape;
import geometry.logging;
```

Lito 会把 module logical name 映射到约定路径：

```text
geometry.logging      -> src/logging.cppm
geometry.render.image -> src/render/image.cppm
geometry:shape        -> src/shape.cppm
```

每个路径也可以使用目录入口，例如 `geometry.logging` 可以放在 `src/logging/mod.cppm`。  
如果 interface 旁边存在同名的 `src/logging.cpp`，它会作为 companion implementation unit，在 `geometry.logging` 进入 module closure 时一起编译。

一个位于 false `#if` 分支里的 `import` 不会进入 graph. 

如果 target 无法遵循约定，也可以显式写，主要是给 c package 留的支持：

```toml
[[bin]]
name = "asset-tool"
sources = ["tool/main.c", "tool/decoder.c"]
```

## 一个简单的例子

最小的 Lito 项目只有两个文件：

```text
hello/
├── lito.toml
└── src/
    └── main.cppm
```

`lito.toml`：

```toml
[package]
name = "hello"
version = "0.1.0"
standard = "c++23"

[[bin]]
name = "hello"
module = "hello"
```

`src/main.cppm`：

```cpp
export module hello;

import std;

auto main() -> int {
    std::println("Hello from Lito");
    return 0;
}
```

构建和运行：

```sh
lito build
./build/debug/bin/hello/hello
```
