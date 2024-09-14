title: protobuf
author: weihehe
date: 2024-09-13 20:46:46
tags:
---

PB的安装与使用，什么叫PB
<!--more-->

## protobuf的安装

- [链接](https://github.com/protocolbuffers/protobuf/releases)

### Window下直接去下载对应版本的压缩包，然后将解压的文件中的`bin`添加到环境变量`PATH`中即可。

- 成功验证方法
![PB安装成功](/images/window安装PB成功.png)

### 在 Linux 下的安装

**需要安装依赖库——`autoconf automake libtool curl make g++ unzip`**
- Ubuntu下的安装方法:
`sudo apt-get install autoconf automake libtool curl make g++ unzip -y`

#### 直接自动安装`sudo apt-get install protobuf-compiler libprotobuf-dev`

#### 下载文件安装
1. **进入解压好的文件目录**：  
  确保已经下载了ProtoBuf的源代码并解压到了某个目录，然后使用`cd`命令进入该目录。
  
2. **执行`autogen.sh`（如果适用）**：  
  如果下载的是包含多种语言的通用源代码包，需要执行`autogen.sh`脚本来生成配置文件。如果 下载的是针对特定语言的包，这一步可能不需要。
  
  ```sh
  ./autogen.sh
  ```
  
3. **执行`configure`脚本**：  
  有两种执行方式， 可以选择其中一种：
  
  - **默认安装**：  
    这将把ProtoBuf默认安装在`/usr/local`目录下，各个组件（如库文件和可执行文件）会分散在相应的子目录中。
    
    ```sh
    ./configure
    ```
    
  - **自定义安装目录**：  
    如果 希望将ProtoBuf统一安装在一个特定的目录下，比如`/usr/local/protobuf`，可以使用`--prefix`选项来指定安装路径。
    
    ```sh
    ./configure --prefix=/usr/local/protobuf
    ```
    
4. **编译和安装**：  
  在成功执行`configure`脚本后，接下来通常需要使用`make`命令进行编译，然后使用`make install`命令进行安装。
  
  ```sh
  make  sudo make install
  ```
  - 如果安装失败，可能是交换区大小存在问题，使用`make check`可以调整查看。
5. **验证安装**：  
  - 安装完成后，可以通过检查安装目录或使用`protoc --version`命令来验证ProtoBuf是否安装成功。


## 概念：

### 序列化和反序列化

- 序列化：把对象转换为字节序列的过程 称为对象的序列化。

- 反序列化：把字节序列恢复为对象的过程称为对象的反序列化。

- 什么情况下需要序列化

	- 存储数据:当你想把的内存中的对象状态保存到一个文件中或者存到数据库中时。
	- 网络传输:网络直接传输数据，但是无法直接传输对象，所以要在传输前序列化，传输完成后反序列化成对象。







