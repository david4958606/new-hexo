---
title: 如何在三大编译器中使用 STD 模块
tags:
  - C++
  - C++23
  - MSVC
  - GCC
  - Clang
description: C++23 标准中引入了 Standard Library Modules，如何在三大编译器中使用 STD 模块呢？
date: 2025-02-03 22:44:02
---


## C++ 23 中的 Standard Library Modules

自 C++ 20 开始，C++ 标准委员会就开始引入模块化的概念，C++ 23 标准中引入了 Standard Library Modules，可以在代码中直接引入标准库模块，而不再需要使用 `#include` 来引入头文件。

比如如下的代码：

```cpp
import std;

int main() {
    std::println("Hello, World!");
    return 0;
}
```

这里直接使用了 `std` 模块，而不再需要 `#include <print>`（`<print>` 也是C++ 23 的新特性）。

## MSVC 中使用 STD 模块

Visual Studio 2022 开始支持 C++ 23 标准，需要在项目属性中手动开启模块支持，之后便可以直接`import std;`了。

## GCC 中使用 STD 模块

GCC 将在 15 版本后支持 Standard Library Modules， 在第一次编译时需要用：

```bash
gcc -std=c++23 -fmodules -fsearch-include-path bits/std.cc file.cpp
```

来得到`gcm.cache/std.gcm`文件，之后就可以用：

```bash
gcc -std=c++23 -fmodules file.cpp
```

编译文件了（gcc 15 中将把 `-fmodules-ts` 改为 `-fmodules`）。

## Clang 中使用 STD 模块

GCC 15 版本还没正式发布，但是 Clang 自 17 版本之后已经可以使用 STD 模块了，同时需要libc++。

```bash
clang++ -std=c++23 -stdlib=libc++ \
    -Wno-reserved-identifier -Wno-reserved-module-identifier \
    --precompile -o std.pcm /usr/share/libc++/v1/std.cppm
```

来预编译 libc++ 的 std 模块，之后就可以用：

```bash
clang++ -std=c++23 -stdlib=libc++ \
    -fmodule-file=std=std.pcm -o test std.pcm test.cpp
```

来编译文件了。

### clangd 设置

如果使用 clangd，需要在配置文件中加入：

```yml
# .clangd
CompileFlags:
  Add: [-std=c++23, -stdlib=libc++, -fmodule-file=std=std.pcm]
  Compiler: clang++
```

## 参考：

- [Using the C++23 std Module with Clang 18](https://0xstubs.org/using-the-c23-std-module-with-clang-18/)
- [How to use module std with gcc](https://stackoverflow.com/questions/76154680/how-to-use-module-std-with-gcc)
