title: muduo库的安装使用
author: weihehe
tags: []
categories:
  - 网络
date: 2024-09-15 12:47:00
---
网络库之一
<!--more-->
## Muduo库 —— 网络通信

## 编译安装库

```bash
# 克隆 Muduo 仓库
git clone https://github.com/chenshuo/muduo.git

# 进入 Muduo 目录
cd muduo

# 创建构建目录并进入
mkdir build
cd build

# 生成构建文件
cmake ..

# 编译并安装
make 
sudo make install

```

## 使用注意

1. 链接时，`-lmuduo_net`也要在` -lmuduo_base`之前链接。

	- 例如`LFLAG = -lmuduo_net -lmuduo_base -pthread`
    
		- `-ljsoncpp`：这个选项告诉链接器链接`jsoncpp`库，`jsoncpp`是一个用于处理JSON的C++库。
    
		- `-lmuduo_net` 和 `-lmuduo_base`：这两个选项分别告诉链接器链接`muduo_net`和`muduo_base`库。
    
		- `-pthread`：这个选项告诉链接器链接POSIX线程库，这是用于多线程编程的库。在Linux下，使用多线程通常需要这个选项。

2. `_baseloop`需要在`_server`的上方声明，因为要使用`_bashloop`来构造`_server`。

3. **`_loopthread.startLoop()`**: 
  - `EventLoopThread` 的 `startLoop` 方法被调用以启动事件循环线程。这个方法会创建一个新的线程，并在该线程中运行事件循环 (`EventLoop` 的 `loop` 方法)。
  - `startLoop` 返回一个指向事件循环 (`EventLoop`) 的指针，这个事件循环将用于处理网络事件。

4. **客户端的事件循环**

	- 客户端需要循环输入数据 —— 阻塞，因此事件循环的需要另启一个线程。

5. **客户端事件循环的开始**:
  - 事件循环是在 `EventLoopThread` 的内部线程中开始的。当 `startLoop` 被调用时，新的线程启动，并在这个线程中调用 `EventLoop::loop()`，从而开始事件循环。
  
  - 在 `DictClient` 的构造函数中，`_loopthread.startLoop()` 会启动这个事件循环线程，并将事件循环对象的指针保存在 `_baseloop` 中。如下：
```cpp
 DictClient(const std::string &sip, int sport):
            _baseloop(_loopthread.startLoop()),//初始化客户端，新的线程启动，进行事件循环
            _downlatch(1),//初始化计数器为1，因为为0时才会唤醒wait所在的线程
            _client(_baseloop, muduo::net::InetAddress(sip, sport), "DictClient")
        {
            //设置连接事件（连接建立/管理）的回调
            _client.setConnectionCallback(std::bind(&DictClient::onConnection, this, std::placeholders::_1));
            //设置连接消息的回调
            _client.setMessageCallback(std::bind(&DictClient::onMessage, this, 
                std::placeholders::_1, std::placeholders::_2, std::placeholders::_3));
            
            //连接服务器
            _client.connect();//connet是一个非阻塞的接口，此时不一定有数据，因此需要进行同步
            _downlatch.wait();//经过连接的回调函数处理后，_downlatch.countDown()经过--，此时为0阻塞
        }
```

6. `downlatch.wait`用于同步等待客户端连接的建立。具体步骤：

	1. **初始化计数器**:
  
  	- 在 `DictClient` 构造函数中，`_downlatch` 被初始化为 1。
  	- 这意味着，只有在 `countDown()` 被调用一次之后，调用 `wait()` 的线程才会被唤醒。
  
	2. **等待连接建立**:
  
  	- 在构造函数中调用 `wait()`，使主线程进入等待状态，直到连接建立为止。
    
	3. **连接回调中减小计数器**:
  
  	- 当连接建立时，`onConnection` 回调函数会被调用。在连接成功时，调用 `_downlatch.countDown()` 减小计数器，使其变为 0。
  	- 当计数器变为 0 时，阻塞在 `wait()` 上的**主线程**会被唤醒，继续执行后续代码。