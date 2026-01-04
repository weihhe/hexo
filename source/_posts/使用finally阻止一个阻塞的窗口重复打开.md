title: 使用finally防止一个模态（阻塞）窗口重复打开
author: weihehe
tags: []
categories:
  - .NET
date: 2025-08-07 15:34:00
---
<!--more-->
```csharp
private bool _isDialogOpen = false;

private async void OnEditAtt()
{
    if (_isDialogOpen)
        return; // 防止重复打开

    _isDialogOpen = true;

    try
    {
        //await （打开窗口）
                 
    }
    catch (Exception ex)
    {
        await ShowAsync($"打开窗口失败: {ex.Message}");
    }
    finally
    {
        _isDialogOpen = false; // 窗口关闭后恢复状态
    }
}

```
finally关键字用于定义一个代码块，**无论是否发生异常，该代码块都会在try块和任何匹配的catch块之后执行**