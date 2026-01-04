title: 安装.msi文件，报错2503 and 2502
author: weihehe
date: 2025-10-15 17:43:08
tags:
---
Window下的一些权限问题
<!--more-->

## 问题

### 安装一些`.msi`文件，可能出现的权限问题

- 例如：
![upload successful](/images/2502.png) 

## 解决方法

### 使用`PowerShell`
 ```shell
Start-Process msiexec.exe -ArgumentList '/i "your .msi path"' -Verb RunAs
```
- 将需要安装的`.msi`替换到路径当中。