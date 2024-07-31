title: 生产者消费者问题（Producer-consumer problem）
author: weihehe
tags: []
categories:
  - 多线程
date: 2024-07-30 10:43:00
---
有限缓冲问题（Bounded-buffer problem）
<!--more-->

## 概念


![生产消费模型](/images/生产-消费模型.png)
- 为何要使用生产者消费者模型生产者消费者模式就是**通过一个容器来解决生产者和消费者的强耦合问题**。
	- 生产者和消费者彼此之间不直接通讯，而通过阻塞队列来进行通讯。
	- **生产者生产完数据之后不用等待消费者处理，直接扔给阻塞队列，消费者不找生产者索要数据，而是直接从阻塞队列里取**，阻塞队列就相当于一个**缓冲区**，平衡了生产者和消费者的处理能力。这个阻塞队列就是用来给生产者和消费者**解耦**的。

- 生产者根据数据来源**生成数据**与消费者**加工数据都是需要耗费资源的**，**多生产者多消费者的意义就是提高这方面的效率**。
    
## 关系

- **生产者** 与 **生产者**:互斥。
- **消费者** 与 **消费者**:互斥。
- **生产者** 与 **消费者**：互斥（保证数据安全），同步——保证它们在访问共享资源的时候具有一定的顺序性。（因为它们都需要访问`共享资源`）

## 总结：

- **3**种关系。
- **2**种角色。
- **1**个用于生产者和消费者交易场所——特定结构的内存空间。


## 构建

1. 基于`BlockingQueue`的生产者消费者模型。

	- 在多线程编程中阻塞队列(Blocking Queue)是一种常用于实现生产者和消费者模型的数据结构。
	- 其与普通的队列区别在于，当队列为空时，从队列获取元素的操作将会被阻塞，直到队列中被放入了元素；当队列满时，往队列里存放元素的操作也会被阻塞，直到有元素从队列中被取出
	- 上述的操作都是基于不同的线程来说的，线程在对阻塞队列进程操作时会被阻塞

使用条件变量实现简单生产者-消费者模型的示例：
```cpp
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>

#define BUFFER_SIZE 10

int buffer[BUFFER_SIZE];
int count = 0;
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

void* producer(void* arg) {
    for (int i = 0; i < 20; ++i) {
        pthread_mutex_lock(&mutex);

        while (count == BUFFER_SIZE) {
            // 等待消费者消费
            pthread_cond_wait(&cond, &mutex);
        }

        // 生产数据
        buffer[count++] = i;
        printf("Produced: %d\n", i);

        // 唤醒消费者
        pthread_cond_signal(&cond);
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

void* consumer(void* arg) {
    for (int i = 0; i < 20; ++i) {
        pthread_mutex_lock(&mutex);

        while (count == 0) {
            // 等待生产者生产
            pthread_cond_wait(&cond, &mutex);
        }

        // 消费数据
        int item = buffer[--count];
        printf("Consumed: %d\n", item);

        // 唤醒生产者
        pthread_cond_signal(&cond);
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

int main() {
    pthread_t prod_thread, cons_thread;

    // 创建生产者和消费者线程
    pthread_create(&prod_thread, NULL, producer, NULL);
    pthread_create(&cons_thread, NULL, consumer, NULL);

    // 等待线程完成
    pthread_join(prod_thread, NULL);
    pthread_join(cons_thread, NULL);

    // 销毁互斥锁和条件变量
    pthread_mutex_destroy(&mutex);
    pthread_cond_destroy(&cond);

    return 0;
}
```

2. 基于`基于环形队列`的生产者消费者模型。
	- 当消费者和生产者**不指向同一个位置的时候，才时访问**，其余情况都可以并发访问。
    
	- 当队列为空，需要生产者先生产,列满了也就没有剩余空间，无法再继续生产。消费者同理。



## 伪唤醒

伪唤醒（Spurious Wakeup）是指在多线程编程中，资源条件已不满足，但线程被唤醒的现象。这种现象在使用条件变量（condition variable）进行线程同步时可能会发生。

![伪唤醒](/images/生产消费问题-伪唤醒.png)
为了防止这种情况，我们需要使用while（`条件判断`）的方法。