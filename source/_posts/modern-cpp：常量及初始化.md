title: 'modern-cpp：常量 变量及其{}初始化 类型推导'
author: weihehe
tags:
  - 现代C++
categories:
  - C++
date: 2025-10-13 14:06:00
---
高速上手C++11/14/17/20阅读笔记
<!--more-->

[书籍——原文链接](https://changkun.de/modern-cpp/zh-cn/02-usability/#constexpr)

## 常量

#### nullptr

它能被隐式转换成任何指针类型的空指针值，或 C++ 中的任何成员指针类型的空成员指针值。用来代替C中`NULL`空指针，避免歧义。

#### constexpr（C++11）

常量表达式 —— 总是会产生相同结果且无副作用的表达式。C++ 标准中数组的长度**必须是一个常量表达式**，即使是一个`const`常数也不行。
- 修饰的函数可以使用递归。
-  C++14 开始，constexpr 函数可以在内部使用局部变量、循环和分支等简单语句

```
    constexpr int len_2_constexpr = 1 + 2 + 3;
    
    constexpr int fibonacci(const int n) {
    return n == 1 || n == 2 ? 1 : fibonacci(n-		1) + fibonacci(n-2);
}

```

## 变量初始化

#### `if`和`switch`语句当中，支持临时变量（c++17）

#### 初始化列表`{}`

C++11 引入了 花括号`{}` —— 的初始化语法，使得几乎所有对象（包括内置类型、自定义类型、容器等）都能用统一的形式来初始化：

```cpp
public:
    void foo(std::initializer_list<int> list) {
        for (std::initializer_list<int>::iterator it = list.begin();
            it != list.end(); ++it) vec.push_back(*it);
    }

magicFoo.foo({6,7,8,9});
Foo foo2 {3, 4};
相当于 Foo foo2 = {3, 4};
```
#### 结构化绑定（C++11,C++17）

`std::tuple`(c++11)容器用于构造一个元组，进而囊括多个返回值。但是`C++11/14` 并没有提供一种简单的方法直接从元组中拿到并定义元组中的元素，管我们可以使用 std::tie 对元组进行拆包，但我们依然必须非常清楚这个元组包含多少个对象，各个对象是什么类型，非常麻烦。
C++17 完善了这一设定
```cpp
#include <iostream>
#include <tuple>

std::tuple<int, double, std::string> f() {
    return std::make_tuple(1, 2.3, "456");
}

int main() {
    auto [x, y, z] = f();
    std::cout << x << ", " << y << ", " << z << std::endl;
    return 0;
}
```
#### `auto`，`decltype`

在最新的`C++14`和`C++20`中，`auto`可在函数传参时使用，不过`auto`仍不支持推导**数组类型**。

`decltype` 关键字是为了解决 auto 关键字只能对变量进行类型推导的缺陷而出现的，和`typeof`很类似，不过`typeof`并不属于C++标准内，`decltype` 可以对关键字进行推导。

```cpp
decltype(表达式)
```
`bool std::is_same<T, U>` 用于判断` T `和` U `这两个类型是否相等。

#### 尾推导类型（C++11）

- `auto`可以用于推导函数的返回类型，同时还可以配合`decltype`来使用(C++14),主要用于对转发函数或封装的返回类型进行推导。使我们无需显式的指定类型，从而写出如下的代码
```cpp

decltype(auto) look_up_a_string_1() {
    return lookup1();
}

```
