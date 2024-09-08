title: LT模式和ET模式 文件描述符非阻塞
author: weihehe
tags:
  - 'LT '
  - ET
categories:
  - 网络
date: 2024-09-08 21:45:00
---
水平触发（Level Triggred）和边缘触发（Edge Triggred）
<!--more-->

## 1. 水平触发（Level Triggered, LT） —— 默认方式

- **行为**: 在 LT 模式下，只要文件描述符上有数据可读（可写），内核就会不断地通知应用程序，直到应用程序处理完所有数据。这意味着，即使你在第一次收到通知时没有读取数据，内核在下次 `epoll_wait` 或其他 I/O 复用调用中仍会通知你。
- **特点**: 
  - 易于使用：因为内核会持续通知你，你不必担心错过事件。
  - 需要处理所有就绪的事件：在读取或写入数据之前，内核会一直通知你。
  - 事件到来，如果上层不处理，高电平，数据一直有效。

- **适用场景**: 当你不想错过任何数据并且希望内核持续通知时，使用 LT 是一个很好的选择。

- **代码示例**:
  ```cpp
  struct epoll_event event;
  event.events = EPOLLIN;  // 默认是 LT 模式
  epoll_ctl(epfd, EPOLL_CTL_ADD, fd, &event);
  ```

## 2. 边缘触发（Edge Triggered, ET）

- **行为**: 在 ET 模式下，内核只在文件描述符状态发生变化时通知应用程序一次。这意味着如果你在收到通知后没有及时读取或写入数据，下次即使有数据可读（可写），内核也不会再通知你，除非状态再次发生变化。
- **特点**: 
  - 高效：减少了内核与用户空间之间的交互频率，适合高效处理大量 I/O 事件。
  - 需要更复杂的处理逻辑：应用程序必须一次性处理所有可用数据，否则可能会错过通知。
  - ET:数据或者连接，当数据量发生变化的时候，才会通知我们**一次** —— 如果数据没有一次取完，并且对应事件没有后续在对数据的处理，那么数据可能会丢失。
	- 通知效率更高，IO效率也会变高，因为它对于数据比较严格的处理，要求每次通知都需要把本轮的数据都取走。 
		- 如何将本轮数据都取走？循环读取，直到读取出错。不过诸如`read`等接口在没有数据的时候，会阻塞住。我们不想让这种情况发生，于是我们需要让接口设置为非阻塞。

- **代码示例**:
  ```cpp
  struct epoll_event event;
  event.events = EPOLLIN | EPOLLET;  // 设置 ET 模式
  epoll_ctl(epfd, EPOLL_CTL_ADD, fd, &event);
  ```

### 举例

假设某个文件描述符变得可读：

- 在 **LT 模式** 下，只要文件描述符上有数据可读，每次 `epoll_wait` 都会通知你，直到你把所有数据读完为止。

- 在 **ET 模式** 下，当数据第一次变得可读时，会通知你一次。如果你没有读完所有数据，下次再调用 `epoll_wait` 时不会再收到通知，除非有新的数据到达。

## 设置非阻塞

1. 使用 `fcntl` 获取当前文件描述符的标志。

2. 将文件描述符的标志设置为非阻塞模式。

3. 使用 `fcntl` 将修改后的标志重新应用到文件描述符上。

### 代码示例

```cpp
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>
#include <errno.h>

int main() {
    int fd = /* your file descriptor, e.g., socket, pipe, etc. */;

    // 获取当前文件描述符的标志
    int flags = fcntl(fd, F_GETFL, 0);
    if (flags == -1) {
        perror("fcntl(F_GETFL) failed");
        return -1;
    }

    // 设置文件描述符为非阻塞模式
    flags |= O_NONBLOCK;
    if (fcntl(fd, F_SETFL, flags) == -1) {
        perror("fcntl(F_SETFL) failed");
        return -1;
    }

    // 现在你可以执行非阻塞的 read 操作了
    char buffer[1024];
    ssize_t n = read(fd, buffer, sizeof(buffer));
    if (n == -1) {
        if (errno == EAGAIN || errno == EWOULDBLOCK) {
            // 处理没有数据可读的情况
            printf("No data available now, try again later\n");
        } else {
            perror("read failed");
        }
    } else if (n == 0) {
        // 处理文件结束符（EOF）
        printf("EOF reached\n");
    } else {
        // 处理读取的数据
        printf("Read %zd bytes: %s\n", n, buffer);
    }

    return 0;
}

```

- **`fcntl(fd, F_GETFL, 0)`**: 获取文件描述符的当前标志。
- **`flags |= O_NONBLOCK`**: 将文件描述符的标志设置为非阻塞模式。
- **`fcntl(fd, F_SETFL, flags)`**: 应用新的标志设置。

在非阻塞模式下，`read` 操作如果没有数据可读，会立即返回 `-1`