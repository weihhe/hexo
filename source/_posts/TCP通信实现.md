title: '实现简单TCP通信 '
author: weihehe
tags:
  - TCP
categories:
  - 网络
date: 2024-08-16 11:05:00
---
Tcp协议实现
<!--more-->

- **Tcp是面向连接的**，因此服务器一般是比较“被动的”，即**持续监听**。

- **监听套接字——监听新链接**和**通信套接字——和客户端通信**分离，分别负责不同的功能。

- **socket通信**，都需要`IP`+`Port`的方式来区分自身唯一性。只是对于客户端来说，这个端口只要是能标记**自身唯一性**就行。

## 简单的重连机制

```cpp
do
{
    int n = write(sockfd, message.c_str(), message.size());
    if (n < 0)
    {
        state = true;
        cnt--;
    }
    else
    {
        state = false;
    }

} while (state && cnt);
```

## Tcp中涉及的经典函数

### `listen()` 函数

**原型：**
```cpp
int listen(int sockfd, int backlog);
```

**功能：**
- `listen()` 函数将一个已绑定（`bind()`）的套接字（`sockfd`）设置为被动监听状态，准备接受传入的连接请求。
- 该函数通常在服务器端使用，是创建 TCP 服务器的重要步骤之一。

**参数：**
- `sockfd`：要设置为监听状态的套接字描述符，通常是通过 `socket()` 和 `bind()` 创建并绑定的。
- `backlog`：指定连接队列的最大长度，表示在处理当前连接时，可以有多少个未处理的连接在队列中等待。

**返回值：**
- 成功时返回 `0`，失败时返回 `-1` 并设置 `errno`。

### `connect() `函数

**原型**

```cpp
int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);
```
**sockfd**: 这是套接字描述符，是通过调用 `socket()` 函数创建的。

**addr**: 这是一个指向`sockaddr` 结构体的指针，包含了服务器的地址信息（IP 地址和端口号）。

**addrlen**: 这是`addr`构体的长度，通常使用 sizeof(struct sockaddr_in) 或 sizeof(struct sockaddr_in6)。

### `accept()` 函数

**原型：**
```cpp
int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

**功能：**
- `accept()` 函数从监听套接字中提取一个已完成的连接请求，生成一个新的套接字用于与客户端进行通信。
- 该函数阻塞调用，直到有新的连接请求进来。

**参数：**
- `sockfd`：监听套接字描述符，之前通过 `listen()` 设置为监听状态。
- `addr`：指向 `sockaddr` 结构体的指针，用于存储客户端的地址信息。可以为 `NULL`，表示不关心客户端地址。
- `addrlen`：指向 `socklen_t` 类型的指针，用于存储 `addr` 的大小。传递 `NULL` 时，表示忽略地址长度。

**返回值：**
- 成功时返回一个新的套接字描述符（用于与客户端通信），失败时返回 `-1` 并设置 `errno`。

## 实验代码（无重连机制）

```cpp
//<----Main.cc---->
#include "TcpServer.hpp"
#include <memory>

void Usage(std::string proc)
{
    std::cout<<"Usage:"<<proc<<"请输入端口号\n"<<std::endl;
}

int main(int argc, char* argv[]) {
    if (argc != 2) {
        Usage(argv[0]);
        exit(USAGEERROR);
    }

    uint16_t port = std::stoi(argv[1]);
    std::unique_ptr<TcpServer> tcp_ptr(new TcpServer(port));
    tcp_ptr->initServer();
    std::cout << "Server started" << std::endl;
    tcp_ptr->start();

    return 0;
}

```
```cpp
//<--TcpClient.cc-->
#include <iostream>
#include <sys/types.h>          /* See NOTES */
#include <sys/socket.h>
#include <cstring>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>


enum{
    USAGEERROR = 1,
    SOCKETERROR,
    CONNECTERROR,
    READERROR
};

void usage(const char* instr)
{
    std::cerr<<instr<<"+IP + 端口号";
    
}


int main(int argc,char* argv[])
{
    if(argc != 3){
        usage(argv[0]);
        exit(USAGEERROR);
    }
    
    std::string serverip = argv[1];
    uint16_t serverport = std::stoi(argv[2]);

    int sockfd = socket(AF_INET,SOCK_STREAM,0);
    if(sockfd < 0)
    {
        std::cerr<<"sockfd fail.";
        exit(SOCKETERROR);
    }
    std::cout<<"创建socket成功"<<std::endl;
    sockaddr_in server;
    memset(&server,0,sizeof(server));
    server.sin_family = AF_INET;
    inet_pton(AF_INET,serverip.c_str(),&(server.sin_addr));
    server.sin_port = htons(serverport);

    int n = connect(sockfd,(struct sockaddr*)&server,sizeof(server));
    if (n < 0)
    {
        std::cerr<<"connect error.."<<std::endl;
        exit(CONNECTERROR);
    }
     std::cout<<"connet成功"<<std::endl;
    std::string message;
    while(true)
    {
        std::cout<<"输入要发送的信息";
        std::getline(std::cin,message);

        write(sockfd,message.c_str(),message.size());

        char inbuffer[2048];
        int n = read(sockfd,inbuffer,sizeof(inbuffer));
        if(n > 0)
        {
            inbuffer[n] = 0;
            std::cout<<inbuffer<<std::endl;
        }
    }
    close(sockfd);
    return 0;
}

```

```Makefile

.PHONY:all
all:TcpServer TcpClient

TcpServer : Main.cc
	g++ -o $@ $^ -std=c++11 -g
TcpClient : TcpClient.cc
	g++ -o $@ $^ -std=c++11 -g

.PHONY:clean
clean:
	rm -f TcpServer TcpClient
```