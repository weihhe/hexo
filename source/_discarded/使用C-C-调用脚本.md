title: 使用C/C++调用脚本(python)
author: weihehe
tags: []
categories: []
date: 2024-07-16 16:11:00
---

### 脚本语言

- 所有的脚本语言都要以`#!`开头，后面跟上**解释器**的路径。由**解释器**逐行来解释我们的**脚本语言的内容**。

*shell脚本中的函数调用不需要圆括号*

### 调用

例如：
- 使用`execle()`调用以下python程序`：

execle("/usr/bin/python3","python","test.sh,NULL)；
```python
#!/usr/bin/python3
print("hello cpp&python")
```