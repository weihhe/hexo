title: Makefile CMake
author: weihehe
tags:
  - Makefile
categories:
  - C++
date: 2024-07-16 14:18:00
---
Linux项目自动化构建工具-make/Makefile
<!--more-->

## Makefile由来

一个工程中的源文件不计数，其按类型、功能、模块分别放在若干个目录中，makefile定义了一系列的规则来指定，哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，甚至于进行更复杂的功能操作。

makefile带来的好处就是——“自动化编译”，一旦写好，只需要一个make命令，整个工程完全自动编译，极大的提高了软件开发的效率。

`make`是一个命令工具，是一个解释makefile中指令的命令工具。两个搭配使用，完成项目自动化构建。

## 使用

在默认的方式下，我们只需要当前目录下输入`make`命令。 make会在当前目录下找名字 为`makefile`或`makefile`的文件：

```Makefile
hello: hello.o
	gcc hello.o -o hello

hello.o: hello.s
	gcc -c hello.s -o hello.o

hello.s: hello.i
	gcc -S hello.i -o hello.s

hello.i: hello.c
	gcc -E hello.c -o hello.i

.PHONY: clean
clean:
	rm -f hello.i hello.s hello.o hello

```
**可以使用`$@` `$^`来分别表示`:`的左右。**

## 生成多个可执行文件
```Makefile
.PHONY:all
all:otherExe otherfile

otherExe:otherExe.cpp
	g++ -o $@ @^ -std=c99
.PHONY:clean
clean:
	rm -f otherfile otherExe
```
用伪目标`all`来搭建依赖链。

## 项目清理

像上述中的clean，它没有被第一个目标文件直接或间接关联，那么它后面所定义的命令将不会被自动执行，不过，我们可以通过`make`执行。即命令——`make clean`，以此来清除所有的目标文件，以便于重新编译。

## 伪目标

- 总是被执行。
- 定义一些常用命令或管理任务，例如清理构建文件、生成文档、运行测试等。这些命令不会生成对应名字的文件，只是执行一些动作。

## 指定库和头文件，并且添加变量
```makefile
CFLAGS = -std=c++11 -I../../build/release-install-cpp11/include
LDFLAGS = -L../../build/release-install-cpp11/lib -lmuduo_base -lmuduo_net

server: server.cpp
    g++ $(CFLAGS) -o $@ $^ $(LDFLAGS)


```
- `CFLAGS`：指定了编译器的标志选项。其中，`-std=c++11` 是设置使用 C++11 标准，`-I../../build/release-install-cpp11/include` 是添加头文件搜索路径。
  - `LDFLAGS`：指定了链接器的标志选项。其中，`-L../../build/release-install-cpp11/lib` 是设置库文件的搜索路径，`-lmuduo_base` 和 `-lmuduo_net` 是需要链接的库。
- **目标规则**
  
  - `server: server.cpp`：定义了目标 `server`，它依赖于 `server.cpp` 文件。这意味着当 `server.cpp` 发生变化时，`server` 目标需要重新构建。
  - `g++ $(CFLAGS) -o $@ $^ $(LDFLAGS)`：这是构建 `server` 目标的命令。
    - `$(CFLAGS)` 插入了编译标志。
    - `-o $@` 指定了输出文件名，`$@` 表示目标文件，即 `server`。
    - `$^` 表示所有的依赖文件，在这里是 `server.cpp`。
    - `$(LDFLAGS)` 插入了链接标志。
## 原理:

- 在上述例子中，Makefile文件会`自顶向下`扫描。
- 如果`当前文件`的`依赖文件`文件不存在，那么make会在当前文件中找目标为`依赖文件`的文件。如果找不到则**再根据依赖方法来生成**,直到`自顶向下`的依赖链结束。

如果依赖文件比目标文件新，则目标需要重新生成。——make的依赖性：会一层又一层地去找文件的依赖关系，直到最终编译出第一个目标文件。

## CMake

`CMake`是一个跨平台的构建系统生成器，广泛用于管理 C/C++ 项目的构建过程。可以生成`Makefile`。

### 1. 安装 CMake
在大多数操作系统中，CMake 可以通过包管理器安装。

- **Ubuntu**:
  ```bash
  sudo apt-get install cmake
  ```

### 2. 创建 `CMakeLists.txt`
在你的项目根目录中创建一个 `CMakeLists.txt` 文件，这个文件包含构建项目所需的配置信息。

#### 基本结构
```cmake
# 设置 CMake 的最低版本要求
cmake_minimum_required(VERSION 3.10)

# 项目名称和版本号
project(MyProject VERSION 1.0)

# 指定 C++ 标准
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# 添加可执行文件
add_executable(MyExecutable main.cpp)
```

#### 添加多个源文件
```cmake
set(SOURCES
    main.cpp
    foo.cpp
    bar.cpp
)
add_executable(MyExecutable ${SOURCES})
```

#### 添加库
```cmake
# 添加静态或动态库
add_library(MyLibrary STATIC foo.cpp bar.cpp)

# 将库链接到可执行文件
target_link_libraries(MyExecutable PRIVATE MyLibrary)
```

#### 包含目录
```cmake
# 包含头文件目录
include_directories(${PROJECT_SOURCE_DIR}/include)
```

### 3. 生成构建系统
```bash
cmake ..
```

### 4. 常用指令和变量

- **设置变量**: 
  ```cmake
  set(VAR_NAME value)
  ```
  
- **选项开关**:
  ```cmake
  option(USE_FEATURE "Use special feature" ON)
  ```

- **检查库/文件/函数的存在**:
  ```cmake
  find_package(OpenCV REQUIRED)
  find_library(MY_LIB mylib)
  find_file(MY_HEADER myheader.h)
  ```

- **条件语句**:
  ```cmake
  if(USE_FEATURE)
    add_definitions(-DENABLE_FEATURE)
  endif()
  ```

### 5. 添加第三方库
如果需要使用第三方库，可以使用 `find_package` 或手动指定库的路径：

```cmake
find_package(Boost 1.36.0 REQUIRED)
include_directories(${Boost_INCLUDE_DIRS})
target_link_libraries(MyExecutable ${Boost_LIBRARIES})
```

### 6. CMake 编译类型
通过 `CMAKE_BUILD_TYPE` 设置编译类型：

```bash
cmake -DCMAKE_BUILD_TYPE=Release ..
```

常用的编译类型有：

- `Debug`: 包含调试信息
- `Release`: 优化代码
- `RelWithDebInfo`: 优化代码并包含调试信息

### 7. 其他功能

- **安装目标**: 
  ```cmake
  install(TARGETS MyExecutable DESTINATION bin)
  ```

- **测试**: 
  ```cmake
  enable_testing()
  add_test(NAME MyTest COMMAND MyExecutable)
  ```

### 8. 使用外部项目
如果你需要从外部下载并构建一个项目，可以使用 `ExternalProject` 模块：

```cmake
include(ExternalProject)

ExternalProject_Add(
  ext_project
  GIT_REPOSITORY https://github.com/example/repo.git
  PREFIX ${CMAKE_BINARY_DIR}/ext_project
  CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
)
```

