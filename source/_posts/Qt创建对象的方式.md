title: Qt创建对象的方式
author: weihehe
date: 2025-06-23 17:51:38
tags:
---
Qt基础
<!--more-->

- 三种构建模式，如果之前配置了多个Qt套件，还需要先配置套件，再选择构建类型。

![filename already exists, renamed](/images/pasted-27.png)

## 使用form文件

1. 拖拽`控件`⾄UI设计界⾯中，并双击修改标签内容。

2. `qmake`在编译项目的时候，会基于这个内容，生成C++代码，通过C++代码构建出界面内容。`工程名\learn_1_autogen\include`目录下的文件。

## 使用代码

- 通常会将构建界面的代码，放到`Widget/MainWindow`的构造函数内

![filename already exists, renamed](/images/pasted-28.png)

- `this`在此处的作用是用于对象树。

- 默认生成的控件会出现在左上角