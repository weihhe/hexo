title: Linux中的权限管理
author: weihehe
tags:
  - 权限
categories:
  - 操作系统
date: 2024-07-17 11:42:00
---
权限，组，命令
<!--more-->
## 用户组

Linux 系统中的目录和文件的访问身份分为 user, group, others, all，分别简写为 u, g, o, a。

- **user (u)**: 文件的所有者
- **group (g)**: 文件所有者所在组的其他成员
- **others (o)**: 不在所有者所在组的其他用户
- **all (a)**: 代表所有用户

当某个用户创建一个文件后，这个文件的所有者为该用户，所在组则为该用户所在的组（直至所有者或所在组被修改）。

## 组命令

修改文件属性的常用命令：

- `chgrp`
- `chown`
- `chmod`
- `umask`

### chgrp

`chgrp` 命令用于变更文件或目录的所属群组。

```sh
chgrp [OPTION]… GROUP FILE…
chgrp [参数]… [所在组] 文件或目录
```

将文件的所在组修改为 `www`：

```sh
chgrp www install.sh
```

### chown

`chown` 命令用于改变文件拥有者。

```sh
chown [OPTION]… [OWNER][:[GROUP]] FILE…
chown [参数]… [所有者][:[所在组]] 文件或目录
```

将文件的所有者及所在组修改为 `www`：

```sh
chown www:www install.sh
```

### chmod

Linux/Unix 的文件调用权限分为三级：文件拥有者、群组、其他。利用 `chmod` 可以控制文件如何被他人调用。

使用权限: 所有使用者。

```sh
chmod [OPTION]… MODE[,MODE]… FILE…
chmod [参数]… [模式][,模式] 文件或目录
```

详细操作见下方文件权限。

## umask

`umask` 用来设定**权限掩码。权限掩码**是由 3 个八进制的数字组成，将现有的权限**减掉权限掩码**后，即可产生建立文件时预设的权限。

```sh
umask [-S][权限掩码]
```

当前的存取权限：

```sh
umask -S
u=rwx,g=rwx,o=rx
```
假设`umask` 为 002，减掉权限掩码后，当前的存取权限即为 775 (u=rwx, g=rwx, o=rx)。


## 文件权限


在 Linux 中可以使用 `ll` 或 `ls -l` 命令来显示一个文件的属性以及文件所属的用户和组，如：

```sh
$ ls -l
total 64
dr-xr-xr-x   2 root root 4096 Dec 14  2018 bin
dr-xr-xr-x   4 root root 4096 Apr 19  2018 boot
```

第一个字符代表这个文件的类型（如目录、文件或链接文件等）：
- 当为 `[d]` 则是目录
- 当为 `[-]` 则是文件
- 若是 `[l]` 则表示为连结档 (link file)
- 若是 `[b]` 则表示为装置文件里面的可供储存的接口设备 (可随机存取装置)
- 若是 `[c]` 则表示为装置文件里面的串行端口设备，例如键盘、鼠标 (一次性读取装置)

接下来的字符中，以三个为一组，且均为『rwx』的三个参数的组合：
- [r] 代表可读 (read)
- [w] 代表可写 (write)
- [x] 代表可执行 (execute)
- [-] 代表无权限

### 权限分组：
- 第一组为『文件拥有者的权限』
- 第二组为『同群组的权限』
- 第三组为『其他非本群组的权限』

### 修改文件权限

修改文件的权限用命令 `chmod` 来执行，有两种权限定义方式：数字方式和符号方式。

### 数字方式修改文件权限

权限值对应：
- read (r): 4
- write (w): 2
- execute (x): 1

权限设置为：
- 所有者可读可写可以执行
- 同组可读可执行不可写
- 其他组的用户可读

此时符号权限为 `-rwxr-xr--`，则权限的分数应为 `[4+2+1][4+0+1][4+0+0]=754`

```sh
chmod 754 install.sh
```

## 符号方式修改文件权限（直接使用`+`,`-`,`=`）

权限设置为 `-rwxr-xr--`，同上数字方式：

```sh
chmod u=rwx,g=rx,o=r install.sh
```

其他组用户删除可读权限：

```sh
chmod o-r install.sh
```

所有用户增加执行权限：

```sh
chmod a+x install.sh
```
### 特殊权限
- setuid
- setgid
- sticky bits
---
- [Linux 用户组及文件权限详解](https://tillend.github.io/2019/02/26/linux-group-authority/)

- [Linux权限控制的基本原理](https://gist.github.com/baymaxium/45ad3b68bcd392c3c3feebf935f1618f)