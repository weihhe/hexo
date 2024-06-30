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

任何一个程序，在加载到一个内存的时候，都要先创建结构体来管理它——即PCB（进程控制块）。**PCB**中记录了许多有关运行进程的重要信息，包括进程的状态，进程运行的内存信息，优先级等等

![upload successful](/images/PCB.png)


linux环境下使用ps命令实时监测pid

**替换process-name**

```shell
while : ; do ps ajx | head -1 && ps ajx | grep process-name | grep -v grep ; echo "---------------------" ; sleep 1 ; done

```
**grep在过滤时自身也会变成一个进程**，并且可能携带process-name信息，成为**ps**的一个干扰项

在操作系统内核中进程创建之后，我们需要通过系统调用接口 **getpid()** 才能获得进程的pid，并且只保证在本次运行期间pid有效

**PPID(parent process ID)**:也可以通过getppid()获取。

---

可以看到此时父进程的pid值不变

![upload successful](/images/ppid && pid.png)

查询该处的父进程

可以看到bash字段,即 **bash** 进程，即命令行解释进程。几乎所有shell指令都是它的子进程

![upload successful](/images/bash_pid.png)


#### 创建一个子进程

在linux下，可以通过**fork**函数在代码运行期间创建一个子进程，

```
Fork - RETURN VALUE
       On success, the PID of the child process is returned in the parent, and 0 is returned  in  the  child. 
       On failure, -1 is returned in the parent, no child process is created, and errno is set appropriately.
```

**fork的运行逻辑和使用**

- 由于父子进程的返回值不同，于是我们可以通过返回值来对父子进程进行管理

- 对于子进程而言只有一个父进程，所以只需要用‘0’来表示父进程即可，而对于父进程可能有不同的子进程，于是需要记录子进程的pid


