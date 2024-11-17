---
title: 使用 vcpkg 引入 Google Test 框架以及对应 CMake 配置写法
tags: [C++, cmake]
---

## 关于 vcpkg

vcpkg 是微软官方推出的 C/C++ 依赖管理系统，原生支持多平台，可以与 CMake 或 MSBuild 集成。

vcpkg 支持全局安装以及分项目配置的清单模式。微软官方推荐后者。

与 CMake 集成时，仅需为 CMake 指定环境变量 `"CMAKE_TOOLCHAIN_FILE": "$env{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake"`。

更多 vcpkg 用法，请参考 [Microsoft Learn 网站](https://learn.microsoft.com/zh-cn/vcpkg/)，本文不做赘述。

## 关于 Google Test

Google Test 是谷歌开发的 C++ 单元测试框架，现已有很多成熟教程。本文也不再赘述 gtest 的使用。

## 假定的项目结构

```bash
\home\Documents\Projects\Cpp\HelloGtest
├── build
├── src
│   ├──CMakeLists.txt
│   ├──*.cpp
│   ├──*.h
├── test
│   ├──CMakeLists.txt
│   ├──TestAll.cpp
├── main.cpp
├──CMakeLists.txt
```

`src` 存放主要的 C++ 代码，由 `src\CMakeLists.txt` 编译为静态库。

`test` 存放编写的测试代码，由 `test\CMakeLists.txt` 编译为可执行文件。

根目录中有 `main.cpp` 作为程序入口，并由 `CMakeLists.txt` 与静态库相链接。

## 各个 `CMakeLists.txt` 写法

src\CMakeLists.txt

```bash
# 设置源文件目录
file(GLOB_RECURSE SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/*.cpp")
file(GLOB_RECURSE HEADERS "${CMAKE_CURRENT_SOURCE_DIR}/*.h")

# 将文件打包为库
add_library(NumericalRecipesLibs STATIC ${SOURCES} ${HEADERS})
```

test\CMakeLists.txt

```bash
# 引入 Google Test
find_package(GTest CONFIG REQUIRED)
# 创建一个单一的可执行文件，将所有测试源文件包含进去
set(TEST_SOURCES TestAll.cpp)
add_executable(AllTests ${TEST_SOURCES})
# 设置包含路径
target_include_directories(AllTests PRIVATE ${CMAKE_SOURCE_DIR}/src)
# 链接 GTest 和其他必要的库
target_link_libraries(AllTests PRIVATE GTest::gtest_main NumericalRecipesLibs)
# 启用测试
enable_testing()
# 将单一的可执行文件作为一个测试添加
add_test(NAME AllTests COMMAND AllTests)
```

\CMakeLists.txt

```bash
cmake_minimum_required(VERSION 3.20)
set(CMAKE_TOOLCHAIN_FILE "C:\\Programming\\Cpp\\vcpkg\\scripts\\buildsystems\\vcpkg.cmake")
# 设置项目名称和语言
project(NumericalRecipes LANGUAGES CXX)
# 设置C++标准为C++20
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
# 添加子目录
add_subdirectory(src)
add_subdirectory(test)
# 添加main.cpp可执行目标
add_executable(NumericalRecipes main.cpp)
# 链接静态库和其他依赖
# 链接NumericalRecipes静态库到MainExecutable
# 确保在 src/CMakeLists.txt 中创建的库名称是正确的
# 使用target_link_libraries进行链接

# 链接src中的静态库NumericalRecipes到主可执行文件
target_link_libraries(NumericalRecipes PRIVATE NumericalRecipesLibs)
```

## 运行测试

这样，可以直接在 Clion 中运行对应的 `TEST()` 了。

如果需要使用命令行：

```bash
cd build
cmake ..
cmake --build . --config Debug # 或 Release，需要与后面的 ctest -C 对应
ctest ctest --verbose -C Debug # 使用 --verbose 查看详情
```
