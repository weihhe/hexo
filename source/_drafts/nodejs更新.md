title: nodejs更新
author: weihehe
date: 2025-07-18 15:28:41
tags:
---
你遇到的错误根本原因是：**你使用的 Node.js 版本太旧**，不支持代码中用到的现代 JavaScript 语法，例如 `?.`（可选链操作符）和 `.at(-1)`（数组取最后一个元素的新方法）。

---

## 🔍 错误分析

你日志中出现的两个典型报错：

```bash
SyntaxError: Unexpected token '.'
```

分别出现在：

* `hexo-renderer-marked` 的 `marked.cjs` 文件：

  ```js
  if (cells.length > 0 && !cells.at(-1)?.trim())
  ```
* `@adobe/css-tools` 的：

  ```js
  options?.source
  ```

这些写法是 **Node.js v14+** 或 **v16+** 才支持的，而你的当前环境的 Node.js 是 **低版本（估计是 v12 或更低）**。

---

## ✅ 解决方法

### 🔧 1. 升级 Node.js 至 v18 或以上

你可以用如下方式升级：

#### 如果你使用的是 Ubuntu，可以使用 [nvm](https://github.com/nvm-sh/nvm) 来安装管理 Node.js 版本：

```bash
# 安装 nvm（如尚未安装）
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
# 重新加载 shell 配置（例如 bash）
source ~/.bashrc

# 安装 Node.js 最新 LTS（推荐）
nvm install --lts

# 使用刚安装的 Node.js
nvm use --lts

# 设置为默认版本
nvm alias default lts/*
```

验证版本：

```bash
node -v
# 输出应为 v18.x 或更高
```

---

### 🔁 2. 清理并重装依赖

升级完 Node.js 后，建议清除旧依赖重新安装：

```bash
cd ~/hexo/hexo
rm -rf node_modules package-lock.json
npm install
```

---

### ▶️ 3. 再次运行 Hexo

```bash
hexo clean
hexo generate
hexo server -p 4399
```

---

## 🔚 总结

| 操作            | 说明                        |
| ------------- | ------------------------- |
| 🔄 升级 Node.js | 这是必须的，因为你使用的插件用到了现代 JS 语法 |
| 🧹 清除并重装依赖    | 避免老旧模块冲突                  |
| ✅ 重新启动 Hexo   | 检查是否成功                    |

如果还有报错，请贴出 `node -v` 和 `npm ls` 的结果，我可以进一步帮你分析。
