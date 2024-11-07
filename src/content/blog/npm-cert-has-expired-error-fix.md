---
author: 小王爷
pubDatetime: 2024-11-07T08:19:00+08:00
modDatetime: 2024-11-07T08:19:00+08:00
title: "registry.npm.taobao.org站certificate has expired问题"
draft: false
tags:
  - npm
  - TLS
description:
  "淘宝 NPM 镜像证书过期问题解决"
---

NPM 设置为淘宝 NPM 镜像源后，提示 `certificate has expired` 错误。具体错误如下：

```bash
$ npm i
npm ERR! code CERT_HAS_EXPIRED
npm ERR! errno CERT_HAS_EXPIRED
npm ERR! request to https://registry.npm.taobao.org/dotenv failed, reason: certificate has expired

npm ERR! A complete log of this run can be found in: /home/wss/.npm/_logs/2024-11-06T07_39_31_373Z-debug-0.log
```

上面的错误表明 npm 在尝试安装依赖时，遇到了证书过期的问题。具体来说，npm 请求了 https://registry.npm.taobao.org/dotenv 这个地址，但由于该服务器的 TLS 证书已过期，导致请求失败。

解决办法很简单，执行以下命令忽略证书校验后重新安装依赖即可。

```bash
npm config set strict-ssl false
```
