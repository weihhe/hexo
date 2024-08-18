title: Linux守护进程
author: weihehe
tags:
  - 进程
categories:
  - 操作系统
date: 2024-08-18 00:25:00
---

一种进程状态
<!--more-->

## 概念

- 每位用户登录时，我们都要创建一个`session`，一个`bash`
	- 为了管理`session`，系统提供了一个`SID——session ID`字段。
	- 并且`SID`的值是当前用户登录时，创建的`bash`的`PID`值。

- `PGID`——进程组ID。
	- 组长的`PID`作为组内所有成员的`PGID`。
    
- 进程组和任务(`jobs`)的关系 

	- 任务代表具体的业务工作，而**任务是需要指派给进程组的**。
    
	- 多个任务 ，在同一个`session`内启动,则它们的`sid`是相同的。

## 守护进程 —— 不受用户登录和注销的影响

图中随着`bash`被关闭——用户注销，其所属的任务也被关闭了。

![守护进程](/images/守护进程2.png)    
    
为了避免这样的情况，我们需要将其变为**自成会话的进程，即——守护进程**。

`setsid` 函数是一个在 Unix/Linux 系统中用于创建新会话并使调用进程成为该会话的领导者的函数。它通常用于创建守护进程（daemon），使进程脱离终端并在后台运行。

### 函数原型
```c
#include <unistd.h>

pid_t setsid(void);
```

### 功能描述

- **创建新会话**:如果**调用的进程不是一个进程组的组长** `setsid` 函数创建一个新的会话，调用的进程将成为新的进程组的组长

- **脱离控制终端**: 调用 `setsid` 的进程会脱离原来的控制终端，并且新会话不会有控制终端——自成一家。

- **更改进程组 ID**: 调用 `setsid` 后，进程的进程组 ID（PGID）会被设置为与进程 ID（PID）相同，形成新的进程组。

### 返回值

- **成功**: 返回调用进程的会话 `ID`。

- **失败**: 返回 `-1`，并设置 `errno` 以指示错误原因。

### 实验代码
```c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>

int main() {
    pid_t pid = fork();  // 创建子进程
    if (pid < 0) {
        perror("fork failed");
        exit(1);
    }

    if (pid > 0) {
        // 父进程退出
        exit(0);
    }

    // 子进程继续
    if (setsid() < 0) {
        perror("setsid failed");
        exit(1);
    }

    // 在此之后，子进程成为新会话的领导者，并且没有控制终端
    printf("New session created, session ID: %d\n", getpid());

    // 执行守护进程的其他操作...

    while (1) {
        sleep(1);  // 模拟守护进程的工作
    }

    return 0;
}
```

### 应用场景

- **创建守护进程**: 守护进程在启动时通常会调用 `setsid` 来脱离控制终端，从而独立运行。

- **创建新会话**: 在某些需要创建新会话的场景下，`setsid` 可用于分离进程和控制终端，使其不受控制终端的影响。

     
    
    
