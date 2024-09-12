title: VScode，C++基本配置
author: weihehe
tags:
  - VScode
categories:
  - Environment
date: 2024-06-12 12:38:00
---
VScode编辑器(不是编译器)
<!--more-->

1. 下载编译器,安装拓展包

	1. Remote - SSH - 远程登录Linux
	2. C/C++ - 必装
	3. C/C++ Extension Pack - C/C++扩展包，下载直接安装，它包含了 vscode 编写 C/C++ ⼯程需要的插件：例如（C/C++、C/C++ Themes、CMake、CMake Tools和Better C++ Syntax等），和以前⽐不需要⼀个个找了。
	4. C/C++ Themes - 主题设置，插件里面可以点击设置
	5. Chinese (Simplified) (简体中文)
	6. vscode-icons - 改变编辑器⾥⾯的⽂件图标
	7. filesize - 左下角显示文件大小的插件
	8. Include AutoComplete - 自动包含头文件
	9. GBKtoUTF8 - 自动将 GBK 转换为 UTF8

2. 配置环境变量(path当中添加bin文件夹的目录)，重启VScode
![环境变量添加成功](/images/环境变量添加成功.png)

3. 配置VScode编辑器选项。
![编译器选项](/images/编译器选项.png)

4. 配置任务，选择刚刚添加的编译器,生成的`.vscode`可以复用。
![配置任务](/images/配置任务.png)

5. 从终端中，生成任务，后按住ctrl + ·可呼出终端后使用`.\name.exe`执行。
![执行任务](/images/执行任务.png)

6. 设置可调试(`launch.json`文件)，并复用`"program": "${fileDirname}\\${fileBasenameNoExtension}.exe"`从task找到生成可执行文件的名称,并且指定`gdb.exe`所在路径，然后就可以愉快使用了，如果想要弹出黑窗的话，只需要修改  `“externalConsole”` , 为 true即可。
![launch.json文件](/images/launch.png)

## 部分变量函数

| 变量名 | 含义  |
| --- | --- |
| `${workspaceFolder}` | 当前打开的工作区文件夹的绝对路径。 |
| `${workspaceFolderBasename}` | 当前打开的工作区文件夹的基本名称，不包含路径。 |
| `${file}` | 当前打开文件的绝对路径文件名。 |
| `${relativeFile}` | 当前打开文件相对于工作区文件夹的相对路径名。 |
| `${fileBasename}` | 当前打开文件的基本名称，包含文件扩展名。 |
| `${fileBasenameNoExtension}` | 当前打开文件的基本名称，不包含文件扩展名。 |
| `${fileDirname}` | 当前打开文件所在的目录路径。 |
| `${fileExtname}` | 当前打开文件的扩展名。 |
| `${cwd}` | 启动 VSCode 的工作目录，或者是在 `tasks.json` 中为任务指定的当前工作目录。 |
| `${lineNumber}` | 当前激活文件中所选定的行号。 |
| `${selectedText}` | 当前激活文件中所选择的文本。 |
| `${execPath}` | VSCode 执行文件所在的目录。 |
| `${defaultBuildTask}` | 默认编译任务（build task）的名字。注意，这个变量可能不直接用于任务配置，但可用于引用默认任务。 |
| `${env:VARIABLE}` | 访问环境变量，其中 `VARIABLE` 是你想要访问的环境变量的名称。 |