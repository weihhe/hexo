title: 进程
tags:
  - process
categories:
  - computer basics
author: weihehe
date: 2024-06-17 16:48:00
---
## 目录

- [引言](#引言)

- [第一部分](#第一部分)

- [1.1节](#11节)

- [1.2节](#12节)

- [第二部分](#第二部分)



## 进程

被装载到了**内存的程序**被称为进程，并使用PCB来对进程进行管理.

#### 构成
![upload successful](/images/pasted-0.png)


 **内核PCB数据结构对象**（进程属性的集合） + 代码 + 数据
 
## PCB（Process Control Block）

目的是为了方便操作系统去管理数据进程，将对进程的管理，转化为对PCB的管理。

 <ins>例如:task_struct是linux内核的一种用于管理进程的数据结构

#### 构成

任何一个程序，在加载到一个内存的时候，都要先创建结构体来管理它——即PCB（进程控制块）。，**PCB**中记录了许多有关运行进程的重要信息，包括进程的状态，进程运行的内存信息，优先级等等

![upload successful](/images/PCB.png)




#### 


