title: 文件IO 文件描述符 重定向
author: weihehe
tags:
  - 文件
  - IO
categories:
  - 操作系统
date: 2024-07-16 17:17:00
---
文件和进程的关系
<!--more-->

## 概念

- 文件 = 内容 + 属性。
- 文件被打开时，操作系统负责将文件内容加载到内存中，以供CPU调度和访问。
- 访问磁盘（外设）的过程就是访问硬件的过程。


## C语言

在C语言中，默认会打开三个输入输出流，分别是`stdin`,`stdout`, `stderr`，这三个流的类型都是`FILE*`。其中`FILE`是c语言对进程调用的文件信息进行的一层封装。

**`FILE *fopen(const char *filename, const char *mode);`**

### fopen() 
| Mode | Description |
|------|-------------|
| **r** | Open text file for reading. The stream is positioned at the beginning of the file. |
| **r+** | Open for reading and writing. The stream is positioned at the beginning of the file. |
| **w** | Truncate file to zero length or create text file for writing. The stream is positioned at the beginning of the file. |
| **w+** | Open for reading and writing. The file is created if it does not exist, otherwise it is truncated. The stream is positioned at the beginning of the file. |
| **a** | Open for appending (writing at end of file). The file is created if it does not exist. The stream is positioned at the end of the file. |
| **a+** | Open for reading and appending (writing at end of file). The file is created if it does not exist. The initial file position for reading is at the beginning of the file, but output is always appended to the end of the file. |


## 系统调用


![层级图](/images/操作系统层级图.png)

上述库函数也使用了系统调用。例如`c语言`中的`fopen()`就封装了`open()`。

我们还可以**直接采用系统接口**来进行文件访问文件。

例如：`open（）`

```c
#include <fcntl.h>
int open(const char *pathname, int flags, mode_t mode);
```
- pathname: 打开文件的名字。
- flags: 访问模式
- mode（可选）：创建新文件时使用的权限（如目标文件不存在，需要open创建，则第三个参数表示创建文件的默认权限,否则，使用两个参数的open。）。
	- mode参数权限(例如0666)

| Position | Octal | Binary | Meaning                |
|----------|-------|--------|------------------------|
| Special  | 0     | 000    | No special permissions |
| User     | 6     | 110    | Read + Write           |
| Group    | 6     | 110    | Read + Write           |
| Others   | 6     | 110    | Read + Write           |


- 返回值
```
RETURN VALUE
       open(),  openat(), and creat() return the new file descriptor (a nonnegative integer),
       or -1 if an error occurred (in which case, errno is set appropriately).

```

|**[标志位名称](https://weihehe.top/2024/07/13/%E7%BB%9D%E5%A6%99%E7%9A%84%E5%86%99%E6%B3%95/)** | 作用 |
|------|------|
|O_RDONLY| 只读打开
|O_WRONLY| 只写打开
| O_RDWR | 读，写,打开这三个常量，必须指定一个且只能指定一个
|O_CREAT | 若文件不存在，则创建它。需要使用mode选项，来指明新文件的访问权限
|O_APPEND|追加写

## `fd`和`FILE *`

**fd:文件描述符**

那这两者的返回值有什么关系呢？

在**进程**想要**访问磁盘文件**的时候，**内核**会生成一个**struct file结构体**，内部会包含文件各种各样的信息。每打开一个文件，就会生成一个。并用了类似双链表的形式将他们联系起来。

一个进程和它打开的文件使用`pcb`和`struct_files（文件描述符表）`联系起来


![进程的文件描述符表的运行机制](/images/pasted-17.png)

- **`fd`**是文件描述符表的**下标**，当`fd == -1`的时候，此时打开失败。

- 一个进程的`fd`分配规则：从0下标开始，寻找最小的没有使用的数组位置，它的下标就是新文件的文件描述符。

`操作系统`会默认打开以下文件便于用户使用。这时**操作系统的特性,而不是语言的特性**（Linux下一切皆文件）

- stdin:键盘文件`fd == 0`

- stdout:显示器文件`fd == 1`

- stdeer：显示器文件`fd == 2`

（可以使用类似`stdin->_fileno`来验证）

## `close（）` 和 `引用计数`

一个`struct_file`可能被**多个进程使用**，其中每多一个进程，`struct_file`中的`引用计数`就自增，反之自减。

当执行**关闭进程**的时候，我们只需要

- 根据提供的文件描述符，定位到文件描述符表的某个位置。
- 通过文件描述符表找到`struct_file`,并将`引用计数`自减。
- 在进程的文件描述符表中将文件描述符的位置置空。

## 重定向


- 原理：将进程的进程描述表中**需要重定向的位置上的地址**内容替换(复制新地址，粘贴到旧地址)为**重定向目标文件的地址**。

使用系统调用接口` int dup2(int oldfd, int newfd);`
