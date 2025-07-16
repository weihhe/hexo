title: QT基础与自动生成的Form文件
author: weihehe
date: 2025-01-12 15:53:55
tags:
---
QT的安装和使用
<!--more-->
在PATH路径下，添加QT的路径

- 1.最可能是属性浏览器控件（比如 QtVariantPropertyManager / QtVariantEditorFactory）内部，当用户点击这个 QVariant::Color 类型的属性项时，自动弹出 QColorDialog。

# 前端开发

1. 桌面应用开发
  
2. 网页前端开发（主要应用）
  
3. 移动应用开发
  

TUI 命令行/终端界面开发 —— 专业软件，例如(gcc , gdb)

GUI 图形化界面 —— 面向普通用户

# Windows下的GUI方案

1. Windows API
  
2. MFC,VC6.0
  
3. Qt
  
4. Windows Forms 给 C#(.net)量身打造的一套技术体系
  
5. WPF
  
6. UWP
  
7. Electron 本质上是将HTML这样的页面，打包成一个Windows上运行的客户端程序
  

# QT

- 跨平台（Windows / Linux / Mac）的图形用户界面应用程序框架
- Qt Creator会自动维护项目的文件结构

### Qt 开发环境的三个必要安装部分

1. **C++ 编译器** ：如 gcc、cl.exe 等。编译器不等同于 IDE，它是 IDE 调用的一个程序。
2. **Qt SDK** ：软件开发工具包。Windows 版本的 Qt SDK 内置了 C++ 编译器（通常是 MinGW，即 Windows 版本的 gcc/g++）。如果想使用 Visual Studio 内置的 cl.exe 作为编译器，虽然可行，但需要配置很多额外的东西，容易出错。在安装过程中，需要将对应的 C++ 编译器一起勾选上。
3. **Qt 的集成开发环境（IDE）** ：
  - **Qt Creator** ：这是 Qt 官方提供的 IDE，最容易入门和上手，开箱即用，无需额外配置。虽然使用过程中可能存在一些影响体验的 bug，但整体来说比较方便，适合初学者。
  - **Visual Studio** ：功能强大，但需要更多的额外配置，出错概率较高。一些公司开发商业 Qt 程序时可能会选用 Visual Studio。需要给 Visual Studio 安装 Qt 插件，并且要把 Qt SDK 使用 Visual Studio 的编译器重新编译（现在也有预编译好的版本，方便了一些）。
  - **Eclipse** ：Eclipse 是一个 IDE 平台，可以搭配不同插件构成不同的 IDE。目前市场份额受到冲击，有重量级的 JetBrains 工具和轻量级的 VSCode 等竞争。

### 安装说明

实际上，安装 Qt SDK 后，通常会自带 C++ 编译器和 Qt Creator IDE，因此只需安装一个 Qt SDK，另外两个也就都有了，类似于 Java 搭建环境时的 JDK + IDE。

以下是图片中内容的整理：

### Qt SDK 自带的工具程序

1. **Assistant** ：Qt 自带的离线版本的官方文档。
2. **Designer** ：Qt 设计师，是图形化设计界面的工具。通过拖拽控件的方式可以快速生成界面。
3. **Linguist** ：Qt 语言家，作用是对国际化进行支持。
4. **Qt Creator** ：Qt 的集成开发工具。

# 框架

1. 本质上是一群大佬发明出来的一套技术规范，用于规定代码，已经写好了一些代码，然后让我们去使用。

# Qt代码结构

Qt Creator 会自动维护项目的文件结构，主要体现在以下几个方面：

### 自动生成项目文件

当通过项目向导创建项目时，Qt Creator 会自动生成项目所需的头文件、源文件及项目文件（.pro）等。

### 自动更新项目管理文件

- **添加文件时** ：当向项目中添加新文件，如头文件、源文件或界面文件等，Qt Creator 会自动将这些文件添加到项目管理文件（.pro 文件）中对应的变量列表，如 HEADERS、SOURCES 或 FORMS 等后面，无需用户手动修改 .pro 文件。
- **删除文件时** ：从项目中删除文件时，Qt Creator 也会自动将对应的文件从 .pro 文件中移除。

### 自动分类管理文件

在项目管理界面，Qt Creator 会将项目中的文件自动分类到不同的分组中，如头文件分组（Headers）、源文件分组（Sources）和界面文件分组（Forms）等，方便用户查看和管理，但在实际文件系统中可能并不会自动创建对应的文件夹。用户也可以通过创建子目录路径或使用 pri 文件等方式，进一步细化项目的文件结构，Qt Creator 能较好地识别和管理这些自定义的结构。

## ## 代码部分 (`main.cpp`)

```cpp
#include <QApplication>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    Widget w;
    w.show();
    return a.exec();
}
```

### 注释解释

- **`main` 函数形参** ：`main` 的形参 `argc` 和 `argv` 是命令行参数，程序运行时可以通过命令行传递参数给程序。
- **`QApplication` 对象** ：编写一个 Qt 的图形化界面程序，必须要有 `QApplication` 对象。`QApplication a(argc, argv);` 创建了一个 `QApplication` 对象 `a`，它管理与 GUI 程序相关的资源和初始化设置，并且需要传递命令行参数 `argc` 和 `argv` 给它。
- **`Widget` 对象** ：`Widget w;` 创建了一个 `Widget` 对象 `w`，`Widget` 是在创建项目时填写的生成的类名，其父类是 `QWidget`。`QWidget` 是所有用户界面对象的基类，提供了基本的用户界面功能，如绘制、事件处理等。
- **`.show()` 方法** ：`w.show();` 用于将 `Widget` 对象显示出来。`QWidget` 提供了 `.show()` 方法让控件显示，以及 `.hide()` 方法让控件隐藏，通过这些方法可以控制控件的可见性。
- **`exec()` 方法** ：`return a.exec();` 中的 `exec()` 方法表示让程序执行起来。它启动了应用程序的事件循环，开始处理各种事件，如用户的鼠标点击、键盘输入等，从而使程序能够响应用户的操作并正常运行。**这里的exec（）和操作系统中的程序替换，没有任何关系。**

```cpp
// widget.h
#ifndef WIDGET_H
#define WIDGET_H

#include <QWidget>

QT_BEGIN_NAMESPACE
namespace Ui {
class Widget;
}
QT_END_NAMESPACE

class Widget : public QWidget
{
    Q_OBJECT

public:
    Widget(QWidget *parent = nullptr);
    ~Widget();

private:
    Ui::Widget *ui;
};
#endif // WIDGET_H
```

### 文件头部

- **头文件保护** ：`#ifndef WIDGET_H` 和 `#define WIDGET_H` 作为头文件保护，确保头文件只被包含一次，避免重复定义。注释中提到更推荐使用 `#pragma once`，它作用相同，但书写更简洁。
- **包含头文件** ：`#include <QWidget>` 表示包含 `QWidget` 类的头文件，因为 `Widget` 类继承自 `QWidget`，需要使用其功能。

### 命名空间

`QT_BEGIN_NAMESPACE` 和 `QT_END_NAMESPACE` 用于定义一个命名空间范围，`namespace Ui { class Widget; }` 在此命名空间中声明了一个与 `Widget` 类同名的类，这通常是用于与 Qt Designer 生成的 UI 文件相关联的代码。

### 类定义

- **继承关系** ：`class Widget : public QWidget` 定义了一个名为 `Widget` 的类，它继承自 `QWidget` 类，继承方式为公有继承，这样 `Widget` 类就可以使用 `QWidget` 的公有成员和保护成员。
- **`Q_OBJECT` 宏** ：`Q_OBJECT` 是一个 Qt 内置的宏，本质是文本替换。它的作用是使类能够使用 Qt 的元对象系统，该系统提供了信号和槽机制、运行时类型信息等重要功能。展开后会生成大量代码，与类的信号和槽等功能相关。

### 构造函数和析构函数

- **构造函数** ：`explicit Widget(QWidget *parent = nullptr)` 是构造函数，带有参数 `parent`，用于指定父对象，默认为 `nullptr`。这样可以方便地将创建的 Qt 对象挂到对象树上，指定其父节点，构建对象树结构。
- **析构函数** ：`~Widget();` 是析构函数，用于在对象生命周期结束时清理资源。

### 私有成员变量

`Ui::Widget *ui;` 是一个指向 `Ui::Widget` 类型的指针，`Ui::Widget` 类与 form 文件密切相关，通常是由 Qt Designer 生成的代码，用于管理界面中的控件。

### 文件尾部

`#endif // WIDGET_H` 与文件头部的 `#ifndef WIDGET_H` 和 `#define WIDGET_H` 配合，构成完整的头文件保护。

### 注释说明

- **头文件包含原则** ：Qt 的设定中，使用内置类时，包含的头文件名字通常和类名一致，如使用 `QWidget` 需包含 `<QWidget>`。但在 C++ 中，头文件可能被 “间接包含”，即引入一个头文件，而该头文件又包含了其他头文件，此时就相当于间接包含了多个头文件。
- **对象树机制** ：Qt 引入了对象树机制，一个节点可以有 N 个子节点，但只能有一个父节点，对象树是一个普通的 N 叉树（不是二叉树）。创建 Qt 对象时，可将其挂到对象树上，挂的时候需指定父节点。
- **信号和槽机制** ：`Q_OBJECT` 宏与 Qt 的信号和槽机制密切相关，如果某个类想使用信号和槽，就需要引入这个宏。信号和槽是 Qt 中一个非常核心的机制，用于对象之间的通信。

```cpp
// widget.cpp
#include "widget.h"
#include "./ui_widget.h"

Widget::Widget(QWidget *parent)
    : QWidget(parent)
    , ui(new Ui::Widget)
{
    ui->setupUi(this);
}

Widget::~Widget()
{
    delete ui;
}
```

### 构造函数

- `Widget::Widget(QWidget *parent)` 是 `Widget` 类的构造函数。
- `: QWidget(parent)` 调用父类 `QWidget` 的构造函数，并将 `parent` 作为父对象传递给父类构造函数。
- `, ui(new Ui::Widget)` 创建一个 `Ui::Widget` 类型的对象，并将其地址赋值给成员变量 `ui`。
- 在函数体内调用 `ui->setupUi(this);`，用于将 form 文件生成的界面和当前 `Widget` 对象关联起来。

### 析构函数

- `Widget::~Widget()` 是 `Widget` 类的析构函数。
- 在析构函数体内执行 `delete ui;`，用于释放 `ui` 指针所指向的对象，避免内存泄漏。

### 注释解释

- `Widget` 类的构造函数通过初始化列表创建父类对象和 `ui` 对象，并在函数体内调用 `setupUi` 方法，将 form 文件生成的界面与当前 `Widget` 对象关联。
- 析构函数负责释放 `ui` 对象所占用的内存。

### Form文件

#### Qt 中创建图形化界面的程序,有两种方式：

1. 直接通过 c++ 代码的方式创建界面
  
2. 通过 form file, 以图形化的方式来生成界面此时, 就可以使用 Qt Designer 或者直接使用 Qt Creator 来编辑这个 ui 文件从而以图形化的方式快速方便的生成图形界面![filename already exists, renamed](/images/pasted-25.png)

- 点击左侧边栏的 “编辑” 按钮后显示的内容就是 `.ui` 文件的本体，其格式为 XML。
  
- XML 和 HTML 非常类似，都使用成对的标签来表示数据。
  
- XML 中的标签由程序员自定义，而此处 `.ui` 文件中的标签是开发 Qt 的人员约定的，使用者不需要关注具体含义，只需知道 `.ui` 文件本质上是一个 XML 即可。
  
- HTML 的标签是固定的，有明确的标准和含义，所有浏览器按照同样的规则来解释这些标签。
  

`.ui` 文件是 Qt 用于存储界面设计的文件，本质是 XML 格式，通过特定的标签来描述界面元素及其属性等信息，由 Qt Designer 生成和编辑，开发者通常不需要直接修改其内容。

### 项目运行后生成的目录

- 在运行程序后，项目目录并列的位置会多出一个以 “build -” 开头的目录，如 “build - Empty - Desktop - Qt_5_14_2_MinGW_64_bit - Debug”，里面存放项目运行时生成的临时文件。

### 目录内容及文件

- **Makefile 和相关文件** ：编译 Qt 程序时会用到 Makefile，但无需手动编写，由 qmake 自动生成。目录中有 “Makefile”，还有 “Makefile.Debug” 和 “Makefile.Release” 等文件。
- **ui_widget.h 文件** ：由 widget.ui 文件的 XML 内容生成的头文件，是 Qt 自动生成的代码的一部分。

### 自动生成的代码

- 包含 “ui_widget.h” 后，会引入自动生成的 `Ui::Widget` 类代码。其中 `setupUi` 方法用于根据 UI 文件设置界面细节，包括设置窗口大小、标题等。
- 在项目的 Widget 类的构造函数中，通过调用 `ui - > setupUi(this)`，将自动生成的界面设置应用到当前的 Widget 对象上。

### 可执行文件

- “Empty.exe” 是最终生成的可执行程序，直接运行它与在 Qt Creator 中运行项目的效果相同。
  
  ### 应用程序类型
  
  - **Qt Widgets Application** ：用于创建传统的桌面 GUI 应用程序，开发方式称为 Qt Widget。如果使用 Qt 编写 GUI 程序，通常选择这种方式。
  - **Qt Console Application** ：用于创建控制台应用程序。
  - **Qt for Python** ：包括空项目和窗口项目，表明 Qt 支持 Python 语言。
  - **Qt Quick Application** ：包括多种类型，如空、滚动、堆栈、滑动等。Qt Quick 是 Qt 推出的一套新的 GUI 开发方式，适用于现代动态界面开发，常与 QML 语言配合使用。
  
  ### 元编程技术
  
  通过代码生成代码，Qt 在编译时会调用生成工具，基于开发者代码生成其他 C++ 代码，最终编译的是这些生成代码。