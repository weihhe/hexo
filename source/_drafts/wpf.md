title: wpf
author: weihehe
date: 2025-02-28 16:09:26
tags:
---
<!--more-->
### 1️⃣ **Properties 文件夹**
- **AssemblyInfo.cs**：存储程序集元数据（版本号、作者、版权等）
- **Resources.resx**：集中管理图片、字符串等静态资源
- **Settings.settings**：保存用户个性化设置（如窗口尺寸、主题等）
- **作用**：整个项目的"配置中心"，通过 `Properties.Resources.资源名` 或 `Properties.Settings.Default.设置名` 调用

---

### 2️⃣ **App.config**
- **功能**：XML格式的应用程序级配置文件
- **典型用途**：
  ```xml
  <connectionStrings>
    <add name="MyDB" connectionString="Server=..."/>
  </connectionStrings>
  ```
- **调用方式**：通过 `ConfigurationManager` 类读取配置信息

---

### 3️⃣ **App.xaml 核心文件**
- **程序入口**：`Application` 类的启动入口（等效于传统WinForm的Program.cs）
- **StartupUri**：在App.xaml中指定初始窗口
  ```xaml
  <Application StartupUri="MainWindow.xaml"/>
  ```
- **全局资源**：可在此合并全局样式/模板资源字典
- **关联文件**：App.xaml.cs 处理应用程序生命周期事件（启动/退出/异常捕获）

---

### 4️⃣ **MainWindow.xaml**
- **界面定义**：通过XAML标记语言描述窗口布局
- **代码分离**：对应的 `MainWindow.xaml.cs` 处理业务逻辑（按钮点击等事件）
- **开发技巧**：
  ```csharp
  // 代码后置示例：按钮点击事件
  private void Button_Click(object sender, RoutedEventArgs e)
  {
      MessageBox.Show("Hello WPF!");
  }
  ```

---

### 5️⃣ **引用（References）**
- **必需依赖**：默认包含的WPF核心库：
  - `PresentationCore.dll`（WPF渲染引擎）
  - `PresentationFramework.dll`（控件库）
  - `WindowsBase.dll`（基础类型）
- **扩展能力**：可添加NuGet包（如MVVM框架、图表控件等）

---

###  协作流程示例
1. **启动过程**：App.xaml → 加载MainWindow.xaml
2. **界面开发**：在XAML中使用Grid/StackPanel布局控件
3. **数据绑定**：通过 `Binding` 语法连接界面与数据模型
4. **配置读取**：从App.config获取数据库连接字符串
5. **资源管理**：通过Properties文件夹统一维护多语言文本

## 暂时未理解的

- StackPanel

