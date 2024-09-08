title: C++中的类型转换 RTTI
author: weihehe
tags:
  - 类型
categories:
  - C++
date: 2024-08-10 22:09:00
---
四种类型转换
<!--more-->

| 转换类型         | 描述                                                                 | 用法示例                               |
|------------------|----------------------------------------------------------------------|----------------------------------------|
| `static_cast`    | 执行编译时的普通类型转换。用于大多数类型转换，如基本数据类型、指针、引用等。 | `int i = 10; double d = static_cast<double>(i);` |
| `dynamic_cast`   | 执行运行时的类型安全转换。用于多态类型（具有虚函数的类）之间的转换。 | `Base* b = new Derived(); Derived* d = dynamic_cast<Derived*>(b);` |
| `const_cast`     | 添加或去除对象的 `const` 或 `volatile` 限定符。                    | `const int* c = &i; int* p = const_cast<int*>(c);` |
| `reinterpret_cast` | 执行低级别的位模式转换，通常用于指针类型。                      | `long l = reinterpret_cast<long>(p);` |

# dynamic_cast用于将一个父类对象的指针/引用转换为子类对象的指针或引用(动态转换)

-  dynamic_cast会先检查是否能转换成功，能成功则转换，不能则返回0
- 向上转型：子类对象指针/引用->父类指针/引用(不需要转换，赋值兼容规则)
- 向下转型：父类对象指针/引用->子类指针/引用(用dynamic_cast转型是安全的)



# RTTI：Run-time Type identification的简称，即：运行时类型识别。
- C++通过以下方式来支持RTTI：
	1. typeid运算符
	2. dynamic_cast运算符
	3. decltype