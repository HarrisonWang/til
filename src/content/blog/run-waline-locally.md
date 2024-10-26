---
author: 小王爷
pubDatetime: 2024-10-28T15:22:00Z
modDatetime: 2024-10-28T15:22:00Z
title: Waline 本地运行
draft: false
tags:
  - waline
description:
  在 waline 官方文档和 Google 搜索的帮助下，我弄清楚了 waline 评论站点的最小化部署方式。我只想要一个基本的 waline 评论系统，不依赖 Docker，使用尽量小的内存，从而可以灵活的部署到我的 VPS 上。
---

## Waline 本地运行

在查阅 waline 官方文档和 waline 仓库 Issues 与 Discussions 的情况下，我弄清楚了 waline 评论站点的最小化部署方式。

我只想要一个基本的 waline 评论系统，基于 SQLite，不依赖 Docker 和 LeanCloud，可以灵活的部署到我的 VPS 上。

### 从源码运行

切换至 opt 目录：

```bash
cd /opt
```

下载源代码：

```bash
git clone https://github.com/walinejs/waline.git
```

切换至 server 目录：

```bash
cd /opt/waline/packages/server
```

安装依赖包：

```bash
npm i
```

配置 `SQLITE_PATH` 和 `JWT_TOKEN` 环境变量：

```bash
# For Linux Bash
export SQLITE_PATH=./data/
export JWT_TOKEN=your_random_secret_here

# For Windows 命令行提示符
$env:SQLITE_PATH = "./data/"
$env:JWT_TOKEN = "your_random_secret_here"

# For Windows PowerShell
set SQLITE_PATH=./data/
set JWT_TOKEN=your_random_secret_here
```

初始化数据库：

```bash
# 创建数据库文件存储目录
mkdir -p /opt/waline/packages/server/data

# 下载数据库文件
wget -P /opt/waline/packages/server/data https://raw.githubusercontent.com/walinejs/waline/main/assets/waline.sqlite
```

运行：

```bash
node vanilla.js
```

### 从 npm 包运行

创建 waline 目录：

```bash
mkdir -p /opt/waline
```

切换至 waline 目录：

```bash
cd /opt/waline
```

安装 waline 包：

```bash
npm install @waline/vercel
```

配置 `SQLITE_PATH` 和 `JWT_TOKEN` 环境变量：

```bash
# Linux Bash
export SQLITE_PATH=./data/
export JWT_TOKEN=your_random_secret_here

# Windows 命令行提示符
$env:SQLITE_PATH = "./data/"
$env:JWT_TOKEN = "your_random_secret_here"

# Windows PowerShell
set SQLITE_PATH=./data/
set JWT_TOKEN=your_random_secret_here
```

初始化数据库：

```bash
# 创建数据库文件存储目录
mkdir -p /opt/waline/data

# 下载数据库文件
wget -P /opt/waline/data https://raw.githubusercontent.com/walinejs/waline/main/assets/waline.sqlite
```

运行：

```bash
node node_modules/@waline/vercel/vanilla.js
```

### 访问

打开地址 [http://127.0.0.1:8360](http://127.0.0.1:8360) 测试评论功能，测试结果如下图：

![评论页](@assets/images/image-20241025124653531.webp)
