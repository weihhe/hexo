title: Reactor —— 反应堆
author: weihehe
date: 2024-09-08 23:13:55
tags:
---
处理套接字`read`读取的问题
<!--more-->

## Reactor 模式 —— 半同步半异步模型 

### 事件分发：

- Reactor 模式中，事件分发器（Reactor）负责监视多个 I/O 事件，并在事件发生时分发给相应的处理程序。

- 事件分发器通常使用事件通知机制（如epoll, select, poll）来检测事件的发生。事件处理：
	- 当事件发生时，事件处理程序（Handler）会被调用来处理这些事件。例如，接收数据、发送数据或处理连接的关闭等。
    
	- 事件处理程序的执行通常是异步的，允许系统处理多个并发事件。
    
    
    
    

1. 对于每一个连接，我们都希望它拥有自己的一个缓冲区，用于处理自己的数据。因此我们可以使用
	 - 我们添加对应的事件，除了要加到内核中，fd， event。
	- `_epoller_ptr->EpllerUpdate(EPOLL_CTL_ADD, sock, event);`
    
	- 给sock也建立一个`connection`对象，将`1istensock`添加到`connection`中，同时，`listensock`和`connecion`放入`_connections`
    
    
统一将事件异常，转化为读写问题，因为事件异常，读写肯定也会出错。