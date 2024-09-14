title: Redis
author: weihehe
date: 2024-09-13 16:14:34
tags:
---

Redis常用技巧
<!--more-->
- **RDB（Redis Database）文件**：
  - RDB是Redis的一种持久化方式，它将Redis在某一时刻的内存快照（即内存中的数据）保存到磁盘上。这个过程通常是在后台异步进行的，不会阻塞Redis的主线程。
  - RDB文件的主要用途是在Redis重启时，能够快速地恢复数据库到之前的某个状态。它并不直接参与Redis的运行时数据处理，因此它的存在与否不影响Redis系统的正确性，但可以提高数据恢复的效率和速度。
- **AOF（Append Only File）日志**：
  - AOF是另一种Redis的持久化方式，它记录了Redis执行的每一条写命令。随着Redis的运行，AOF文件会不断增长。
  - 当Redis重启时，它会重新执行AOF文件中的命令来恢复数据。与RDB相比，AOF提供了更好的数据持久性保证，但恢复过程可能会相对较慢，因为它需要执行大量的命令。