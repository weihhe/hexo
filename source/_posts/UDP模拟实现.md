title: 'UDP通信实现 '
author: weihehe
tags:
  - UDP
categories:
  - 网络
date: 2024-08-13 11:25:00
---
UDP实现客户端和服务端之间的小写转换
<!--more-->
# UDP服务实现


### 常用函数


1. **`socket()`**:
   - **创建套接字**：
     ```cpp
     int sockfd = socket(int domain, int type, int protocol);
     ```
     - `domain`：指定协议族，如 `AF_INET`（IPv4）或 `AF_INET6`（IPv6）。
     - `type`：指定套接字类型，如 `SOCK_STREAM`（TCP）或 `SOCK_DGRAM`（UDP）。
     - `protocol`：指定传输协议，通常设为0以选择默认协议。
     - `sockfd`：创建成功后返回的套接字文件描述符，用于后续的网络操作。

2. **`bind()`**:
   - **绑定地址**：
     ```cpp
     int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
     ```
     - `sockfd`：要绑定的套接字。
     - `addr`：指向 `sockaddr` 结构的指针，指定要绑定的本地地址和端口。
     - `addrlen`：地址结构的大小。

3. **`sendto()`**:
   - **发送数据（UDP）**：
     ```cpp
     ssize_t sendto(int sockfd, const void *msg, size_t len, int flags, const struct sockaddr *dest_addr, socklen_t addrlen);
     ```
     - `sockfd`：发送数据的套接字。
     - `msg`：指向要发送的数据的指针。
     - `len`：要发送的数据长度。
     - `flags`：发送标志，通常设置为0。
     - `dest_addr`：指向目标地址的指针。
     - `addrlen`：目标地址的长度。

4. **`recvfrom()`**:
   - **接收数据（UDP）**：
     ```cpp
     ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags, struct sockaddr *src_addr, socklen_t *addrlen);
     ```
     - `sockfd`：接收数据的套接字。
     - `buf`：指向存储接收到数据的缓冲区的指针。
     - `len`：缓冲区的长度。
     - `flags`：接收标志，通常设置为0。
     - `src_addr`：指向存储源地址的结构的指针。
     - `addrlen`：源地址结构的长度。
    
## 准备创建套接字socket

在网络编程中，`sockfd` 是一个用于标识套接字的文件描述符。它是 `socket()` 函数创建的套接字的返回值，用于后续的网络通信操作。`sockfd` 是一个整数值，作为其他网络操作函数的参数之一。

## 绑定端口号，IP

- 端口号在网络通信时是需要相互交换的，因为网络通信的本质。

- 云服务器禁止直接绑定公网IP。并且一般不会固定IP地址，一般将其置为`0`，即**任意地址绑定**，这样凡是发送到本主机的数据，都会根据端口号向上交付。

- 而客户端，也是要绑定的，只不过不需要用户显示的bind，一般由OS自由随机选择绑定，只需要保证其在客户端主机上的唯一性就行。
	
    
    
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


UDP（用户数据报协议）通信的基本原理涉及以下几个方面：

### 1. **基础概念**
- **UDP** 是一种无连接的、面向数据报的协议，属于传输层协议。
- 它在网络上发送数据包（即数据报），每个数据包都独立处理，没有建立连接的过程。
- UDP提供了一种快速且简单的方式来发送数据，但它不保证数据包的顺序、可靠性或数据包的完整性。

### 2. **工作原理**

#### **服务端（Server）**：
1. **创建Socket**：
   - 通过调用 `socket()` 函数创建一个UDP socket。该函数返回一个套接字描述符。

2. **绑定（Bind）**：
   - 使用 `bind()` 函数将套接字与本地地址（IP和端口号）关联起来。这使得操作系统知道当数据包到达时应该将其交给哪个程序。

3. **接收数据（Receive）**：
   - 调用 `recvfrom()` 函数接收数据。接收到的数据会被存储在缓冲区中，同时函数会提供发送方的地址信息。

4. **发送数据（Send）**：
   - 使用 `sendto()` 函数向指定的地址（客户端的IP和端口）发送数据。数据通过网络传输到目标地址。

5. **关闭Socket**：
   - 使用 `close()` 函数关闭套接字，释放相关资源。

#### **客户端（Client）**：
1. **创建Socket**：
   - 通过调用 `socket()` 函数创建一个UDP socket。

2. **构造目标地址**：
   - 配置目标地址（服务器的IP和端口号），用于发送数据。

3. **发送数据（Send）**：
   - 使用 `sendto()` 函数将数据发送到目标地址。

4. **接收数据（Receive）**：
   - 调用 `recvfrom()` 函数接收从服务器发回的数据。接收到的数据会被存储在缓冲区中。

5. **关闭Socket**：
   - 使用 `close()` 函数关闭套接字，释放相关资源。

## 实验代码

```cpp
/*Main.cc*/
#include "UdpServer.hpp"
#include <memory>
#include <cctype>
#include <string>


void Usage(std::string proc)
{
    std::cout<<"Usage:"<<proc<<"请输入端口号\n"<<std::endl;
}

std::string deal(std::string &str)
{
    for(int i = 0; i < str.length();i++)
    {
        if(str[i] <= 'Z' && str[i] >= 'A')
        {
            str[i] += 32; 
        }
    }
    return str;
}

int main(int argc , char *argv[])
{
    if(argc != 2)
    {
        Usage(argv[0]);
        exit(0);
    }

    uint16_t port = std::stoi(argv[1]);
    //创建对象
    std::unique_ptr<UdpServer> svr(new UdpServer(port));
    svr->init();
    std::function<std::string(std::string&)> func = deal;
    svr->run(func);
    return 0;
}
```
--- 
```cpp
/*Udpclient.cc*/
#include <sys/types.h> /* See NOTES */
#include <sys/socket.h>
#include <iostream>
#include <unistd.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <cstring>

char buffer[1024];

void usage(std::string proc)
{
    std::cout<<"请重新输入"<<proc<<" Ip 端口号"<<std::endl;
}

int main(int argc,char *argv[])
{
    if(argc != 3)
    {
        usage(argv[0]);
        exit(0);
    }
    int sockfd = socket(AF_INET,SOCK_DGRAM,0);

    std::string serverip = argv[1];
    uint16_t serverport = std::stoi(argv[2]);

    if(sockfd < 0)
    {
        std::cout<<"creat sockfd fail.";
        return 1;
    }

    std::string message;
    sockaddr_in server;
    while(true)
    {
        std::cout<<"请输入信息"<<std::endl;
        std::getline(std::cin,message);
        
        
        bzero(&server,sizeof(server));
        server.sin_family = AF_INET;
        server.sin_port = htons(serverport);
        server.sin_addr.s_addr = inet_addr(serverip.c_str());

        socklen_t len = sizeof(server);
        sendto(sockfd,message.c_str(),message.size(),0,(sockaddr*)&server,len);

        
        sockaddr_in temp;
        len = sizeof(temp);
        ssize_t s = recvfrom(sockfd,buffer,sizeof(buffer) - 1,0,(sockaddr*)&temp,&len);

        if(s > 0)
        {
            buffer[s] = '\0';
            std::cout<<"收到返回的信息："<<buffer<<std::endl;
        }
    }
    close(sockfd);
    return 0;
}
```
             
--- 
```cpp
/*UdpServer.hpp*/
#pragma once
#include <iostream>
#include <sys/types.h> /* See NOTES */
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <string.h>
#include <unistd.h>
#include <functional>

using func_t = std::function<std::string(std::string&)>;
func_t func;

enum
{ // 退出错误码
    SOCKET_ERR = 1,
    BIND_ERR = 2
};

uint16_t defualtport = 8889;
std::string defaultip = "0.0.0.0";
const int size = 1024;

class UdpServer
{
public:
    UdpServer(const uint16_t &port = defualtport, const std::string &ip = defaultip) : _port(port)
    {
    }
    ~UdpServer()
    {
        if(_sockfd > 0) close(_sockfd);
    }
    void init()
    {
        _sockfd = socket(AF_INET, SOCK_DGRAM, 0);
        if (_sockfd < 0)
        {
            std::cout << "error";
            exit(SOCKET_ERR);
        }
        std::cout << "socket创建成功" << std::endl;

        struct sockaddr_in local;
        bzero(&local, sizeof(local));
        local.sin_family = AF_INET; //(表明域——协议家族类型)//socket inet addr
        local.sin_port = htons(_port);     // 点分十进制字符串风格的IP地址
        local.sin_addr.s_addr = INADDR_ANY;

        int n = bind(_sockfd, (const sockaddr *)&local, sizeof(local));

        if (n < 0)
        {
            std::cout << "bind失败" << std::endl;
            exit(BIND_ERR);
        }
    }
    void run(func_t func)
    {
        char inbuffer[size];
        sockaddr client;
        _isrunning = true;
        socklen_t len = sizeof(client);
        while (_isrunning)
        {
            /* code */

            ssize_t n = recvfrom(_sockfd, inbuffer, sizeof(inbuffer) - 1, 0, (sockaddr *)&client, &len);
            if (n < 0)
            {
                std::cout << "接收信息失败";
                continue;
            }
            inbuffer[n] = 0;
            std::string info = inbuffer;
            std::cout<<"接收到的信息："<<info<<std::endl;
            std::string echo_string = "Udp Server：" + func(info);
            sendto(_sockfd,echo_string.c_str(),echo_string.size(),0,(const sockaddr*)&client,len);
        }
        

        
    }

private:
    int _sockfd; // 网络文件
    uint32_t _ip;
    uint16_t _port; // 服务器进程的端口号
    bool _isrunning;
};
```
---
```Makefile
.PHONY:all
all:Udp Udpclient

Udp:Main.cc
	g++ -o $@ $^ -std=c++11 -g
Udpclient:Udpclient.cc
	g++ -o $@ $^ -std=c++11 -g

.PHONY:clean
clean:
	rm -f Udp Udpclient

```

