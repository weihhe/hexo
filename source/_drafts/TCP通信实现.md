title: TCP通信实现
author: weihehe
date: 2024-08-16 11:05:52
tags:
---

Tcp协议实现
<!--more-->

Tcp是面向连接的，因此服务器一般是比较“被动的”，即持续监听。

```cpp
#include <sys/types.h>          #include <sys/socket.h>

int listen(int sockfd, int backlog);
```