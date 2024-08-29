title: Visual Studio2022
author: weihehe
tags: []
categories:
  - Environment
date: 2024-08-16 08:12:00
---
在使用过程中的一些记录
<!--more-->

1. #pragma warning( disable : `errornum`) 关闭警告，可以用于安全性问题。

2.`Win32`模式下，不存在`_WIN64`，只有`_WIN32`。而`x64`模式下，既有`_WIN64`，也有`_WIN32`。
- 可以通过调整`x64`环境下，它们的顺序来达到条件编译的目的。