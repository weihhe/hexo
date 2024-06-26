title: vscode弹出console window
tags:
  - vscode
categories:
  - environment
author: W_HH
date: 2023-06-17 17:06:00
---
让vscode弹出熟悉的黑框
<!-- more -->

**以(.C)为例**

1. 新建一个空文件夹，作为工作台，并在子目录下创建一个 'project' 的文件夹和 'output' 的文件夹
  
2. 按 ' ctrl + ` ' 唤出 TERMINAL , 输入
  
  ```bash
  code .
  ``` 
3. 在project下创建一个HelloWorld.c文件
  
  ```c
  #include <stdio.h>
  
  int main()
  {
      printf("Hello,VScode");
      return 0;
  }
  ```
  
4. 选择 Run c/cpp file
  
5. 修改 'tasks.json' , 将编译好的可执行文件保存到output
  
  ![upload successful](/images/pasted-1.png)

6. 创建 'launch.json' 文件

 ![upload successful](/images/pasted-3.png)
  
7. 生成完毕后，选择 ‘Add configuration...’ 中的 launch

8. 修改  <u>"externalConsole"</u> , 为 true， <u>"program"</u> 和 <u> "miDebuggerPath"</u>
  
 ![upload successful](/images/pasted-4.png)
  
9. 完成。（在结束前设置断点便于使用 console window查看输出）