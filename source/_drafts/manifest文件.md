title: .manifest文件
author: weihehe
date: 2025-02-25 14:27:24
tags:
---
一种 XML 格式的配置文件
<!--more-->
用于声明应用程序的元数据、依赖关系和系统兼容性要求。它的核心作用是告诉操作系统如何加载和运行应用程序，尤其是在处理 ​版本兼容性、权限控制、依赖库绑定等场景时至关重要。

## .manifest 文件的类型

### 嵌入式清单（Embedded Manifest）

- 编译时通过编译器选项（如 Visual Studio 的 Linker > Manifest File）将清单嵌入到可执行文件（.exe 或 .dll）中。
优先级高于外部清单文件。

### 外部清单文件（External Manifest）

- 独立保存为 YourApp.exe.manifest，与可执行文件同名且位于同一目录。
适用于未嵌入清单的旧程序或需要动态修改配置的场景。


4. 典型应用场景
----------------

### UAC 权限控制
若 DLL 需要访问受限资源（如注册表或系统目录），需通过 `<requestedExecutionLevel>` 提升权限（如 `requireAdministrator`）。

### 依赖版本绑定
若 DLL 依赖特定版本的 VC++ 运行时库，可在清单中声明：

```xml
<dependency>
  <dependentAssembly>
    <assemblyIdentity type="win32" name="Microsoft.VC90.CRT" version="9.0.21022.8" />
  </dependentAssembly>
</dependency>
```
