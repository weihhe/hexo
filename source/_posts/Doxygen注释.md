title: Doxygen注释
author: weihehe
tags:
  - Doxygen
categories:
  - 工具
date: 2025-07-14 17:53:00
---
功能性更强的注释
<!--more-->

### 为什么使用 `/** ... */` 这种写法，而不是 `//` 或 `/* ... */`？

Doxygen 识别的文档注释格式，用于生成函数、类等的自动文档。

---

### `@brief`

`@brief` 是 Doxygen 的命令之一，表示对这个函数或方法的简短说明（brief description）。例如：

```cpp
/**
 * @brief 刷新树形模型数据，显示所有可用的数据源
 */
void refreshModel();
```

| 标签/关键字 | 说明  | 示例  |
| --- | --- | --- |
| `\brief` | 简要说明（函数/类/变量的作用，作为摘要） | `\brief 获取西瓜大小` |
| `\details` | 详细说明（更完整的描述，可多行） | `\details 本函数用于获取西瓜的小大。` |
| `\param` | 参数说明（参数名+含义+取值范围） | `\param weight 西瓜重量，取值范围 0-100` |
| `\return` / `\returns` | 返回值说明 | `\return 当前西瓜大小，单位cm` |
| `\exception` / `\throws` | 抛出的异常说明 | `\exception std::invalid_argument 当 重量 超出范围时抛出` |
| `\author` | 作者信息 | `\author weihehe` |
| `\date` | 日期  | `\date 2025/04/15` |
| `\version` | 版本号 | `\version 1.0` |
| `\note` | 注意事项 | `\note 西瓜不应超过100kg` |
| `\warning` | 警告信息 | `\warning 只能输入一个西瓜的重量` |
| `\see` | 参考链接/函数 | `\see GetWatermelonSize()` |
| `\deprecated` | 已弃用标记 | `\deprecated 请使用 NewGetWatermelonSize()` |

## 示例：

```cpp
/*!
 * \brief 获取西瓜大小
 * \details 本函数用于获取西瓜的大小，基于输入的重量计算西瓜直径。
 *
 * \param weight 西瓜重量，取值范围 0-100。
 * \return 当前西瓜大小，单位 cm。
 * \exception std::invalid_argument 当 weight 超出范围时抛出。
 *
 * \author weihehe
 * \date 2025/04/15
 * \version 1.0
 * \note 西瓜不应超过 100kg。
 * \warning 只能输入一个西瓜的重量。
 * \see GetWatermelonSize()
 * \deprecated 请使用 NewGetWatermelonSize()
 */
int GetWatermelonSize(int weight);


```