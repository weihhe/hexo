title: SWIG
author: weihehe
date: 2025-02-24 15:32:59
tags:
---
Simplified Wrapper and Interface Generator
<!--more-->

[原文链接](https://www.yuque.com/softdev/swig/nuaxps)

**SIWG生成中间层，调用C++代码**

![upload successful](/images/SWIG.png)


## 一些函数

1. **__declspec(dllexport)**，声明函数为 DLL 的导出函数，使得外部程序（如 C#）可以通过动态链接库调用这些函数。

2. **__stdcall**函数，指定函数的调用约定（Calling Convention），确保 C# 与 C++ 之间的参数传递和栈清理规则一致。