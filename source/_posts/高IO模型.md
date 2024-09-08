title: 高IO模型
author: weihehe
tags:
  - IO
categories:
  - 网络
date: 2024-09-05 14:44:00
---
Select Poll Epoll
<!--more-->

## Input && Output

- I/O 操作实际上涉及上下文切换，即从用户态切换到内核态的过程。因此会产生数据缓冲区之间的拷贝。

- 简单来说，`IO` = `等待读写事件` + `拷贝数据`两个过程，单位时间内，IO的过程中，`等待`比重越小，效率就越高。

## 五种IO模型

### 同步式IO：参与了`等待`或者`拷贝`的过程

- 阻塞式IO：

![阻塞式IO](/images/阻塞式IO.png)

- 轮询式——非阻塞IO非阻塞轮询：当`fd`没有数据就绪的时候，`recv`,`read`,`write`,`send`会报错误码，通过`strerror(errno)`就可以准确判断是哪种报错。

![轮询式IO](/images/非阻塞IO.png)

- 信号驱动式IO：

![信号驱动式IO](/images/pasted-22.png)


- 多路复用，多路转接 —— 一次等待多个文件描述符：

![多路复用IO](/images/多路复用IO.png)

### 异步式IO：不参与IO，只是发起IO

![异步IO](/images/信号驱动式IO.png)


`fcntl` 是一个在 Unix 和类 Unix 操作系统（如 Linux）中使用的系统调用，它的名称来自 "file control" 的缩写。`fcntl` 主要用于操作文件描述符的属性和行为。文件描述符是操作系统中用于标识一个打开的文件、socket、管道等 I/O 资源的整数。

## `fcntl` 函数

### 功能
- 复制一个现有的描述符（`cmd=F_DUPFD`)

- 获得/设置文件描述符标记(`cmd=F_GETFD`或`F_SETFD`)

- 获得/设置文件状态标记(`cmd=F_GETFL`或`F_SETFL`)

- 获得/设置异步I/O所有权(`cmd=F_GETOWN`或`F_SETOWN`)

- 获得/设置记录锁(`cmd=F_GETLK`,`F_SETLK或F_SETLKW`)

| **功能（位图方式存储）**                      | **描述**                                      |
|-------------------------------|-----------------------------------------------|
| `F_DUPFD`                      | 复制一个现有的描述符                          |
| `F_GETFD` / `F_SETFD`          | 获得/设置文件描述符标记                       |
| `F_GETFL` / `F_SETFL`          | 获得/设置文件状态标记                         |
| `F_GETOWN` / `F_SETOWN`        | 获得/设置异步 I/O 所有权                      |
| `F_GETLK` / `F_SETLK` / `F_SETLKW` | 获得/设置记录锁                          |



### `fcntl` 的函数原型

```c
int fcntl(int fd, int cmd, ... /* arg */ );
```

- `fd`：表示要操作的文件描述符。
- `cmd`：表示要执行的操作，比如获取或设置文件描述符标志、加锁或解锁等。
- `arg`：可选参数，根据 `cmd` 的不同而有所变化。

### 例子

- **设置文件描述符为非阻塞模式**：

   ```c
   int flags = fcntl(fd, F_GETFL, 0);  // 获取文件描述符的标志
   fcntl(fd, F_SETFL, flags | O_NONBLOCK);  // 设置为非阻塞模式
   ```

## 多路转接（一次等待多个文件描述符） —— `select`（如果有事件就绪，上层不处理，那么select会一直进行通知）

- 注意：新连接会被看作**读事件**，不同于**读取文件**，新连接的文件描述符等于监听套接字的文件描述符。

### 函数原型

```c
int select(int nfds, fd_set *readfds, fd_set *writefds,
           fd_set *exceptfds, struct timeval *timeout);
```

### 参数

- 大多都是输入输出型参数——读取它的值即输出，写入它的值即输入。因此为了保证每次事件的正常允许，常常需要周期的重新设置这些值。

1. **`nfds`** :
  
  - 类型: `int`
  - 说明: 监控的文件描述符的范围是从 0 到 `nfds - 1`。这个值通常是所有监控文件描述符中最大值加 1。**`select` 函数会检查所有小于 `nfds` 的文件描述符，`-1`的原因**。
  
2. **`readfds`**:
  
  - 类型: `fd_set *`
  
  - 说明: 指向 `fd_set` 结构的指针，表示要**监控的可读文件描述符集合**。
  
  	- `fd_set` 是一个数据结构，用于存储文件描述符的集合。
    - 调用 `select` 前需要使用 `FD_ZERO` 清空集合，并用 `FD_SET` 将需要监控的文件描述符添加到集合中。`select` 调用后，`readfds` 中的文件描述符集合将只包含那些已准备好进行读操作的文件描述符（将就绪的文件描述符的标志位置为`1`，并且D_SETSIZE 一般的比特位大小是1024，即最多可以存储1024个文件描述符。位图的思想。）
    
3. **`writefds`**:
  
  - 类型: `fd_set *`
  - 说明: 指向 `fd_set` 结构的指针，表示要**监控的可写文件描述符集合**。
		- 与 `readfds` 类似，`writefds` 中的文件描述符集合将只包含那些已准备好进行写操作的文件描述符。
  
4. **`exceptfds`**:
  
  - 类型: `fd_set * （位图结构）`
  - 说明: 指向 `fd_set` 结构的指针，表示要监控的异常文件描述符集合。异常通常指文件描述符的状态变化，如紧急数据到达等。`exceptfds` 中的文件描述符集合将只包含那些有异常条件的文件描述符。
  
5. **`timeout` —— 输入输出型参数**:
  
  - 类型: `struct timeval *`
  - 说明: 指向 `struct timeval` 结构的指针，表示 `select` 等待事件发生的最大时间。`struct timeval` 结构包含两个字段：`tv_sec`（秒）和 `tv_usec`（微秒）。如果 `timeout` 为 `NULL`，`select` 将无限期等待，直到至少一个文件描述符就绪。如果 `timeout` 设为 0，则 `select` 会立即返回（非阻塞模式）。

### 输入输出型参数

- 对于一些参数来说，可能因为上一次`事件响应`改变了参数值，而影响`本次事件响应`的结果，因此需要周期的重新初始化参数。例如对`timeout`周期性赋值为`等待事件发生的最大时间`，便于正确的等待事件。

### 返回值

- **成功**: 返回值是就绪的文件描述符数量，即 `readfds`、`writefds` 和 `exceptfds` 中的文件描述符数量。

- **失败**: 返回 `-1` 并设置 `errno`，指示出错。

### 位图

| 宏函数       | 功能描述                                                                 |
|--------------|--------------------------------------------------------------------------|
| `FD_SET(fd, &fdset)`    | 将文件描述符 `fd` 加入到文件描述符集合 `fdset` 中。                         |
| `FD_CLR(fd, &fdset)`    | 将文件描述符 `fd` 从文件描述符集合 `fdset` 中移除。                          |
| `FD_ISSET(fd, &fdset)`  | 检查文件描述符 `fd` 是否在文件描述符集合 `fdset` 中。如果在，返回非零值。       |
| `FD_ZERO(&fdset)`       | 清空文件描述符集合 `fdset`，将其初始化为空集合。                              |
- 若fd＝5,执行FD_SET(fd,&set);
后set变为0001,0000(第5位置为1)


### `select`使用示例

```c
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
        ssize_t n = read(fd, buffer, sizeof(buffer) - 1); // bug?
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
#### 关键

- 将等待就绪的套接字的工作交给`select`，而`accept`直进行处理。

- 就绪的套接字，有些是用来连接，有些是用于IO要进行区分。

- `ssize_t n = read(fd, buffer, sizeof(buffer) - 1); // bug?`

### 解释

1. **清空文件描述符集合**: 使用 `FD_ZERO` 清空 `readfds` 集合。
2. **添加文件描述符**: 使用 `FD_SET` 将标准输入（`STDIN_FILENO`）添加到集合中。
3. **设置超时时间**: `timeout` 结构设置为 5 秒。
4. **调用 `select`**: `select` 将阻塞，直到标准输入上有数据可读，或者超时。

### select的缺点
#### 1. 等待的 `fd` 是有上限的

- `select` 函数的设计中，能同时监控的文件描述符（`fd`）数量是有限制的，通常在系统中定义为 `FD_SETSIZE`。在许多系统中，这个上限值是 1024。

#### 2. 输入输出型参数比较多，数据拷贝的频率比较高

- **解释**: `select` 的参数 `readfds`, `writefds`, `exceptfds` ,`timeout`都是输入输出型参数（`inout` 参数）。调用 `select` 时，这些文件描述符集会从用户空间拷贝到内核空间，然后在检测事件之后，再从内核空间拷贝回用户空间。

#### 3. 输入输出型参数比较多，每次都要对关心的 `fd` 进行事件重置

- **解释**: 每次调用 `select` 后，输入输出型参数的文件描述符集合都会被修改（即标记哪些文件描述符已经就绪），所以在下次调用 `select` 之前，必须重新设置这些集合。

#### 4. 用户层，使用第三方数组管理用户的 `fd`，用户层需要很多次遍历，内核中检测 `fd` 事件就绪，也要遍历

- **解释**:
  - 用户层代码通常使用数组或其他数据结构来管理所有需要监控的文件描述符，这些结构需要遍历来设置 `fd_set` 并在 `select` 返回后处理就绪的 `fd`。
  
  - 在内核中，`select` 也需要遍历所有的 `fd_set` 集合来检测哪些文件描述符已经就绪。
- **影响**: 这种双重遍历的方式（用户层遍历和内核遍历）导致性能开销增加

## `poll`

是一种用于 I/O 多路复用的系统调用，提供了类似于 `select` 的功能，但在处理大量文件描述符时通常更高效。下面是如何使用 `poll` 的详细说明。

### `poll` 的函数原型

```cpp
#include <poll.h>

int poll(struct pollfd *fds, nfds_t nfds, int timeout);
```

### 参数

1. **`fds`**:
  
  - 类型: `struct pollfd *`
  - 描述: 指向 `pollfd` 结构体数组的指针。每个 `pollfd` 结构体表示一个文件描述符及其相关事件。
2. **`nfds`**:
  
  - 类型: `nfds_t`
  - 描述: `fds` 数组中 `pollfd` 结构体的数量。
3. **`timeout`**:
  
  - 类型: `int`
  - 描述: 等待的超时时间，以毫秒为单位。如果设置为 `-1`，则 `poll` 将无限期等待直到事件发生。如果设置为 `0`，则 `poll` 会立即返回，不管事件是否发生。

### `pollfd` 结构体

`pollfd` 结构体定义如下：

```cpp
struct pollfd {
    int fd;         // 要监控的文件描述符
    short events;   // 请求的事件类型
    short revents;  // 返回的事件类型
};
```

#### `events` 字段的事件标志

- `POLLIN` : 表示文件描述符可读（即有数据可读）。
- `POLLOUT` : 表示文件描述符可写（即可以写入数据而不会阻塞）。
- `POLLERR` : 表示文件描述符发生了错误。
- `POLLHUP` : 表示文件描述符的挂起状态（例如，管道关闭）。
- `POLLPRI` : 表示文件描述符有紧急数据可读取（如优先级数据）。

##### 例如：
`_event_fds[i].revents & POLLIN `这一表达式的意义是：

检查 revents 中是否包含 POLLIN 标志位。如果包含，说明即此文件描述符上的读事件已经准备就绪。

#### `revents` 字段的事件标志

- `revents` 是 `poll` 返回时设置的字段，表示实际发生的事件类型。其标志通常与 `events` 字段中的标志相同，但可能会有不同的组合。

## `poll`和`select`都是由用户维护的特定数据结构，来把我们的文件描述符管理起来

## epoll

- `#include<sys/epoll.h>`

###  `epoll_create`

```cpp
int epoll_create(int size);
```

- **功能**: 创建一个 `epoll` 实例。`size` 参数在现代 Linux 系统中已被忽略，实际的大小由内核动态管理。
- **返回值**: 返回一个 `epoll` 文件描述符，成功时大于等于 0；失败时返回 -1，并设置 `errno`。

###  `epoll_ctl`

```cpp
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

- **功能**: 控制 `epoll` 实例的行为。可以添加、修改或删除监控的文件描述符。
  - `epfd`: 由 `epoll_create` 返回的 `epoll` 文件描述符。
  - `op`: 操作类型，可能的值有：
    - `EPOLL_CTL_ADD`: 添加文件描述符到 `epoll` 实例中。
    - `EPOLL_CTL_MOD`: 修改文件描述符的监控事件。
    - `EPOLL_CTL_DEL`: 从 `epoll` 实例中删除文件描述符。
  - `fd`: 需要控制的文件描述符。
  - `event`: 指向 `epoll_event` 结构体的指针，指定文件描述符的事件类型。
- **返回值**: 成功时返回 0；失败时返回 -1，并设置 `errno`。

###  `epoll_wait`

```cpp
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
```

- **功能**: 等待 `epoll` 就绪的事件，并将其添加到 `events` 数组中。
  - `epfd`: 由 `epoll_create` 返回的 `epoll` 文件描述符。
  - `events`: 指向 `epoll_event` 结构体数组的指针，用于**指向就绪队列**。
  - `maxevents`: `events` 数组的大小，表示最多可以返回的事件数量。
  - `timeout`: 等待事件的超时时间，单位是毫秒。0 表示非阻塞模式，-1 表示无限等待。
- **返回值**: 成功时返回已经就绪的fd个数；失败时返回 -1，并设置 `errno`。

###  `epoll_event` 结构体
```cpp
struct epoll_event {
    uint32_t events;  // 
    epoll_data_t data; // 用户数据
};
```

![epoll结构](/images/epoll.png)

- **功能**: 定义 `epoll` 事件的类型和用户数据。
  - `events`: 事件类型 —— 位图的形式：
    - `EPOLLIN`: 可读事件。
    - `EPOLLOUT`: 可写事件。
    - `EPOLLERR`: 错误事件。
    - `EPOLLHUP`: 挂起事件。
    - `EPOLLET`: 边缘触发（ET）模式。
    - `EPOLLONESHOT`: 只触发一次。
  - `data`: 用户定义的数据，可以是一个指针或整数，用于在事件回调中识别事件源。
  
  
### epoll模型：

#### 优点

1. 检测就绪 O(1)，获取就绪O(n)
2. `fd`,`event`,没有上限
3. `epoll_wait`返回值n，表示有几个fd就绪了。并且就绪事件`events`是连续的。因为`events`是由**就绪队列一个一个弹出来的**。因此在**处理事件**的时候不需要考虑哪些`fd`是不可使用的了。

#### 简单来说，就是`红黑树`+`就绪队列`+`回调函数（不使用轮询检测提高了效率）` 

- **红黑树与文件描述符 (`fd`) 的关系**：
  - 红黑树这一结构的功能类似于`select`和`poll`中用户维护的数组。
  
  - 在 `epoll` 机制中，操作系统维护了一棵红黑树（`rb_tree`），用于存储注册到 `epoll` 实例中的文件描述符（`fd`）及其对应的事件信息。红黑树的关键字 (`key`) 是文件描述符 (`fd`)，它用于高效地查找和管理这些文件描述符。
	- 红黑树是一种自平衡的二叉查找树，能够保证插入、删除和查找操作的时间复杂度为 O(log N)。
    
#### 以下为回调函数的处理
- **数据从网卡到 `TCP` 接收缓冲区的过程**：
  
  - 当网卡接收到数据时，它通常会触发一个硬件中断。
  
  - 网卡驱动程序处理这个中断，并将数据搬运到内存中的网络接收缓冲区。
  
  - 接着，网卡驱动会调用协议栈的回调函数，将数据交付给协议栈的 `TCP` 接收缓冲区。
  
- **事件检测与红黑树的查找**：
  
  - 当 `TCP` 接收缓冲区中有数据可读时，内核会通过文件描述符 (`fd`) 在 `epoll` 的红黑树中查找对应的事件信息。
  
  - 在红黑树中存储的信息不仅包括文件描述符 (`fd`)，还包括这个文件描述符上注册的事件类型（如 `EPOLLIN`、`EPOLLOUT` 等）。
  
- **就绪节点的构建与插入**：
  
  - 如果检测到有事件发生（例如 `EPOLLIN` 表示有数据可读），内核会构建一个就绪节点（表示该 `fd` 现在处于就绪状态），并将该节点插入到 `epoll` 的就绪队列中。
  
  
#### 用户层面
- **用户从就绪队列获取事件**：
  
  - 用户程序调用 `epoll_wait` 时，`epoll` 会返回就绪队列中的节点，这些节点表示有事件发生的文件描述符。用户程序可以通过这些返回值处理相应的 I/O 操作。

![Epoll模型](/images/Epoll模型.png)

- 当我们创建一个 epoll 实例时，它会在内核中分配一个 `struct file`，该结构体用于表示 `epoll` 实例本身。

这意味着你可以将 **epoll 实例视为一个文件进行操作**。

### 使用示例

```cpp
#include <sys/epoll.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>

int main() {
    int epfd = epoll_create1(0);
    if (epfd == -1) {
        perror("epoll_create1");
        exit(EXIT_FAILURE);
    }

    struct epoll_event event;
    event.events = EPOLLIN; // 关注可读事件
    event.data.fd = STDIN_FILENO; // 关注标准输入
    if (epoll_ctl(epfd, EPOLL_CTL_ADD, STDIN_FILENO, &event) == -1) {
        perror("epoll_ctl");
        close(epfd);
        exit(EXIT_FAILURE);
    }

    struct epoll_event events[10];
    int nfds = epoll_wait(epfd, events, 10, -1); // 阻塞等待事件
    if (nfds == -1) {
        perror("epoll_wait");
        close(epfd);
        exit(EXIT_FAILURE);
    }

    for (int i = 0; i < nfds; ++i) {
        if (events[i].data.fd == STDIN_FILENO) {
            printf("Data available on stdin\n");
        }
    }

    close(epfd);
    return 0;
}
```