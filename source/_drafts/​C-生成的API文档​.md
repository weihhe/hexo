title: ​C#生成的API文档​
author: weihehe
date: 2025-02-25 15:44:21
tags:
---
<!--more-->
[知乎](https://zhuanlan.zhihu.com/p/487063850)


以下是整理后的 Markdown 表格，补充了一些必要的内容以增强可读性：

### 2. 关键标签解析

| 标签/内容 | 作用 |
| --- | --- |
| **`<member>`** | 描述一个类、方法或属性的元数据。`name` 属性标识成员的完整路径（命名空间 + 类名 + 方法签名）。 |
| **`<member name="M:com.smart.core...">`** | `M` 表示方法（Method），后接完整方法签名（如 `createBepSolid`）。 |
| **`<summary>`** | 提供方法的功能描述。 |
| **`<param>`** | 提供方法参数的说明。例如：`<param name="polygons">面类集合</param>` 定义参数 `polygons` 的作用为“面类集合”（可能为几何图形集合）。 |
| **`<returns>`** | 提供方法返回值的说明。例如：`<returns>面类</returns>` 说明方法返回值的类型或用途（如返回几何面对象）。 |
| **`<assembly>`** | 声明该文档关联的程序集名称（图中内容存在乱码，推测实际应为 `com.smart.core` 等有效标识）。 |

