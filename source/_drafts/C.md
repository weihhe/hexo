title: C#
author: weihehe
date: 2025-02-27 14:13:27
tags:
---
<!--more-->

# 关键字

### 保留关键字

| 保留关键字 | 作用 |
|-----------|-------|
| abstract  | 用于类或成员，声明抽象类或成员，不能被实例化或调用 |
| as        | 用于类型转换，隐式转换失败时返回 null，不抛出异常 |
| base      | 用于在派生类中引用基类成员 |
| bool      | 布尔型数据类型，值为 true 或 false |
| break     | 用于跳出循环或 switch 语句 |
| byte      | 无符号 8 位整数型数据类型 |
| case      | switch 语句中用于标记分支 |
| catch     | 用于捕获异常 |
| char      | 单个 Unicode 字符型数据类型 |
| checked   | 启用运行时的溢出检查 |
| class     | 定义一个类 |
| const     | 定义常量 |
| continue  | 跳过循环中的当前迭代，继续下一次循环 |
| decimal   | 高精度的十进制浮点型数据类型 |
| default   | 在 switch 语句中表示默认分支或在泛型中提供默认值 |
| delegate  | 定义委托类型 |
| do        | 后接 while，构成一个循环 |
| double    | 双精度浮点型数据类型 |
| else      | 配合 if 使用，表示条件不满足时执行的代码 |
| enum      | 定义枚举类型 |
| event     | 定义事件 |
| explicit  | 定义显式转换 |
| extern    | 表示方法在外部实现 |
| false     | 布尔值 false |
| finally   | 用于异常处理，无论是否发生异常都会执行 |
| fixed     | 用于固定变量的地址 |
| float     | 单精度浮点型数据类型 |
| for       | 定义 for 循环 |
| foreach   | 遍历集合或数组 |
| goto      | 跳转到指定标签 |
| if        | 条件语句 |
| implicit  | 定义隐式转换 |
| in        | 用于传入参数或查询操作 |
| int       | 32 位整数型数据类型 |
| interface | 定义接口 |
| internal  | 修饰符，表示成员在程序集内可见 |
| is        | 用于类型检查 |
| lock      | 用于锁定代码块，避免多线程竞争 |
| long      | 64 位整数型数据类型 |
| namespace | 命名空间定义 |
| new       | 创建对象实例或隐藏基类成员 |
| null      | 表示空引用 |
| object    | 所有类型的基类 |
| operator  | 重载运算符 |
| out       | 输出参数 |
| override  | 重写基类的虚方法 |
| params    | 允许方法接受可变数量的参数 |
| private   | 修饰符，表示成员在类内可见 |
| protected | 修饰符，表示成员在类及其子类可见 |
| public    | 修饰符，表示成员对所有外部可见 |
| readonly  | 只读字段，只能在声明或构造函数中赋值 |
| ref       | 引用参数 |
| return    | 返回方法的结果 |
| sbyte     | 有符号 8 位整数型数据类型 |
| sealed    | 密封类或成员，不能被继承或重写 |
| short     | 16 位整数型数据类型 |
| sizeof    | 获取数据类型或变量的大小 |
| stackalloc | 在堆栈上分配内存 |
| static    | 修饰符，表示成员属于类型本身 |
| string    | 字符串数据类型 |
| struct    | 定义值类型 |
| switch    | 多分支条件语句 |
| this      | 当前实例引用 |
| throw     | 抛出异常 |
| true      | 布尔值 true |
| try       | 尝试执行代码块，并捕获异常 |
| typeof    | 获取类型的 Type 对象 |
| uint      | 无符号 32 位整数型数据类型 |
| ulong     | 无符号 64 位整数型数据类型 |
| unchecked | 禁用运行时的溢出检查 |
| unsafe    | 表示不安全的代码块 |
| ushort    | 无符号 16 位整数型数据类型 |
| using     | 引入命名空间或定义资源管理 |
| virtual   | 用于声明虚方法或属性，允许被重写 |
| void      | 表示方法没有返回值 |
| volatile  | 表示变量可能被多个线程修改 |
| while     | 定义 while 循环 |

### 上下文关键字

| 上下文关键字 | 作用 |
|--------------|-------|
| add          | 用于事件的自定义添加逻辑 |
| alias        | 用于命名空间或类型的别名 |
| ascending    | 在 LINQ 查询中表示升序排序 |
| descending   | 在 LINQ 查询中表示降序排序 |
| dynamic      | 表示动态类型 |
| from         | LINQ 查询中的数据源 |
| get          | 属性或索引器的 getter |
| global       | 引用全局命名空间 |
| group        | LINQ 的分组查询 |
| into         | LINQ 查询中的结果占位符 |
| join         | LINQ 的连接操作 |
| let          | LINQ 查询中的中间计算 |
| orderby      | LINQ 查询中的排序 |
| partial (type)| 分部类或结构 |
| partial (method)| 分部方法 |
| remove       | 用于事件的自定义移除逻辑 |
| select       | LINQ 查询中的投影 |
| set          | 属性或索引器的 setter |

# 概念

## 顶级语句

- 新的编程范式，允许开发者在文件的顶层直接编写语句，而不需要将它们封装在方法或类中。

## 插值字符串

- 以 $ 符号开头，用 {} 包裹表达式：
- 允许在字符串中直接嵌入表达式（变量、方法调用等），动态生成格式化后的字符串

```csharp
string name = "Alice";
int age = 30;
string message = $"Name: {name}, Age: {age}"; 
// 输出：Name: Alice, Age: 30
```

## LINQ

- LINQ 是 C# 中统一的数据查询技术，支持对集合、数据库、XML 等数据源进行声明式查询。

- 两种语法形式
	- 查询表达式（类似 SQL）：
    ```csharp
    var query = from p in products
            where p.Price > 100
            orderby p.Name
            select p.Name;
    ```
	- 方法语法（链式调用）：
    ```csharp
    var query = products
            .Where(p => p.Price > 100)
            .OrderBy(p => p.Name)
            .Select(p => p.Name);
    ```
- 支持延迟执行
	```csharp
    var query = products.Where(p => p.Price 	> 100); // 未执行
    var result = query.ToList(); // 执行查询
    ```


## 表达式体方法

- C# 6.0+ 允许用 **Lambda 箭头语法** (=>) 简化单行方法、属性、索引器等成员的写法。

```csharp
// 传统写法
public int Add(int a, int b)
{
    return a + b;
}

// 表达式体写法
public int Add(int a, int b) => a + b;
```
## 属性

- 作用：封装对类或结构体内部字段（field）的访问，提供可控的读写逻辑。
- 组成：通过 get 和 set 访问器定义值的获取和设置。
- 本质：是方法（method）的语法糖，编译后会生成 get_XXX 和 set_XXX 方法。

![属性——菜鸟教程](/images/pasted-24.png)

## 索引器

- 作用：允许对象像数组一样通过索引访问其内部集合元素。

- 语法：使用 this 关键字和方括号 [] 定义。

### 样例

#### 1. 类定义
```csharp
public class Matrix
{
    private int[,] _data = new int[3,3];
    // ...
}
```
- 定义了一个名为 `Matrix` 的公共类。
- 类内部包含一个私有字段 `_data`，它是一个 `3x3` 的二维整数数组。

#### 2. 索引器（Indexer）
```csharp
public int this[int row, int col]
{
    get => _data[row, col];
    set => _data[row, col] = value;
}
```
- 定义了一个索引器，语法类似于 `public int this[int row, int col]`。
- 索引器允许通过 `matrix[row, col]` 的方式访问 `Matrix` 的元素。

#### 3. 索引器的 `get` 和 `set` 语义
##### `get` 方法：
```csharp
get => _data[row, col];
```
- 当使用 `matrix[row, col]` 获取值时，`get` 方法会被调用。
- 它返回 `_data` 数组中位于 `(row, col)` 位置的值。

##### `set` 方法：
```csharp
set => _data[row, col] = value;
```
- 当使用 `matrix[row, col] = value` 设置值时，`set` 方法会被调用。
- 它将值赋给 `_data` 数组中位于 `(row, col)` 位置的元素。

#### 4. 使用示例
```csharp
var matrix = new Matrix();
matrix[1, 2] = 100; // 二维索引
```
- 创建了一个 `Matrix` 类的实例，命名为 `matrix`。
- 使用索引器设置 `matrix[1, 2]` 的值为 `100`。

#### 索引器的功能
- 索引器允许类的实例像数组一样被索引，例如 `matrix[1, 2]`。
- 它提供了对类内部数据（如 `_data` 数组）的灵活访问和操作方式。
- 索引器可以接受多个参数（如这里的 `row` 和 `col`），满足对多维数据结构的访问需求。

## Null条件运算符

Null 条件运算符（?.）和空合并运算符（??）是用于处理可能为空的对象的特殊运算符（[转自博客园](https://www.cnblogs.com/ouyangkai/p/17806949.html)）

- Null 条件运算符（?.）用于在访问对象的属性或调用对象的方法之前，先检查对象是否为空。如果对象为空，运算符将返回 null，否则将继续执行后续操作。

- 空合并运算符（??）用于在对象为空时提供一个默认值。如果对象为空，运算符将返回默认值，否则将返回对象本身。

```c#
// 使用 Null 条件运算符和空合并运算符的组合
if (customer?.Face() ?? false)
{
    // 执行操作
}
```
## 引用（引用类型和值类型）

## 特性

- **[CallerMemberName]**：这是一个特性，用于自动获取调用方法的名称。如果调用方法没有显式传递参数，则 propertyName 会自动填充为调用方法的名称。

- https://www.cnblogs.com/fengjq/p/17589245.html