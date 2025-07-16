title: OpenGL创建窗口
author: weihehe
date: 2025-07-16 14:39:10
tags:
---
GLFW
<!--more-->

[原文链接](https://learnopengl-cn.github.io/01%20Getting%20started/02%20Creating%20a%20window/#glfw)

## GLFW

- `GLFW`是一个专门针对OpenGL的C语言库，它提供了一些渲染物体所需的最低限度的接口,允许用户创建OpenGL上下文、定义窗口参数以及处理用户输入。

- GLAD应该提供，两个头文件目录，和一个`glad.c`文件。将两个头文件目录（glad和KHR）复制到你的`Include`文件夹中（或者增加一个额外的项目指向这些目录），并添加`glad.c`文件到你的工程中

## GLAD

- 因为`OpenGL`只是一个标准/规范，具体的实现是由驱动开发商针对特定显卡实现的。由于驱动版本众多，它大多数函数的位置都无法在编译时确定下来，需要在运行时查询`GLAD`是一个开源的库，使用了一个在线服务。在这里我们能够告诉`GLAD`需要定义的OpenGL版本，并且根据这个版本加载所有相关的OpenGL函数。

### Visual studio环境下使用

1. 将`glfw3.lib`链接到工程，并把需要的头文件(GLAD)添加到`Visual studio`工程中。
![upload successful](/images/OpenGL-Link.png)

2. 链接
![filename already exists, renamed](/images/pasted-29.png)

3. 测试使用
```c++
#include <glad/glad.h>
#include <GLFW/glfw3.h>
#include <iostream>
int main()
{
    // 实例化GLFW窗口
    glfwInit();
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 3);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 3);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
    GLFWwindow* window = glfwCreateWindow(800, 600, "LearnOpenGL", NULL, NULL);
    glfwMakeContextCurrent(window);
    if (window == NULL)
    {
        std::cout << "Failed to create GLFW window" << std::endl;
        glfwTerminate();
        return -1;
    }
    while (!glfwWindowShouldClose(window))
    {
        glfwSwapBuffers(window);
        glfwPollEvents();
    }
    glfwTerminate();
    return 0;
}
```
	- `glfwCreateWindow`函数需要窗口的宽和高作为它的前两个参数。第三个参数表示这个窗口的名称（标题）
	- `glfwWindowHint`函数的第一个参数代表选项的名称，我们可以从很多以GLFW_开头的枚举值中选择；第二个参数接受一个整型，用来设置这个选项的值。
	- `glfwMakeContextCurrent` 是 GLFW 提供的一个函数，其作用是 将指定的 OpenGL 上下文（context）设置为当前线程的上下文，也就是说，你需要告诉 OpenGL：我要在这个窗口上进行绘图了。

## 视口

- OpenGL渲染窗口的尺寸大小，即视口(Viewport)，这样OpenGL才只能知道怎样根据窗口大小显示数据和坐标。
	- 我们可以通过调用glViewport函数来设置视口的尺寸(Dimension)
    ```c
    glViewport(0, 0, 800, 600);
    ```
	- glViewport函数前两个参数控制窗口左下角的位置。第三个和第四个参数控制渲染窗口的宽度和高度（像素）。

### 控制视口的回调函数

- 这个帧缓冲大小函数需要一个GLFWwindow作为它的第一个参数，以及两个整数表示窗口的新维度。每当窗口改变大小，GLFW会调用这个函数并填充相应的参数供你处理。
```c
void framebuffer_size_callback(GLFWwindow* window, int width, int height);
```


### 注册回调函数

- 告诉GLFW我们希望每当窗口调整大小的时候调用这个函数，才可以正常使用。
```c
glfwSetFramebufferSizeCallback(window, framebuffer_size_callback);
```
- 渲染循环初始化之前注册需要的回调函数。

## 控制输入

