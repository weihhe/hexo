title: MVC与MVVM
author: weihehe
date: 2025-02-27 09:42:10
tags:
---
<!--more-->

# MVC

[知乎](https://zhuanlan.zhihu.com/p/417635345)

# MVVM框架笔记整理

## 一、框架概述

MVVM（Model-View-ViewModel）是一种设计模式，旨在将用户界面（View）、业务逻辑（ViewModel）和数据模型（）Model分离。

### （一）各部分职责

1. **Model**：表示数据和业务逻辑。
  
2. **View**：定义用户界面。
  
3. **ViewModel**：作为 View 和 Model 之间的桥梁，提供数据绑定和命令处理。

#### 绑定

1. 数据绑定：通过 INotifyPropertyChanged 实现实时更新。
2. 命令绑定：通过 ICommand 处理用户交互。

## 二、样例代码

### （一）XAML（定义界面与数据绑定）

```xml
<Button Content="Submit" Command="{Binding SubmitCommand}" Margin="0,10,0,0"/>
<TextBlock Text="{Binding OutputText}" Margin="0,10,0,0"/>
```

### （二）Model（数据的载体）


```c#
public class Person
{
    public string Name { get; set; }
}
```

### （三）ViewModel（提供数据绑定和命令处理）

1. 实现 `INotifyPropertyChanged` 接口以支持数据绑定。
  
2. 作为所有视图模型类的基类，通过实现该接口，视图模型可以通知 UI 属性值的变化，从而实现数据绑定的动态更新。
  

### （四）RelayCommand 类（实现 `ICommand` 接口）

用于将用户界面（UI）中的操作（如按钮点击）与业务逻辑解耦，通常用于 WPF、UWP 等支持命令模式的框架中。

#### 1. `CanExecuteChanged` 事件


```csharp
public event EventHandler CanExecuteChanged
{
    add => CommandManager.RequerySuggested += value;
    remove => CommandManager.RequerySuggested -= value;
}
```

- **`CanExecuteChanged`**：实现 `ICommand` 接口的 `CanExecuteChanged` 事件，用于通知 UI 命令的可用性可能发生了变化。
  
- **`CommandManager.RequerySuggested`**：WPF 中的静态类，用于管理命令的查询建议。当命令的可用性可能发生变化时，`RequerySuggested` 事件会被触发，从而通知 UI 更新命令的可用性。
  

#### 2. `CanExecute` 方法


```csharp
public bool CanExecute(object parameter)
{
    return _canExecute?.Invoke() ?? true;
}
```

- **`public bool CanExecute(object parameter)`**：方法接受一个参数 `parameter`，其类型是 `object`，可以接受任何类型的输入（可以是 `null` 或任何对象）。
  
- **`return _canExecute?.Invoke() ?? true;`**：
  
  - `_canExecute?.Invoke()`：`_canExecute` 是一个委托，`?.` 是 null 条件运算符，若 `_canExecute` 为 `null`，则避免调用 `_canExecute.Invoke()`，直接返回 `null`；否则执行 `_canExecute` 所指向的方法。
    
  - `?? true`：空合并运算符，若左侧值为 `null`（即 `_canExecute` 为 `null`），则返回右侧的值 `true`。
    

## 三、关键概念补充

### （一）`DataContext`

`DataContext` 是 WPF 中用于数据绑定的属性。设置 `DataContext` 为 ViewModel，意味着当前窗口的数据绑定将依赖于 ViewModel，UI 中的控件将根据 ViewModel 的属性进行数据绑定和显示。

**数据绑定（Binding）** 是一种强大的机制，用来在 UI 控件和应用程序的数据模型（如 ViewModel 或 Model）之间传递数据。数据绑定可以自动保持 UI 与数据源同步。

### 数据绑定的基本逻辑

数据绑定的核心思想是将 UI 元素（如 `TextBox`、`Label`、`ListBox` 等）与数据源（如 ViewModel、Model 或集合）连接起来，使得 UI 和数据源之间的值变化能够自动同步。

**一般的绑定流程**：

1. **定义绑定关系**：在 XAML 中，通过 `Binding` 将 UI 控件的属性与数据源的某个属性绑定。例如，将一个 `TextBox` 的 `Text` 属性绑定到 ViewModel 中的一个属性。
  
2. **数据源**：数据源可以是 ViewModel、Model 或其他对象。在 MVVM 模式下，通常是 ViewModel。
  
3. **绑定目标**：UI 控件或元素，目标属性是通过数据绑定进行设置的属性。
  
4. **更新机制**：当数据源的值发生变化时，绑定目标的值会自动更新，反之，UI 控件的修改（如果是双向绑定）会同步回数据源。
  

### 绑定的类型

在 WPF 中，常见的绑定类型有：

1. **OneWay（单向绑定）**：
  
  - 只从数据源更新到目标（UI 控件）。当数据源的值发生变化时，UI 控件会自动更新。
  - 例如：`TextBox` 显示来自 ViewModel 的字符串。
  
  ```xml
  <TextBlock Text="{Binding Name}" />
  ```
  
2. **TwoWay（双向绑定）**：
  
  - 数据源与目标之间保持同步。UI 控件的变化会自动更新到数据源，数据源的变化也会自动更新到 UI 控件。
  - 例如：`TextBox` 输入的内容会同步回 ViewModel。
  
  ```xml
  <TextBox Text="{Binding Name, Mode=TwoWay}" />
  ```
  
3. **OneWayToSource（单向到源绑定）**：
  
  - 只有目标（UI 控件）更新数据源。UI 控件的变化会更新数据源，但数据源的变化不会自动反向更新 UI 控件。
4. **OneTime（一次绑定）**：
  
  - 数据源的值在初始化时传递到目标，之后不再更新目标属性。适用于静态数据，数据源不会再发生变化。
  
  ```xml
  <TextBlock Text="{Binding Name, Mode=OneTime}" />
  ```
  
5. **Default（默认绑定）**：
  
  - 默认情况下，绑定会根据绑定的属性类型自动选择绑定模式。

### 数据绑定的基本原理

1. **绑定源（Source）**：
  
  - 数据源是绑定的源，通常是一个对象（如 ViewModel 或 Model）。
  - 绑定源通常需要实现 `INotifyPropertyChanged` 接口，以便在属性更改时通知绑定的目标更新其值。
2. **绑定目标（Target）**：
  
  - 目标是 UI 控件的属性，比如 `TextBox.Text`、`ListBox.SelectedItem` 等。
3. **绑定路径**：
  
  - 绑定路径用于指定从源对象到目标属性的路径。例如，`{Binding Person.Name}` 表示从 `Person` 对象中获取 `Name` 属性的值。
  - 如果数据源对象是一个集合或对象，路径可以包括多级属性，例如 `{Binding Address.Street}`。
4. **更新方向（UpdateSourceTrigger）**：
  
  - 这个属性决定了何时从目标更新数据源。常见的值有：
    - **LostFocus**：当控件失去焦点时更新源数据。
    - **PropertyChanged**：每当目标属性值发生变化时更新源数据。
    - **Explicit**：显式调用更新源。

### 数据绑定的关键概念

1. **INotifyPropertyChanged 接口**：
  
  - 用于在数据源对象的属性值发生变化时，通知绑定系统更新绑定的目标属性。常见于 ViewModel 或 Model。
    
  - 示例：
    
    ```csharp
    public class Person : INotifyPropertyChanged
    {
       private string _name;
       public string Name
       {
           get { return _name; }
           set
           {
               if (_name != value)
               {
                   _name = value;
                   OnPropertyChanged(nameof(Name));
               }
           }
       }
    
       public event PropertyChangedEventHandler PropertyChanged;
       private void OnPropertyChanged(string propertyName)
       {
           PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
       }
    }
    ```
    
  - 当 `Name` 属性的值改变时，通过 `OnPropertyChanged` 方法通知绑定系统更新绑定目标的 UI 控件。
    
2. **BindingMode**：
  
  - 可以在绑定中设置绑定模式（如 `OneWay`, `TwoWay` 等）。例如：
    
    ```xml
    <TextBox Text="{Binding Name, Mode=TwoWay}" />
    ```
    
3. **DataContext**：
  
  - `DataContext` 是绑定的上下文对象。在 WPF 中，UI 控件会从 `DataContext` 获取其数据源对象。如果你在父控件设置了 `DataContext`，子控件会自动继承这个上下文，简化了绑定。
  
  ```csharp
  this.DataContext = myViewModel;
  ```
  

