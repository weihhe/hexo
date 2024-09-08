title: Select会话客户端服务器实现
author: weihehe
tags:
  - Select
categories:
  - 网络
date: 2024-09-05 10:50:00
---
练习多路复用
<!--more-->
```cpp
/*Select.h*/
#pragma once

#include <iostream>
#include <sys/select.h>
#include <sys/time.h>
#include "Sock.h"

using namespace std;

static const uint16_t defaultport = 8889;
static const int fd_num_max = (sizeof(fd_set) * 8);//一共的文件描述符大小
int defaultfd = -1;

class SelectServer
{
public:
    SelectServer(uint16_t port = defaultport) : _port(port)
    {
        for (int i = 0; i < fd_num_max; i++)
        {
            fd_array[i] = defaultfd;
            // std::cout << "fd_array[" << i << "]" << " : " << fd_array[i] << std::endl;
        }
    }
    bool Init()
    {
        _listensock.Socket();
        _listensock.Bind(_port);
        _listensock.Listen();

        return true;
    }
    void Accepter()
    {
        // 我们的连接事件就绪了
        std::string clientip;
        uint16_t clientport = 0;
        int sock = _listensock.Accept(&clientip, &clientport); //不会阻塞 // 因为rfds已经被select处理过，内部包含就绪的文件描述符
        if (sock < 0) return;
        lg(Info, "accept success, %s: %d, sock fd: %d", clientip.c_str(), clientport, sock);

        // sock -> fd_array[]
        int pos = 1;
        for (; pos < fd_num_max; pos++) // 第二个循环
        {
            if (fd_array[pos] != defaultfd)
                continue;
            else
                break;
        }
        if (pos == fd_num_max)
        {
            lg(Warning, "server is full, close %d now!", sock);
            close(sock);
        }
        else
        {
            fd_array[pos] = sock;
            PrintFd();
            // TODO
        }
    }
    void Recver(int fd, int pos)
    {
        // demo
        char buffer[1024];
        ssize_t n = read(fd, buffer, sizeof(buffer) - 1);
			if (n >= 0) {
    			 buffer[n] = '\0'; // 手动添加字符串终止符
			}
        if (n > 0)
        {
            buffer[n] = 0;
            cout << "get a messge: " << buffer << endl;
        }
        else if (n == 0)//对方关闭文件描述符
        {
            lg(Info, "client quit, me too, close fd is : %d", fd);
            close(fd);
            fd_array[pos] = defaultfd; // 这里本质是从select中移除
        }
        else
        {
            lg(Warning, "recv error: fd is : %d", fd);
            close(fd);
            fd_array[pos] = defaultfd; // 这里本质是从select中移除
        }
    }
    void Dispatcher(fd_set &rfds)
    {
        for (int i = 0; i < fd_num_max; i++) 
        {
            int fd = fd_array[i];
            if (fd == defaultfd)
                continue;//没有就绪的套接字不进行处

            if (FD_ISSET(fd, &rfds))//就绪的读取套接字集合中
            {
                if (fd == _listensock.Fd())//如果还是监听套接字，那么就是获取新连接
                {
                    Accepter(); 
                }
                else // 单纯读取
                {
                    Recver(fd, i);
                }
            }
        }
    }
    void Start()
    {
        int listensock = _listensock.Fd();
        fd_array[0] = listensock;//将监听套接字的文件描述符存储到辅助数组的0号下标
        for (;;)
        {
            fd_set rfds;
            FD_ZERO(&rfds);//仅仅做了读的套接字处理，但其实需要处理读写
            int maxfd = fd_array[0];
            for (int i = 0; i < fd_num_max; i++) // 监听套接字也要被FD_SET
            {
                if (fd_array[i] == defaultfd)
                    continue;
                FD_SET(fd_array[i], &rfds);
                if (maxfd < fd_array[i]) // 找到最大的文件描述符
                {
                    maxfd = fd_array[i];
                    lg(Info, "max fd update, max fd is: %d", maxfd);
                }
            }

            struct timeval timeout = {0, 0}; // 输入输出，可能要进行周期的重复设置
           
            int n = select(maxfd + 1, &rfds, nullptr, nullptr, /*&timeout*/ nullptr);
            switch (n)
            {
            case 0:
                cout << "time out, timeout: " << timeout.tv_sec << "." << timeout.tv_usec << endl;
                break;
            case -1:
                cerr << "select error" << endl;
                break;
            default:
                // 有事件就绪了，TODO
                cout << "get a new link!" << endl;
                Dispatcher(rfds); // 就绪的事件和fd你怎么知道只有一个呢？？？
                 //不可以使用accept，否则事件会阻塞
                break;
            }
        }
    }
    void PrintFd()
    {
        cout << "online fd list: ";
        for (int i = 0; i < fd_num_max; i++)
        {
            if (fd_array[i] == defaultfd)
                continue;
            cout << fd_array[i] << " ";
        }
        cout << endl;
    }
    ~SelectServer()
    {
        _listensock.Close();
    }

private:
    Sock _listensock;
    uint16_t _port;
    int fd_array[fd_num_max];   // 一个辅助数组，用于存储需要监听的文件描述符。并在不同的函数之间互相传递
};
```
```cpp
/*Sock.h*/
#pragma once

#include <iostream>
#include <string>
#include <unistd.h>
#include <cstring>
#include <sys/types.h>
#include <sys/stat.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <netinet/in.h>
#include "log.h"

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