title: EffectiveModernCpp 条款1：理解模板类型的推导
author: weihehe
date: 2025-10-11 11:09:46
tags:
---
Item 1: Understand template type deduction
<!--more-->
模板类型中类型的推导不总是按我们预取的那样来进行的。举例伪代码如下：
```cpp
template<typename T>
void f(ParamType param);
```
我们使用以下办法去调用：
```cpp
f(expr); 
```
那么在编译的时候，expr会参考**两部分**进行推导，一部分是`T`，另一部分则是`ParamType`。而且`ParamType`可能包含一些修饰符，例如`Const，&(引用)`，导致最后推导的结果反直觉。
当出现以下三种情况时，这种反直觉会更为明显：

1. 