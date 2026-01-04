title: nodejs更新修复SyntaxError报错
author: weihehe
tags:
  - Node.js
  - ''
categories:
  - 工具
date: 2025-07-18 15:28:00
---
## 首先是报错信息

你日志中出现的两个典型报错：

```bash
SyntaxError: Unexpected token '.'
```

这些写法是 **Node.js v14+** 或 **v16+** 才支持的

---

## 解决方法

### 🔧1. 升级 Node.js 至 v18 或以上

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

###  2. 清理并重装依赖

升级完 Node.js 后，建议清除旧依赖重新安装：

```bash
cd ~/hexo/hexo
rm -rf node_modules package-lock.json
npm install
```

---