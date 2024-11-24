---
author: 小王爷
pubDatetime: 2024-11-24T11:06:00+08:00
modDatetime: 2024-11-24T11:06:00+08:00
title: "免费申请 TLS 证书"
draft: false
tags:
  - linux
  - tls
  - acme.sh
description:
  "使用 acme.sh 免费申请 TLS 证书"
---

自从有了域名后，经常需要申请免费的 TLS 证书供各个站点使用，为了方便查阅特此记录详细操作步骤。

## 免费申请 TLS 证书

1. 安装 acme.sh

```bash
curl https://get.acme.sh | sh -s email=hi@voxsay.com
```

2. 重新加载环境变量

```bash
source ~/.bashrc
```

3. 申请证书

> 注意：执行前先做好 DNS 解析。

```bash
acme.sh --issue -d voxsay.com --standalone
```

> 此过程依赖 **socat**，请提前使用 `apt install socat` 命令进行安装。

4. 安装证书

```bash
acme.sh --install-cert -d voxsay.com \
         --key-file /etc/certs/voxsay.com.key \
         --fullchain-file /etc/certs/voxsay.com.pem
```
