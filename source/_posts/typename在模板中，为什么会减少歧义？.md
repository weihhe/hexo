title: typename在模板中，为什么会减少歧义？
author: weihehe
date: 2025-10-13 17:17:37
tags:
---
更好的使用模板
<!--more-->

[代码出处](https://en.cppreference.com/w/cpp/language/dependent_name#The_typename_disambiguator_for_dependent_names)

简单来说，当我们引用依赖于模板参数的名字（dependent name）时，编译器无法提前知道它是类型名还是变量名，所以必须用 typename 来显式告诉编译器它是一个类型。

例如：

```cpp
#include <iostream>
#include <vector>
 
int p = 1;
template<typename T>
void foo(const std::vector<T> &v)
{
    std::vector<T>::const_iterator* p;
    // 这里 T 是模板参数，所以整个表达式依赖于 T

}
...
```
如上的情况，因为某个变量（p）声明依赖于模板参数(T),并且没有`typename`，所以编译器会将`const_iterator`当作一个变量，然后和`p`相乘。以下是相关的报错。

```cpp
E:\test\test.cpp:16:21: error: dependent-name 'std::vector<T>::const_iterator' is parsed as a non-type, but instantiation yields a type
   16 |     std::vector<T>::const_iterator* p;
      |                     ^~~~~~~~~~~~~~
E:\test\test.cpp:16:21: note: say 'typename std::vector<T>::const_iterator' if a type is meant
```

## 解决歧义的方法

### 加上 `typename`
```
typename std::vector<T>::const_iterator it = v.begin(); //明确告诉编译器，const_iterator是一个类型名

```
但是，不可单独只加上`typedef`，这样会导致错误：
```
E:\test\test.cpp:16:13: error: need 'typename' before 'std::vector<T>::const_iterator' because 'std::vector<T>' is a dependent scope [-Wtemplate-body]
   16 |     typedef std::vector<T>::const_iterator* p;
      |             ^~~
      |             typename 
```

```