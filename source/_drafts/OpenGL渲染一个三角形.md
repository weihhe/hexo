title: OpenGL渲染一个2D三角形
author: weihehe
tags:
  - 渲染
categories:
  - OpenGL
date: 2025-07-16 17:02:00
---

<!--more-->
[原文链接](https://learnopengl-cn.github.io/01%20Getting%20started/04%20Hello%20Triangle/)

## 渲染三角形：从顶点数据到图形输出

---

### 顶点定义

要在 OpenGL 中渲染一个三角形，需要指定 **三个顶点**，每个顶点包含一个 3D 位置。我们将这些顶点以 `float` 数组的形式写出，并使用**标准化设备坐标（NDC）**：

```cpp
float vertices[] = {
    -0.5f, -0.5f, 0.0f,  // 左下角
     0.5f, -0.5f, 0.0f,  // 右下角
     0.0f,  0.5f, 0.0f   // 顶部
};
```

* 由于我们渲染的是 **2D 三角形**，Z 坐标都设置为 `0.0f`，表示三角形位于同一深度面上，看起来是平面的。

---

### 深度（Depth）和可见性

* Z 坐标代表片段与观察者之间的距离，称为**深度**。
* 如果某个片段的 Z 值大于其他片段，它可能会被遮挡并被丢弃，以节省资源。
* 深度值的使用在\*\*深度测试（Depth Testing）\*\*中非常重要，用于决定哪个像素应最终显示。

---

### 标准化设备坐标（NDC）

* 在顶点着色器处理完顶点后，顶点的位置会被变换为 **标准化设备坐标**，范围是：

  ```
  x ∈ [-1.0, 1.0]
  y ∈ [-1.0, 1.0]
  z ∈ [-1.0, 1.0]
  ```

* 特点：

![upload successful](/images/三角形.png)
  * `(0, 0)` 是屏幕中心，而不是左上角。
  * `y` 轴正方向是**向上**。
  * 任何超出这个范围的顶点将被**裁剪（Clipping）**，不会显示。

---

### 视口变换（Viewport Transform）

* NDC 坐标接下来会通过 `glViewport()` 设置的参数被映射为 **屏幕空间坐标**。
* 屏幕空间坐标表示了像素在窗口中的实际位置。
* 最终这些坐标会生成**片段（Fragment）**，并传递给片段着色器进行颜色计算。

---

### 顶点缓冲对象（Vertex Buffer Object, VBO）

* 为了高效管理顶点数据，我们使用 **VBO** 来将顶点数据传输到 GPU：

  * 数据被存储在 GPU 的 **显存** 中。
  * 使用 VBO 可以**一次性传输多个顶点**，而不是每次一个，效率更高，数据传到 GPU 后，顶点着色器能**快速访问并处理**。。

- 我们可以使用glGenBuffers函数生成一个带有缓冲ID的VBO对象：
```cpp
unsigned int VBO;
glGenBuffers(1, &VBO);
```

#### 缓冲对象类型

OpenGL 中有多种缓冲对象类型，而**顶点缓冲对象（VBO）**使用的缓冲类型是：

```cpp
GL_ARRAY_BUFFER
```

OpenGL 支持**同时绑定多个不同类型的缓冲对象**，但每种类型一次只能绑定一个。例如，我们可以绑定一个 VBO 到 `GL_ARRAY_BUFFER` 目标：

```cpp
glBindBuffer(GL_ARRAY_BUFFER, VBO);
```

> 从此刻开始，所有作用于 `GL_ARRAY_BUFFER` 的操作都会影响这个绑定的 VBO。

---

####  将数据复制到缓冲区：`glBufferData`

使用 `glBufferData` 将顶点数据传输到当前绑定的缓冲对象（即 VBO）：

```cpp
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
```

##### 参数说明：

| 参数位置               | 含义              |
| ------------------ | --------------- |
| `GL_ARRAY_BUFFER`  | 目标缓冲类型（此处为顶点缓冲） |
| `sizeof(vertices)` | 数据大小（以字节为单位）    |
| `vertices`         | 要复制的实际数据        |
| `GL_STATIC_DRAW`   | 指定数据使用模式（见下）    |

---

##### 缓冲数据使用模式

OpenGL 需要知道你打算如何使用这份数据，以便将其放入最合适的显存区域。`glBufferData` 的第四个参数就是这个**使用模式**：

| 常量                | 适用场景   | 含义                    |
| ----------------- | ------ | --------------------- |
| `GL_STATIC_DRAW`  | 数据不会改变 | 最适合静态顶点数据，例如固定图形      |
| `GL_DYNAMIC_DRAW` | 数据偶尔更改 | 适合需要频繁更新的场景           |
| `GL_STREAM_DRAW`  | 每帧都更新  | 用于每次绘制都变化的数据，比如实时动画数据 |

>  由于三角形的位置数据是固定的，所以使用 `GL_STATIC_DRAW` 是最合适的选择。

以下是对你提供的内容整理后的版本，保留技术要点并使逻辑更清晰易懂：

---

###  顶点着色器（Vertex Shader）

顶点着色器是 OpenGL 可编程图形渲染管线中的**第一个着色器阶段**，也是**必须存在的着色器类型之一**（另一个是片段着色器）。它的作用是对每个顶点进行处理，主要任务是将 3D 空间中的顶点坐标变换为标准化设备坐标（NDC），为后续图形管线做准备。

---

####  顶点着色器示例

```glsl
#version 330 core
layout (location = 0) in vec3 aPos;

void main()
{
    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
}
```

####  代码详解

##### 1. `#version 330 core`

声明使用的 **GLSL（OpenGL Shading Language）版本**。
版本号与 OpenGL 对应，如：

* `#version 330 core` → OpenGL 3.3
* `#version 420 core` → OpenGL 4.2

> `core` 表示使用核心模式，不包含已废弃的旧功能。

##### 2. `layout (location = 0) in vec3 aPos;`

声明一个名为 `aPos` 的输入变量，用于接收传入的顶点位置。

* `layout(location = 0)`：明确绑定这个变量的位置索引为 0；
* `vec3`：表示这是一个包含 3 个浮点数的向量（即顶点的 x、y、z）。

##### 3. `gl_Position = vec4(...);`

`gl_Position` 是顶点着色器的内置输出变量，类型为 `vec4`，表示变换后的顶点坐标。

由于输入是 `vec3` 类型，需要转为 `vec4`，通常我们将 w 分量设置为 `1.0`：

```glsl
gl_Position = vec4(aPos, 1.0);
```

> 这个 `w = 1.0` 是为\*\*透视除法（Perspective Division）\*\*准备的，我们会在后续教程中深入讲解其意义。

---

###  向量（Vector）简介

* 在图形编程中，向量是描述**位置**、**方向**、**速度**等的重要数学工具；
* GLSL 中支持 `vec2`、`vec3`、`vec4`；
* 可以通过 `.x`、`.y`、`.z`、`.w` 获取其分量。

以下是你提供内容的整理版，清晰地分步骤说明了如何在 OpenGL 中**编译顶点着色器**：

---

###  编译顶点着色器（Vertex Shader）

在 OpenGL 中，使用着色器前必须先 **编译着色器源码** 并检查其是否成功。以下是编译过程的详细步骤。

---

####  1. 编写 GLSL 顶点着色器源码

我们将顶点着色器源码以 C 风格字符串的形式硬编码在程序中：

```cpp
const char *vertexShaderSource = "#version 330 core\n"
    "layout (location = 0) in vec3 aPos;\n"
    "void main()\n"
    "{\n"
    "   gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n"
    "}\0";
```

---

#### 2. 创建着色器对象

使用 `glCreateShader` 创建一个着色器对象，并指定它的类型（此处为顶点着色器）：

```cpp
unsigned int vertexShader;
vertexShader = glCreateShader(GL_VERTEX_SHADER);
```

---

#### 3. 附加着色器源码并编译

使用 `glShaderSource` 将源码附加到着色器对象，然后调用 `glCompileShader` 进行编译：

```cpp
glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
glCompileShader(vertexShader);
```

* 参数说明：

  * 第一个参数：要编译的着色器对象 ID；
  * 第二个参数：源码字符串的数量（这里只有一个）；
  * 第三个参数：源码字符串本身； 
  * 第四个参数：源码字符串长度（设置为 `NULL` 表示自动计算）。

---

#### 4. 检查编译是否成功

在编译之后，建议检查是否成功并输出错误信息（如果有）：

```cpp
int success;
char infoLog[512];
glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);
if (!success)
{
    glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);
    std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;
}
```

* `glGetShaderiv`：查询编译状态；
* `GL_COMPILE_STATUS`：指定获取的是编译状态；
* `glGetShaderInfoLog`：获取错误信息；
* `infoLog`：用于保存错误日志。




