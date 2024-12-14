title: '实现UDP通信 '
author: weihehe
tags:
  - UDP
categories:
  - 网络
date: 2024-08-13 11:25:00
---
UDP实现客户端和服务端之间的小写转换
<!--more-->

## UDP（用户数据报协议）通信的基本原理涉及以下几个方面：


### 1. **工作原理**

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
### Windowns下的Client实现
```cpp
#include <iostream>
#include <winsock2.h>
#include <windows.h>
#include <cstdlib>
#include <cstdio>
#include <string>
#pragma comment(lib, "ws2_32.lib") // 初始化
#pragma warning( disable : 4996)

int serverport = /*  */;
std::string serverip = /*"-----"*/;

int main(int argc, char* argv[])
{
	std::cout << "hello client" << std::endl;

	WSADATA wsd;
	if (WSAStartup(MAKEWORD(2, 2), &wsd) != 0) {
		std::cerr << "WSAStartup failed" << std::endl;
		return 1;
	}
	//创建套接字

	SOCKET sockfd = socket(AF_INET, SOCK_DGRAM, 0);
	if (sockfd == SOCKET_ERROR)
	{
		std::cout << "creat sockfd fail.";
		return 1;
	}
	char buffer[1024];

	sockaddr_in server;
	std::string message;
	while (true)
	{
		std::cout << "" << std::endl;
		std::getline(std::cin, message);

		memset(&server, 0, sizeof(server));
		server.sin_family = AF_INET;
		server.sin_port = htons(serverport);
		server.sin_addr.s_addr = inet_addr(serverip.c_str());

		int len = sizeof(server);
		sendto(sockfd, message.c_str(), message.size(), 0, (sockaddr*)&server, len);

		sockaddr_in temp;
		len = sizeof(temp);
		size_t s = recvfrom(sockfd, buffer, sizeof(buffer) - 1, 0, (sockaddr*)&temp, &len);

		if (s > 0)
		{
			buffer[s] = '\0';
			std::cout << "收到的信息：" << buffer << std::endl;
		}
	}
	closesocket(sockfd);
	WSACleanup();
	return 0;
}
```
### 补充函数

`FILE *popen(const char *command, const char *mode);`

`popen` 函数用于创建一个进程来执行指定的命令，并建立一个管道，以便在父进程和子进程之间进行通信。它可以用来运行外部命令并读取其输出，或者将数据传递给外部命令进行处理。


## 结合Json进行UDP传输（Windows）

准备工作

1. 在 Visual Studio 项目中，确保包含 `winsock2.h` 并链接 `Ws2_32.lib`。
2. 下载并包含 nlohmann/json 头文件（例如，`json.hpp`）。
3. 分别创建客户端和服务器端代码文件。

---

### 服务端代码

```cpp
#include <winsock2.h>
#include <iostream>
#include "json.hpp" // 包含 nlohmann/json 头文件

#pragma comment(lib, "Ws2_32.lib")

using json = nlohmann::json;

int main() {
    // 初始化 Winsock
    WSADATA wsaData;
    if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0) {
        std::cerr << "WSAStartup failed.\n";
        return -1;
    }

    // 创建 UDP 套接字
    SOCKET serverSocket = socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP);
    if (serverSocket == INVALID_SOCKET) {
        std::cerr << "Socket creation failed.\n";
        WSACleanup();
        return -1;
    }

    // 绑定地址和端口
    sockaddr_in serverAddr{};
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_port = htons(8080);
    serverAddr.sin_addr.s_addr = INADDR_ANY;

    if (bind(serverSocket, (sockaddr*)&serverAddr, sizeof(serverAddr)) == SOCKET_ERROR) {
        std::cerr << "Bind failed.\n";
        closesocket(serverSocket);
        WSACleanup();
        return -1;
    }

    std::cout << "Server is running on port 8080...\n";

    // 接收数据
    char buffer[512];
    sockaddr_in clientAddr{};
    int clientAddrLen = sizeof(clientAddr);

    while (true) {
        int recvLen = recvfrom(serverSocket, buffer, sizeof(buffer) - 1, 0, (sockaddr*)&clientAddr, &clientAddrLen);
        if (recvLen == SOCKET_ERROR) {
            std::cerr << "Receive failed.\n";
            continue;
        }

        buffer[recvLen] = '\0';
        std::cout << "Received: " << buffer << "\n";

        // 解析 JSON 数据
        try {
            json receivedJson = json::parse(buffer);
            std::cout << "Parsed JSON: " << receivedJson.dump(4) << "\n";
        } catch (json::parse_error& e) {
            std::cerr << "JSON Parse Error: " << e.what() << "\n";
        }
    }

    // 关闭套接字
    closesocket(serverSocket);
    WSACleanup();
    return 0;
}
```

---

### 客户端代码

```cpp
#include <winsock2.h>
#include <iostream>
#include "json.hpp" // 包含 nlohmann/json 头文件
#pragma warning( disable : 4996)


#pragma comment(lib, "Ws2_32.lib")

using json = nlohmann::json;

int main() {
    // 初始化 Winsock
    WSADATA wsaData;
    if (WSAStartup(MAKEWORD(2, 2), &wsaData) != 0) {
        std::cerr << "WSAStartup failed.\n";
        return -1;
    }

    // 创建 UDP 套接字
    SOCKET clientSocket = socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP);
    if (clientSocket == INVALID_SOCKET) {
        std::cerr << "Socket creation failed.\n";
        WSACleanup();
        return -1;
    }

    // 设置目标地址和端口
    sockaddr_in serverAddr{};
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_port = htons(8080);
    serverAddr.sin_addr.s_addr = inet_addr("127.0.0.1");

    // 构建 JSON 数据
    json data;
    data["message"] = "Hello, Server!";
    data["timestamp"] = time(nullptr);

    std::string jsonString = data.dump();
    std::cout << "Sending JSON: " << jsonString << "\n";

    // 发送数据
    if (sendto(clientSocket, jsonString.c_str(), jsonString.size(), 0, (sockaddr*)&serverAddr, sizeof(serverAddr)) == SOCKET_ERROR) {
        std::cerr << "Send failed.\n";
    }

    // 关闭套接字
    closesocket(clientSocket);
    WSACleanup();
    return 0;
}
```

---

### 编译和运行

1. 分别编译服务端和客户端代码。
2. 启动服务端。
3. 启动客户端，观察服务端接收并解析 JSON 数据。

### 注意事项

1. 确保防火墙允许 UDP 端口（例如 8080）。
2. 如果需要跨设备测试，请修改客户端代码中的 IP 地址为服务端的实际 IP。

## Linux环境完成Json进行UDP传输

在 Linux 下完成上述实验，你需要修改代码以使用 Linux 的套接字库。主要区别在于：

1. 替换 `winsock2.h` 和相关函数为标准的 POSIX 套接字库。
2. 不需要初始化和清理套接字（如 `WSAStartup` 和 `WSACleanup`）。
3. 使用 GCC 或 G++ 编译代码。

以下是 Linux 下的客户端和服务端代码示例：

---

### 服务端代码

```cpp
#include <iostream>
#include <cstring>
#include <arpa/inet.h>
#include <unistd.h>
#include "json.hpp" // nlohmann/json 头文件

using json = nlohmann::json;

int main() {
    // 创建 UDP 套接字
    int serverSocket = socket(AF_INET, SOCK_DGRAM, 0);
    if (serverSocket < 0) {
        perror("Socket creation failed");
        return -1;
    }

    // 绑定地址和端口
    sockaddr_in serverAddr{};
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_port = htons(8080);
    serverAddr.sin_addr.s_addr = INADDR_ANY;

    if (bind(serverSocket, (sockaddr*)&serverAddr, sizeof(serverAddr)) < 0) {
        perror("Bind failed");
        close(serverSocket);
        return -1;
    }

    std::cout << "Server is running on port 8080...\n";

    // 接收数据
    char buffer[512];
    sockaddr_in clientAddr{};
    socklen_t clientAddrLen = sizeof(clientAddr);

    while (true) {
        ssize_t recvLen = recvfrom(serverSocket, buffer, sizeof(buffer) - 1, 0, (sockaddr*)&clientAddr, &clientAddrLen);
        if (recvLen < 0) {
            perror("Receive failed");
            continue;
        }

        buffer[recvLen] = '\0';
        std::cout << "Received: " << buffer << "\n";

        // 解析 JSON 数据
        try {
            json receivedJson = json::parse(buffer);
            std::cout << "Parsed JSON: " << receivedJson.dump(4) << "\n";
        } catch (json::parse_error& e) {
            std::cerr << "JSON Parse Error: " << e.what() << "\n";
        }
    }

    close(serverSocket);
    return 0;
}
```

---

### 客户端代码

```cpp
#include <iostream>
#include <cstring>
#include <arpa/inet.h>
#include <unistd.h>
#include "json.hpp" // nlohmann/json 头文件

using json = nlohmann::json;

int main() {
    // 创建 UDP 套接字
    int clientSocket = socket(AF_INET, SOCK_DGRAM, 0);
    if (clientSocket < 0) {
        perror("Socket creation failed");
        return -1;
    }

    // 设置目标地址和端口
    sockaddr_in serverAddr{};
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_port = htons(8080);
    if (inet_pton(AF_INET, "127.0.0.1", &serverAddr.sin_addr) <= 0) {
        perror("Invalid address");
        close(clientSocket);
        return -1;
    }

    // 构建 JSON 数据
    json data;
    data["message"] = "Hello, Server!";
    data["timestamp"] = time(nullptr);

    std::string jsonString = data.dump();
    std::cout << "Sending JSON: " << jsonString << "\n";

    // 发送数据
    if (sendto(clientSocket, jsonString.c_str(), jsonString.size(), 0, (sockaddr*)&serverAddr, sizeof(serverAddr)) < 0) {
        perror("Send failed");
    }

    close(clientSocket);
    return 0;
}
```

---

### 编译代码

使用 G++ 编译服务端和客户端代码：

```bash
g++ -o server server.cpp -std=c++17
g++ -o client client.cpp -std=c++17
```

如果 `json.hpp` 文件存放在单独的目录下，可以使用 `-I` 参数指定路径，例如：

```bash
g++ -o server server.cpp -std=c++17 -I/path/to/json
g++ -o client client.cpp -std=c++17 -I/path/to/json
```

---

### 什么是 Header-only 库？

1. **nlohmann/json** 是一个完全由模板实现的库，所有功能都在头文件 `json.hpp` 中实现。
2. 使用这种库时，不需要预编译生成动态链接库（`*.so`）或静态链接库（`*.a`），只需在代码中 `#include "json.hpp"` 即可。
3. 因此，在编译时，编译器会直接将 `json.hpp` 的代码编译到目标程序中，而无需额外链接库。

--- 

使用 CJSON 库时，程序只需包含`<cjson/cJsoN.h>`头文件。而在编译时，
需要添加`-lcjson `选项

---

### 什么情况下需要 `-l` 选项？

`-l` 选项（例如 `-lcjson`）用于链接动态或静态库，通常用于：

1. **预编译的库文件**：例如，`cJSON` 是一个 C 实现的 JSON 处理库，提供的是动态/静态库文件（如 `libcjson.so` 或 `libcjson.a`）。编译时需要显式地告诉编译器链接这个库：
  
  ```bash
  g++ -o program program.cpp -lcjson
  ```
  
2. **操作系统提供的库**：例如，链接数学库时需要 `-lm`。
  

---

### 如何判断是否需要 `-l`？


1. 如果使用的是 header-only 库（如 nlohmann/json），只需 `#include`，无需额外的 `-l` 参数。
2. 如果库提供了预编译的二进制文件（如 `.so` 或 `.a` 文件），则需要在编译时通过 `-l` 指定。
3. 查看库的官方文档是最佳实践。对于 nlohmann/json，官方明确说明是 header-only 库，因此无需任何 `-l` 链接操作。