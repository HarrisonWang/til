---
author: 小王爷
pubDatetime: 2024-10-27T09:21:00+08:00
modDatetime: 2024-10-27T09:21:00+08:00
title: "remark42 本地运行"
draft: false
tags:
  - remark42
description:
  "在 remark42 官方文档和 Google 搜索的帮助下，我弄清楚了 remark42 评论站点的最小化部署方式。我只想要一个基本的 remark42 评论系统，不依赖 Docker，使用尽量小的内存，从而可以灵活的部署到我的 VPS 上。"
---

在 remark42 官方文档和 Google 搜索的帮助下，我弄清楚了 remark42 评论站点的最小化部署方式。我只想要一个基本的 remark42 评论系统，不依赖 Docker，使用尽量小的内存，从而可以灵活的部署到我的 VPS 上。

## remark42 本地运行

### 从二进制包运行

创建目录：

```bash
mkdir -p /opt/remark42
```

访问地址 [https://github.com/remark42/releases/releases/latest](https://github.com/remark42/releases/releases/latest) 下载最新版本。

切换至目录：

```bash
cd /opt/remark42
```

解压：

```bash
tar -tvf remark42.linux-amd64.tar.gz -C /opt/remark42
```

环境变量配置：

```bash
# Remark42 服务器 URL
$ export REMARK_URL=https://example.com
# JWT 密钥
$ export SECRET=****
# 站点名称
$ SITE=voxsay
# 开启 emoji
$ EMOJI=true
# 管理员密码
$ export ADMIN_PASSWD=****
```

运行：

```bash
./remark42.linux-amd64 server
```
