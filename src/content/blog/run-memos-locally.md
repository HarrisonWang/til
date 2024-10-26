---
author: 小王爷
pubDatetime: 2024-10-26T21:38:00Z
modDatetime: 2024-10-26T21:54:00Z
title: "Memos 本地部署"
draft: false
tags:
  - memos
description:
  "因 VPS 内存较小，不想通过 Docker 方式部署 memos 服务，所以找到了本地部署最省内存的方式，部署到 VPS 上。"
---

因 VPS 内存较小，不想通过 Docker 方式部署 memos 服务，所以找到了本地部署最省内存的方式，部署到 VPS 上。

## Memos 本地部署

1. 下载 Memos 二进制包

```bash
wget https://github.com/usememos/memos/releases/download/v0.22.5/memos_v0.22.5_linux_amd64.tar.gz
```

2. 创建 memos 目录

```bash
mkdir -p /opt/memos/data
```

3. 解压到 memos 目录

```bash
tar -zxvf memos_v0.22.5_linux_amd64.tar.gz -C /opt/memos
```

4. 切换到 memos 目录

```bash
cd /opt/memos
```

5. 启动 Memos 服务

```bash
nohup ./memos --data /opt/memos/data --mode prod --port 5230 &
```

6. 申请 TLS 免费证书

```bash
# 安装 acme.sh
curl https://get.acme.sh | sh -s email=hi@example.org

# 重新加载环境变量
source ~/.bashrc

# 申请证书
acme.sh --issue -d example.org --standalone

# 安装证书
acme.sh --install-cert -d example.org \
         --key-file /etc/certs/example.org.key \
         --fullchain-file /etc/certs/example.org.pem
```

7. 使用 nginx 反向代理：

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name example.org;
    return 301 https://$server_name:443$request_uri;
}

server {
    listen       443 ssl http2;
    listen       [::]:443 ssl http2;
    server_name example.org;
    charset utf-8;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers TLS13-AES-256-GCM-SHA384:TLS13-CHACHA20-POLY1305-SHA256:TLS13-AES-128-GCM-SHA256:TLS13-AES-128-CCM-8-SHA256:TLS13-AES-128-CCM-SHA256:EECDH+CHACHA20:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_session_cache builtin:1000 shared:SSL:10m;
    ssl_session_timeout 10m;
    ssl_buffer_size 1400;
    ssl_stapling on;
    ssl_stapling_verify on;
    ssl_session_tickets off;
    ssl_certificate /etc/certs/example.org.pem;
    ssl_certificate_key /etc/certs/example.org.key;
    location / {
        proxy_pass http://localhost:5230;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
