title: Makefile
author: weihehe
tags:
  - Makefile
categories:
  - C++
date: 2024-07-16 14:18:00
---
Linux项目自动化构建工具-make/Makefile
<!--more-->

## 由来

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

## 原理:

- 在上述例子中，Makefile文件会`自顶向下`扫描。
- 如果`当前文件`的`依赖文件`文件不存在，那么make会在当前文件中找目标为`依赖文件`的文件。如果找不到则**再根据依赖方法来生成**,直到`自顶向下`的依赖链结束。

如果依赖文件比目标文件新，则目标需要重新生成。——make的依赖性：会一层又一层地去找文件的依赖关系，直到最终编译出第一个目标文件。