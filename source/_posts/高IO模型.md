title: 高IO模型
author: weihehe
tags:
  - IO
categories:
  - 操作系统
date: 2024-09-05 14:44:00
---
轮询，复用
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


### 示例

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/select.h>
#include <sys/time.h>

int main() {
    fd_set readfds;
    struct timeval timeout;
    int retval;

    // 设置超时时间
    timeout.tv_sec = 5;  // 等待5秒
    timeout.tv_usec = 0;

    // 清空文件描述符集合
    FD_ZERO(&readfds);

    // 添加标准输入到集合中
    FD_SET(STDIN_FILENO, &readfds);

    // 等待标准输入可读
    retval = select(STDIN_FILENO + 1, &readfds, NULL, NULL, &timeout);

    if (retval == -1) {
        perror("select()");
        exit(EXIT_FAILURE);
    } else if (retval) {
        printf("Data is available on stdin.\n");
    } else {
        printf("No data within five seconds.\n");
    }

    return 0;
}
```
```cpp
/*select 和 accpet相互配合的伪代码代码*/
int listensock = _listensock.Fd(); // 监听套接字
fd_array[0] = listensock;          // 记录监听套接字
for (;;)
{
    fd_set rfds;
    FD_ZERO(&rfds);

    int maxfd = fd_array[0];
    for (int i = 0; i < fd_num_max; i++) // 第一次循环
    {
        if (fd_array[i] == defaultfd) // 当前fs已经被使用过了
            continue;
        FD_SET(fd_array[i], &rfds); // 没有使用过，则记录当前fd
        if (maxfd < fd_array[i])
        {
            maxfd = fd_array[i];
            cout << "max fd update, max fd is: %d", maxfd; // 找到最大的fd，便于select使用
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
        cout << "get a new link" << endl;
        Accepter();//注意，这里实际还需要处理多个就绪的问题 
        break;
    }
}

void Accepter()
{
    // 我们的连接事件就绪了
    std::string clientip;
    uint16_t clientport = 0;
    int sock = _listensock.Accept(&clientip, &clientport);
}

```
### 解释

1. **清空文件描述符集合**: 使用 `FD_ZERO` 清空 `readfds` 集合。
2. **添加文件描述符**: 使用 `FD_SET` 将标准输入（`STDIN_FILENO`）添加到集合中。
3. **设置超时时间**: `timeout` 结构设置为 5 秒。
4. **调用 `select`**: `select` 将阻塞，直到标准输入上有数据可读，或者超时。


