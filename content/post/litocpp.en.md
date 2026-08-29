+++
title = 'Lito: A C++ Modules Build System and Package Manager'
date = 2026-08-26T23:06:46+08:00
draft = false
tags = ['Coding']

+++

The world has suffered CMake for far too long.  

Writing C++ means I like flexibility, but that does not mean the build system should be chaotic.  
Quite the opposite: the flexibility of C++ calls for subtraction, and its build system needs subtraction even more.

Lito is a module-first package manager for C++, and also a build system.  
It is written with C++20 Modules, takes its overall design inspiration from Cargo, and uses lua for scripting.  

Lito has only one goal:  

**Make C++ simple and fun**.

The name Lito follows the same idea: lito/lite, because I want writing C++ to feel easier.  
Project: [litocpp/lito](https://github.com/litocpp/lito). Documentation: [lito.litocpp.org](https://lito.litocpp.org/).

## The Problem with CMake

Aside from its mishmash of syntax, human-unfriendly documentation, and black-box interfaces, what bothers me most about CMake is this:

**CMake makes almost no assumptions about a project**

CMake does have a target-based build model, but that is something every build system must have. Beyond that, it takes responsibility for nothing.  
How source code is organized, where it is downloaded from, where it is placed, and how it is compiled are all left to the project. FetchContent support was added later, but it only makes it easier to implement those operations in a CMakeLists file; CMake itself still makes no assumptions.  
It supports 45 compilers through [CMAKE_LANG_COMPILER_ID](https://cmake.org/cmake/help/latest/variable/CMAKE_LANG_COMPILER_ID.html). That is a bit of a dark joke: imagine a library that supports 45 compilers.  
It has no build facility of its own, instead relying on [CMake generators](https://cmake.org/cmake/help/latest/manual/cmake-generators.7.html) to support Makefile, Ninja, Visual Studio, and more.  
It makes no assumptions about packages. CMake is not a package manager.

**CMake is designed as a build system for a single project**

Although CMake supports a vast number of existing C and C++ projects, those projects remain isolated from one another.  
It takes no side, so it runs everyone over indiscriminately.  
Unmatched flexibility, paired with the worst scripting language.  

## Lito's Assumptions and Design

### Only Clang

I made this decision at the very beginning. It is Lito's fundamental assumption.

### Module First

Lito requires C++20 or newer and is designed around Modules.

### Typed Compiler Flags

Once Lito only needs to support Clang, it can understand your compiler options.  
Common compiler flags are parsed into typed options.  

Lito does not quietly inherit `CFLAGS`, `CXXFLAGS`, or `LDFLAGS` from the current shell. Of course, you can explicitly select the `plain` profile and use `--use-env-flags` to make environment variables explicit inputs to the current build.  

### Builtin Frontend

Source code itself contains a great deal of build information, so Lito parses the source.

Lito assumes the default entries for library and runnable targets are:

```text
src/lib.cppm
src/main.cppm
```

Starting from the entry, Lito follows active `import` declarations to discover the module closure. It does not compile every `.cppm` under `src/` merely because the file happens to be there.  
Lito also provides macros such as `LITO_PKG_VERSION` and `LITO_FEAT_XXX`. The frontend records which sources actually query these macros, then supplies consistent values to the corresponding scan and compile environments.  

### Manifest with Package

Lito is a package manager, not merely a build tool.

### Lua Script
Lito provides `build.lua` and `install.lua`.  

- `build.lua` declares generated sources and invokes build tools such as moc, rcc, protoc, or esbuild
- `install.lua` describes how targets, runtime assets, and resource files are placed under the install prefix

Lito also supports script packages. Its builtin Qt script package can already handle common workflows involving moc, QML, resources, translations, and protobuf.

### External Dependencies
Lito supports external dependencies provided through PkgConfig and CMake.  


## Modules and Source Discovery

Suppose there is a root module named `geometry`:

```cpp
// src/lib.cppm
export module geometry;

export import :shape;
import geometry.logging;
```

Lito maps logical module names to conventional paths:

```text
geometry.logging      -> src/logging.cppm
geometry.render.image -> src/render/image.cppm
geometry:shape        -> src/shape.cppm
```

Each path may also use a directory entry; for example, `geometry.logging` can live at `src/logging/mod.cppm`.  
If a matching `src/logging.cpp` exists beside the interface, Lito treats it as a companion implementation unit and compiles it when `geometry.logging` enters the module closure.

An `import` inside a false `#if` branch does not enter the graph.

If a target cannot follow these conventions, its sources can be listed explicitly. This mainly exists to support C packages:

```toml
[[bin]]
name = "asset-tool"
sources = ["tool/main.c", "tool/decoder.c"]
```

## A Simple Example

The smallest Lito project contains only two files:

```text
hello/
├── lito.toml
└── src/
    └── main.cppm
```

`lito.toml`:

```toml
[package]
name = "hello"
version = "0.1.0"
standard = "c++23"

[[bin]]
name = "hello"
module = "hello"
```

`src/main.cppm`:

```cpp
export module hello;

import std;

auto main() -> int {
    std::println("Hello from Lito");
    return 0;
}
```

Build and run:

```sh
lito build
./build/debug/bin/hello/hello
```
