title: UDP服务实现 IP转换
author: weihehe
date: 2024-08-13 11:25:16
tags:
---
# UDP服务实现

## 准备创建套接字socket

- int socket(int domain, int type, int protocol);
	- domain:表明域，和加锁类型。
	- 套接字类型。
	- protocol协议类型。
    
创建一个套接字，类似于打开一个网络文件。

## 绑定端口号，IP

- 端口号在网络通信时是需要相互交换的，因为网络通信的本质。

- 云服务器禁止直接绑定公网IP。并且一般不会固定IP地址，一般将其置为`0`，即**任意地址绑定**，这样凡是发送到本主机的数据，都会根据端口号向上交付。
- 而客户端，一定要绑定的，只不过不需要用户显示的bind，一般由OS自由随机选择。
    
    
## 将需要交换的信息转化为网络序列


### 有关网络编程中IP地址处理的函数信息

| 函数                   | 原型                                          | 描述                                                 |
|------------------------|---------------------------------------------|------------------------------------------------------|
| `inet_aton`            | `int inet_aton(const char *cp, struct in_addr *inp);` | 将点分十进制的IP地址字符串转换为`in_addr`结构体。             |
| `inet_addr`            | `in_addr_t inet_addr(const char *cp);`      | 将点分十进制的IP地址字符串转换为`in_addr_t`类型的网络地址。      |
| `inet_network`         | `in_addr_t inet_network(const char *cp);`   | 将点分十进制的IP地址字符串转换为网络字节序的地址（不包括主机部分）。 |
| `inet_ntoa`            | `char *inet_ntoa(struct in_addr in);`       | 将`in_addr`结构体转换为点分十进制的IP地址字符串。              |
| `inet_makeaddr`        | `struct in_addr inet_makeaddr(in_addr_t net, in_addr_t host);` | 将网络号和主机号组合成一个`in_addr`结构体。                    |
| `inet_lnaof`           | `in_addr_t inet_lnaof(struct in_addr in);`  | 提取`in_addr`结构体中的主机部分。                              |
| `inet_netof`           | `in_addr_t inet_netof(struct in_addr in);`  | 提取`in_addr`结构体中的网络部分。                              |

## 接收数据


| 函数               | 原型                                                                                      | 描述                                                            |
|--------------------|-------------------------------------------------------------------------------------------|-----------------------------------------------------------------|
| `recv`             | `ssize_t recv(int sockfd, void *buf, size_t len, int flags);`                            | 从套接字`sockfd`中接收数据并存储到`buf`中，`len`指定缓冲区大小，`flags`控制接收操作。 |
| `recvfrom`         | `ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags, struct sockaddr *src_addr, socklen_t *addrlen);` | 从套接字`sockfd`中接收数据并存储到`buf`中，同时获取发送者的地址信息。`len`指定缓冲区大小，`flags`控制接收操作。 |
| `recvmsg`          | `ssize_t recvmsg(int sockfd, struct msghdr *msg, int flags);`                             | 从套接字`sockfd`中接收数据并存储到`msg`指定的缓冲区中，`flags`控制接收操作。`msg`结构体包含有关消息的详细信息。 |