title: 多线程TCP通信
author: weihehe
tags:
  - 线程
  - TCP
categories:
  - 网络
date: 2024-08-17 16:09:00
---

代码实现
<!--more-->

## 须知

- 代码运行环境为:Ubuntu22.04，gcc 11.2.0。

- 子进程会继承父进程的文件描述符表，包括`listenSocketfd`和`socketfd`。
	- 因此**多进程通常要在父子进程中处理不需要的文件描述符表**（处理其引用计数）。
	- 而**线程**则将其视为**共享资源**，因此通常不需要在线程中对于文件描述符表进行处理。
    
## 代码如下

```cpp
//<---Main.cc--->
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
//<---TcpServer.hpp--->
#pragma once
#include <sys/socket.h>
#include <netinet/in.h>
#include <cstdlib>
#include <arpa/inet.h>
#include <string.h>
#include <unistd.h>
#include <iostream>
#include <sys/wait.h>
#include <signal.h>
#include <pthread.h>

enum error
{
    USAGEERROR = 1,
    SOCKET_ERROR,
    BIND_ERROR,
    LISTENERROR
};

const int defaultfd = -1;
const std::string defaultIp = "127.0.0.1"; // 提供一个有效的默认 IP 地址
const uint16_t defaultPort = 8889;
const int backlog = 5;
class TcpServer;
class ThreadDate{
    public:
        ThreadDate(int fd, const std::string &ip,const uint16_t &p,TcpServer* t)
        :_sockfd(fd),
        _clientPort(p),
        _clientIp(ip),
        _tsvr(t)
        {}

    public:
        int _sockfd;
        std::string _clientIp;
        uint16_t _clientPort;
        TcpServer* _tsvr;
};

class TcpServer
{
public:
    TcpServer(const uint16_t &port = defaultPort, const std::string &ip = defaultIp)
        : _listenSockfd(defaultfd), _port(port), _ip(ip) {}

    void initServer()
    {
        _listenSockfd = socket(AF_INET, SOCK_STREAM, 0);
        if (_listenSockfd < 0)
        {
            perror("Socket creation failed");
            exit(SOCKET_ERROR);
        }
        std::cout << "socket creation succeeded,fd-" << _listenSockfd << std::endl;
        struct sockaddr_in local;
        memset(&local, 0, sizeof(local));
        local.sin_family = AF_INET;
        local.sin_port = htons(_port);

        if (inet_pton(AF_INET, _ip.c_str(), &local.sin_addr) <= 0)
        {
            std::cerr << "Invalid IP address" << std::endl;
            exit(BIND_ERROR);
        }

        if (bind(_listenSockfd, (sockaddr *)&local, sizeof(local)) < 0)
        {
            std::cout << "bind error" << std::endl;
            exit(BIND_ERROR);
        }

        if (listen(_listenSockfd, backlog) < 0)
        {
            perror("Listen failed");
            exit(LISTENERROR);
        }

        std::cout << "initserver succeeded" << std::endl;
        std::cout << "已同步" << std::endl;
    }


    static void *routine(void *args) // static 取消this指针
    {
        pthread_detach(pthread_self()); // 设置线程分离状态
        ThreadDate *td = static_cast<ThreadDate *>(args);   
        td->_tsvr->server(td ->_sockfd,td->_clientIp,td->_clientPort);
        delete td;
        return nullptr;
    }


    void start()
    {
        // signal(SIGCHLD,SIG_IGN) 使用之后就不需要等待 <-----补充------->
        std::cout << "获取到了新链接" << std::endl;
        for (;;)
        {
            // 获取新连接
            sockaddr_in client;
            socklen_t len = sizeof(client);
            int sockfd = accept(_listenSockfd, (sockaddr *)&client, &len);
            if (sockfd < 0)
            {
                exit(SOCKET_ERROR);
            }
            u_int16_t cilentPort = ntohs(client.sin_port);

            char clientip[32];
            inet_ntop(AF_INET, &(client.sin_addr), clientip, sizeof(clientip));
            // server(sockfd,clientip,cilentPort);
            // close(sockfd); 单线程
            //-----------------↑ version 1 ↑--------------

            // pid_t id = fork();
            // if( id == 0)//子进程
            // {
            //     close(_listenSockfd);
            //     if(fork() > 0)  exit(0); //退出子进程
            //     server(sockfd,clientip,cilentPort);//对于孙进程来说，其父进程已经退出，会将其给系统托管，不需要我们手动回收
            //     close(sockfd);
            //     exit(0);
            // }
            // close(sockfd);
            // pid_t fid = waitpid(id,nullptr,0);//由于子进程立刻退出，立刻等待成功,回收子进程
            // (void)fid;

            //-----------------↑ version 2 ↑ 并发多进程，代价很高--------------
            ThreadDate *td = new ThreadDate(sockfd,clientip,cilentPort,this);
            pthread_t tid;
            pthread_create(&tid, nullptr, routine,td); // 新线程执行routine
            //-----------------↑ version 3 ↑ 多线程版本--------------
            
        }
    }

    void server(int sockfd, const std::string clientip, const uint16_t &clientPort)
    {
        char buffer[4096];
        while (true)
        {
            ssize_t n = read(sockfd, buffer, sizeof(buffer));
            if (n > 0)
            {
                buffer[n] = 0;
                std::cout << "读取成功" << buffer << std::endl;
                std::string echoString = "tcpServer 处理后:";
                echoString += buffer;

                write(sockfd, echoString.c_str(), echoString.size());
            }
        }
    }
    ~TcpServer()
    {
        if (_listenSockfd > 0)
            close(_listenSockfd);
    }

private:
    int _listenSockfd; // 负责获取新链接
    uint16_t _port;
    std::string _ip;
};
// 同步

```

```makefile
.PHONY:all
all:TcpServer TcpClient

TcpServer : Main.cc
	g++ -o $@ $^ -std=c++11 -lpthread
TcpClient : TcpClient.cc
	g++ -o $@ $^ -std=c++11 

.PHONY:clean
clean:
	rm -f TcpServer TcpClient
```