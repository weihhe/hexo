title: Socket类
author: weihehe
tags:
  - 套接字
categories:
  - 网络
date: 2024-09-06 07:58:00
---
对socket相关的函数进行一些封装
<!--more-->

```cpp
/*-----实验代码如下-----*/
enum
{
    SocketErr = 2,
    BindErr,
    ListenErr,
};

// TODO
const int backlog = 10;

class Sock
{
public:
    Sock()
    {
    }
    ~Sock()
    {
    }

public:
    void Socket()
    {
        sockfd_ = socket(AF_INET, SOCK_STREAM, 0);
        if (sockfd_ < 0)
        {
            lg(Fatal, "socker error, %s: %d", strerror(errno), errno);
            exit(SocketErr);
        }
        int opt = 1;
        setsockopt(sockfd_, SOL_SOCKET, SO_REUSEADDR | SO_REUSEPORT, &opt, sizeof(opt));
    }
    void Bind(uint16_t port)
    {
        struct sockaddr_in local;
        memset(&local, 0, sizeof(local));
        local.sin_family = AF_INET;
        local.sin_port = htons(port);
        local.sin_addr.s_addr = INADDR_ANY;

        if (bind(sockfd_, (struct sockaddr *)&local, sizeof(local)) < 0)
        {
            lg(Fatal, "bind error, %s: %d", strerror(errno), errno);
            exit(BindErr);
        }
    }
    void Listen()
    {
        if (listen(sockfd_, backlog) < 0)
        {
            lg(Fatal, "listen error, %s: %d", strerror(errno), errno);
            exit(ListenErr);
        }
    }
    int Accept(std::string *clientip, uint16_t *clientport)
    {
        struct sockaddr_in peer;
        socklen_t len = sizeof(peer);
        int newfd = accept(sockfd_, (struct sockaddr *)&peer, &len);
        if (newfd < 0)
        {
            lg(Warning, "accept error, %s: %d", strerror(errno), errno);
            return -1;
        }
        char ipstr[64];
        inet_ntop(AF_INET, &peer.sin_addr, ipstr, sizeof(ipstr));
        *clientip = ipstr;
        *clientport = ntohs(peer.sin_port);

        return newfd;
    }
    bool Connect(const std::string &ip, const uint16_t &port)
    {
        struct sockaddr_in peer;
        memset(&peer, 0, sizeof(peer));
        peer.sin_family = AF_INET;
        peer.sin_port = htons(port);
        inet_pton(AF_INET, ip.c_str(), &(peer.sin_addr));

        int n = connect(sockfd_, (struct sockaddr *)&peer, sizeof(peer));
        if (n == -1)
        {
            std::cerr << "connect to " << ip << ":" << port << " error" << std::endl;
            return false;
        }
        return true;
    }
    void Close()
    {
        close(sockfd_);
    }
    int Fd()
    {
        return sockfd_;
    }

private:
    int sockfd_;
};
```

## 套接字类

### 全局常量

```cpp
const int backlog = 10;
```

- 定义了一个全局常量 `backlog`，用于设置 `listen` 函数的待连接队列的最大长度（即允许的最大等待连接的数目）。

### Sock 类

```cpp
class Sock
{
public:
    Sock() { }
    ~Sock() { }

public:
    void Socket();
    void Bind(uint16_t port);
    void Listen();
    int Accept(std::string *clientip, uint16_t *clientport);
    bool Connect(const std::string &ip, const uint16_t &port);
    void Close();
    int Fd();

private:
    int sockfd_;
};
```

- `Sock` 类封装了与套接字操作相关的方法：
  - `Socket()`：创建套接字。
  - `Bind(uint16_t port)`：绑定本地地址和端口。
  - `Listen()`：将套接字设置为监听模式。
  - `Accept(std::string *clientip, uint16_t *clientport)`：接受新的连接。
  - `Connect(const std::string &ip, const uint16_t &port)`：发起到指定 IP 和端口的连接。
  - `Close()`：关闭套接字。
  - `Fd()`：返回套接字的文件描述符。

#### Socket 方法

```cpp
void Sock::Socket()
{
    sockfd_ = socket(AF_INET, SOCK_STREAM, 0);
    if (sockfd_ < 0)
    {
        lg(Fatal, "socker error, %s: %d", strerror(errno), errno);
        exit(SocketErr);
    }
    int opt = 1;
    setsockopt(sockfd_, SOL_SOCKET, SO_REUSEADDR | SO_REUSEPORT, &opt, sizeof(opt));
}
```

- `Socket()` 方法用于创建一个 IPv4 的 TCP 套接字。
- `socket(AF_INET, SOCK_STREAM, 0)` 创建了一个 IPv4 地址族的流式套接字。
- 如果套接字创建失败，程序会记录错误信息并退出。
- `setsockopt()` 设置了套接字选项 `SO_REUSEADDR` 和 `SO_REUSEPORT`，使得端口可以被重用。

#### Bind 方法

```cpp
void Sock::Bind(uint16_t port)
{
    struct sockaddr_in local;
    memset(&local, 0, sizeof(local));
    local.sin_family = AF_INET;
    local.sin_port = htons(port);
    local.sin_addr.s_addr = INADDR_ANY;

    if (bind(sockfd_, (struct sockaddr *)&local, sizeof(local)) < 0)
    {
        lg(Fatal, "bind error, %s: %d", strerror(errno), errno);
        exit(BindErr);
    }
}
```

- `Bind()` 方法将套接字绑定到指定的本地端口。
- `struct sockaddr_in local` 用于定义本地地址信息（`AF_INET`，指定端口，任意本地 IP 地址）。
- 使用 `bind()` 绑定套接字，如果失败则记录错误并退出。

#### Listen 方法

```cpp
void Sock::Listen()
{
    if (listen(sockfd_, backlog) < 0)
    {
        lg(Fatal, "listen error, %s: %d", strerror(errno), errno);
        exit(ListenErr);
    }
}
```

- `Listen()` 方法将套接字设置为监听模式，允许接受传入的连接。
- `listen(sockfd_, backlog)` 将套接字设置为被动模式，等待连接，并设置最大待连接数目为 `backlog`。

#### Accept 方法

```cpp
int Sock::Accept(std::string *clientip, uint16_t *clientport)
{
    struct sockaddr_in peer;
    socklen_t len = sizeof(peer);
    int newfd = accept(sockfd_, (struct sockaddr*)&peer, &len);
    if(newfd < 0)
    {
        lg(Warning, "accept error, %s: %d", strerror(errno), errno);
        return -1;
    }
    char ipstr[64];
    inet_ntop(AF_INET, &peer.sin_addr, ipstr, sizeof(ipstr));
    *clientip = ipstr;
    *clientport = ntohs(peer.sin_port);

    return newfd;
}
```

- `Accept()` 方法用于接受传入连接并获取客户端信息。
- `accept(sockfd_, (struct sockaddr*)&peer, &len)` 阻塞等待一个新的连接，并返回新的套接字描述符 `newfd`。
- 如果 `accept()` 失败，记录错误并返回 `-1`。
- 将客户端的 IP 地址和端口转换为字符串和主机字节序，并通过指针返回。

#### Connect 方法

```cpp
bool Sock::Connect(const std::string &ip, const uint16_t &port)
{
    struct sockaddr_in peer;
    memset(&peer, 0, sizeof(peer));
    peer.sin_family = AF_INET;
    peer.sin_port = htons(port);
    inet_pton(AF_INET, ip.c_str(), &(peer.sin_addr));

    int n = connect(sockfd_, (struct sockaddr*)&peer, sizeof(peer));
    if(n == -1) 
    {
        std::cerr << "connect to " << ip << ":" << port << " error" << std::endl;
        return false;
    }
    return true;
}
```

- `Connect()` 方法用于发起到指定服务器 IP 和端口的连接。
- 使用 `inet_pton` 将字符串形式的 IP 地址转换为网络字节序，并指定远程地址。
- 调用 `connect()` 发起连接，如果失败，记录错误并返回 `false`。

#### Close 方法

```cpp
void Sock::Close()
{
    close(sockfd_);
}
```

- `Close()` 方法关闭套接字连接。

#### Fd 方法

```cpp
int Sock::Fd()
{
    return sockfd_;
}
```

- `Fd()` 方法返回套接字的文件描述符 `sockfd_`。

### 总结

- 这段代码实现了一个简洁的 C++ 套接字类 `Sock`，封装了常见的套接字操作，如创建、绑定、监听、接受、连接和关闭。
- 错误处理通过日志记录和 `exit` 完成，保证了程序在出现关键错误时能够及时终止。

### 为什么第一次创建 TCP 连接时 `fd` 返回 4？

1. **文件描述符的分配机制**：
  
  - 当进程启动时，操作系统会为每个进程提供标准输入（stdin）、标准输出（stdout）、标准错误（stderr）分配文件描述符 0、1 和 2。
  
2. **第一次调用 `accept` 函数**：
  
  - 当 `accept` 函数成功从监听套接字 `sockfd_` 中接受一个新的连接时，操作系统会为这个新的连接分配一个文件描述符。
  
  - 如果这是程序第一次接受连接请求，并且除了标准输入、输出、错误和监听套接字外，程序中没有其他打开的文件描述符，那么操作系统会将文件描述符 4 分配给这个新连接。

因此，**第一次创建 TCP 连接时返回的 `fd` 是 4** 是因为在这之前，已经有了 0、1、2（标准输入、输出、错误）和 3（监听套接字）几个文件描述符被占用了，所以下一个可用的文件描述符就是 4。