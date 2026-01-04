title: modern-CPP：constexpr的条件判断（控制流）和模板
author: weihehe
date: 2025-11-07 16:29:33
tags:
---
阅读笔记
<!--more-->

[原文链接](https://changkun.de/modern-cpp/zh-cn/02-usability/#2-4-%E6%8E%A7%E5%88%B6%E6%B5%81)

## 控制流

`constexpr `关键字的存在，可以将表达式或者函数在**编译**时，就转为为函数结果，我们可以利用这一点，将其在条件判断当中使用。

例如：

![upload successful](/images/modern-控制流.png)

## 模板

**模板的哲学在于将一切能够在编译期处理的问题丢到编译期进行处理，仅在运行时处理那些最核心的动态服务**

- C++11 引入了外部模板，使我们能够显式的通知编译器何时进行模板的实例化,或者不进行实例化。

```cpp
template class std::vector<bool>;          // 强行实例化
extern template class std::vector<double>; // 不在该当前编译文件中实例化模板
```

### using与模板

对比以下两中方法，来定义函数指针
```cpp
// 使用typedef定义一个名为handler的函数指针

typedef void (*handler)(int, const char*);

//使用 using（更为推荐）：
using handler = void(*)(int, const char*);

// using还支持模板
using TrueDarkMagic = MagicType<std::vector<T>, std::string>;

```
### 变长参数模板

写法：
```cpp
template<typename... Ts> class Magic;
```

Magic 的对象，能够接受不受限制个数的 typename 作为模板的形式参数，例如下面的定义：

```cpp
class Magic<int,
            std::vector<int>,
            std::map<std::string,
            std::vector<int>>> darkMagic;
```
