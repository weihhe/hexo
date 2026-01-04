title: XAML（AvaloniaUI）中的一些概念
author: weihehe
date: 2025-08-07 15:04:19
tags:
---
<!--more-->
如果没有行号，从文本编辑器->所有语言->行号，取消然后再次勾选

```xaml

```
















```csharp
public class MessageService : IMessageService
{
    public void ShowMessage(string message)
    {
        MessageBox.Show(message);
    }
}

// 创建 VM 时注入
var vm = new MyViewModel(new MessageService());
this.DataContext = vm;
```
# 2.`vm`和`DataContext`,让 View 和 ViewModel 关联起来

- `vm` 是`ViewModel`层类对象的一个实例，负责处理业务逻辑、数据绑定、命令（如按钮点击的操作）等,它通常实现`INotifyPropertyChanged`接口来支持数据绑定。

- `this` 指的是 当前 View（通常是一个 Window 或 UserControl）。

- `DataContext` 是 WPF 中用于**数据绑定的上下文对象**，假如View当中设置了`Binding value`，那么WPF会以`DataContext.value`的方式找到这个值。

