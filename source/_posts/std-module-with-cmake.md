---
title: 如何在 CMake 项目中使用 std 模块
tags:
  - C++
  - C++23
  - CMake
  - clang
description: C++23 标准中引入了 Standard Library Modules，如何在 CMake 项目中使用 STD 模块呢？
date: 2025-10-03 19:36:12
---


CMake 在 3.30 版本后添加了对 C++23 引入的 Standard Library Modules 支持。本文参照官方说明，给出了一个简单的示例。

## 项目结构

项目结构比较简单，包含了两个子目录，exe 为可执行程序，uses_std 为动态库，在两个子模块中都使用了 `import std`，并在根目录的 CMakeLists 中链接。

```text
.
├── CMakeLists.txt
├── exe
│   ├── CMakeLists.txt
│   └── src
│       └── main.cpp
└── uses_std
    ├── CMakeLists.txt
    └── src
        └── uses_std.cpp
```

## 示例代码

uses_std.cpp

```cpp
import std;

void hello_world(std::string const &name)
{
    std::cout << "Hello World! My name is " << name << std::endl;
}
```

main.cpp

```cpp
import std;

void hello_world(std::string const &name);

int main(int argc, char *argv[])
{
    hello_world(argv[0] ? argv[0] : "Voldemort?");
    return 0;
}
```

代码很简单，不多赘述。

## CMakeLists.txt

在根目录的 CMakeLists.txt 中，主要做这几件工作：

1. 设置最小版本为 3.30
2. 打开 import std 的实验特性开关，这里具体要看CMake 对应版本源码下的 `Help/dev/experimental.rst` 中的说明，填写对应的 UUID。Fedora 42 中使用的 3.31.6 版本的值为 `0e5b6991-d74f-4b3d-a41c-cf096e0b2508`
3. 设置 C++ 标准为23，且将 `CMAKE_CXX_MODULE_STD` 设置为 `ON`
4. 添加子模块
5. 链接两个子模块

```cmake
cmake_minimum_required(VERSION 3.30)

# 1) import std 实验开关
set(CMAKE_EXPERIMENTAL_CXX_IMPORT_STD
    "0e5b6991-d74f-4b3d-a41c-cf096e0b2508")

# 2) 使用 clang+libc++
set(CMAKE_CXX_COMPILER "clang++")
set(CMAKE_CXX_FLAGS "-stdlib=libc++")

project(hello-cmake-module LANGUAGES CXX)

# 3) 统一标准，并打开扩展
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_MODULE_STD ON)

# 4) 子库
add_subdirectory(uses_std)

# 5) 可执行程序
add_subdirectory(exe)

# 6) 链接可执行程序与子库
target_link_libraries(exe PRIVATE
  uses_std
)
```

两个子模块的设置就很简单了

```cmake
aux_source_directory(src SRC_FILES)

add_executable(exe
  ${SRC_FILES}
)
```

```cmake
aux_source_directory(src SRC_FILES)
add_library(uses_std SHARED
    ${SRC_FILES}
)
```

注意要使用 Ninja

```bash
cmake -S . -B build -G Ninja
cmake --build build
```
