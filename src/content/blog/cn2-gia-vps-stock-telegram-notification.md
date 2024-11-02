---
author: 小王爷
pubDatetime: 2024-11-02T11:12:00+08:00
modDatetime: 2024-11-02T11:12:00+08:00
title: "CN2 GIA VPS 库存通知"
draft: false
tags:
  - VPS
  - 搬瓦工
  - DMIT
  - Telegram
  - Node.js
description:
  "每天自动监控 CN2 GIA 线路的实惠 VPS 库存，并发送 Telegram 通知"
---

一直在关注搬瓦工和 DMIT 的 CN2 GIA 线路的主机库存，由于一直缺货，为了避免每天人工的方式查看主机库存，所以基于 Node.js 和 Telegram API 开发了一个主机库存的监控程序。

## Table of Contents

## CN2 GIA 线路的实惠 VPS 库存通知

### Google 搜索 CN2 GIA 的库存查看地址

- 搬瓦工 The DC9 Plan 方案：[https://bandwagonhost.com/cart.php?a=add&pid=145](https://bandwagonhost.com/cart.php?a=add&pid=145)
- DMIT LAX.Pro.WEE 方案：[https://www.dmit.io/cart.php?a=add&pid=183](https://www.dmit.io/cart.php?a=add&pid=183)

### 创建 Telegram 机器人

与 BotFather 对话，操作指令如下，获取生成的 token：

```
Harrison Wang:
/newbot

BotFather:
Alright, a new bot. How are we going to call it? Please choose a name for your bot.

Harrison Wang:
stock

BotFather:
Good. Now let's choose a username for your bot. It must end in `bot`. Like this, for example: TetrisBot or tetris_bot.

Harrison Wang:
host_stock_bot

BotFather:
Done! Congratulations on your new bot. You will find it at t.me/host_stock_bot. You can now add a description, about section and profile picture for your bot, see /help for a list of commands. By the way, when you've finished creating your cool bot, ping our Bot Support if you want a better username for it. Just make sure the bot is fully operational before you do this.

Use this token to access the HTTP API:
xxxx:xxxx
Keep your token secure and store it safely, it can be used by anyone to control your bot.

For a description of the Bot API, see this page: https://core.telegram.org/bots/api
```

### 获取 chatId

首先，访问地址 `https://api.telegram.org/bot{token}/getUpdates`，观察输出如下：

> token 替换为上一步生成的 token 值

```json
{
  "ok": true,
  "result": []
}
```

然后，开启与 stock 机器人的对话，随意发送一条消息，比如：hello，再次访问 `https://api.telegram.org/bot{token}/getUpdates`，输出如下：

```json
{
  "ok": true,
  "result": [
    {
      "update_id": 687138978,
      "message": {
        "message_id": 53,
        "from": {
          "id": 6652465174,
          "is_bot": false,
          "first_name": "Harrison",
          "last_name": "Wang",
          "username": "harrisonwang",
          "language_code": "zh-hans"
        },
        "chat": {
          "id": 6652465174,
          "first_name": "Harrison",
          "last_name": "Wang",
          "username": "harrisonwang",
          "type": "private"
        },
        "date": 1729843456,
        "text": "hello"
      }
    }
  ]
}
```

可以看到，chat 节点下的 id 属性值 `6652465174` 即为 chatId。

### 监控搬瓦工库存

创建目录：

```bash
mkdir -p /opt/stock
```

切换至目录：

```bash
cd /opt/stock
```

新建文件：

```bash
touch host_inventory.js 
```

编辑文件，添加以下内容：

> 替换 botToken 和 chatId 值。

```js
const bandwagonUrl = 'https://bandwagonhost.com/cart.php?a=add&pid=145';

const botToken = '****:****';
const chatId = '****';

async function getStock() {
  const bandwagonInStock = await isBandwagonInStock();
  const dmitInStock = await isDmitInStock();

  if (bandwagonInStock) {
    await send(`BandwagonHost product is in stock! ${bandwagonUrl}`);
  }
}

async function isBandwagonInStock() {
  try {
    const response = await fetch(bandwagonUrl);
    const text = await response.text();
    const isInStock = !text.includes('Out of Stock');
    return isInStock;
  } catch (error) {
    console.error('Error checking BandwagonHost stock:', error);
    return false;
  }
}

async function send(message) {
  const telegramUrl = `https://api.telegram.org/bot${botToken}/sendMessage`;

  try {
    const response = await fetch(telegramUrl, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        chat_id: chatId,
        text: message
      })
    });

    const data = await response.json();
    if (!response.ok) {
      console.error(`Error sending message to Telegram: ${data.description}`);
    } else {
      console.log('Message sent successfully to Telegram');
    }
  } catch (error) {
    console.error('Error sending message to Telegram:', error);
  }
}

getStock();
```

手动运行程序：

```bash
node host_inventory.js
```

可以看到没有任何输出，因为当前没有库存。

由于库存不会经常发生变化，我们设计为一天查询两次库存，为了实现定时通知，直接使用 `crontab` 来实现。

因 `crontab` 执行时使用的环境变量与用户的 Shell 环境变量不同，通常 `crontab` 中没有直接可用的 Node.js 环境，所以我们新建一个 shell 脚本 `host_inventory.sh` 来执行 `node host_inventory.js` 命令：

```bash
#!/bin/bash

cd /opt/stock
/root/.nvm/versions/node/v20.17.0/bin/node /opt/stock/host_inventory.js
```

接着，执行以下命令：

```bash
crontab -e
```

然后，添加以下内容并保存：

```crontab
0 9 * * * /opt/stock/host_inventory.sh >> /opt/stock/host_inventory.log 2>&1
0 17 * * * /opt/stock/host_inventory.sh >> /opt/stock/host_inventory.log 2>&1
```

> 上面的配置表示每天早上 9 点和下午 17 点各查询一次库存。

保存后将输出：

```bash
crontab: installing new crontab
```

### 监控 DMIT 库存

由于 DMIT 使用了 Cloudflare 提供的反爬机器人，我们需要通过 [ScrapeOps](https://scrapeops.io/) 代理访问 [DMIT 库存页](https://www.dmit.io/cart.php?a=add&pid=183)，来绕过反爬机器人的限制。

> 如何绕过反爬机器人？请参见文章 [How To Bypass Cloudflare in 2024](https://scrapeops.io/web-scraping-playbook/how-to-bypass-cloudflare)。

首先，注册一个 ScrapeOps 账号，获取其提供的 API Key，它给免费账号提供了 1000 次请求的机会，对于库存监控来说足够了，超过一年后可以重新申请账号继续使用。

然后，对要爬取的网页地址进行编码并开启渲染 JS 的配置，请求示例如下：

```http
GET https://proxy.scrapeops.io/v1/?api_key=****&url=${encodedUrl}&render_js=true
```

> render_js 参数表示开启渲染 JS 的功能。DMIT 库存页渲染 JS 后内容才会加载出来，所以需要开启此功能。

最后，完整的代码如下，一旦检测到有库存将收到 Telegram 通知：

```js
const bandwagonUrl = 'https://bandwagonhost.com/cart.php?a=add&pid=145';
const dmitUrl = 'https://www.dmit.io/cart.php?a=add&pid=183';

const botToken = '****';
const chatId = '****';

async function getStock() {
  const bandwagonInStock = await isBandwagonInStock();
  const dmitInStock = await isDmitInStock();

  if (bandwagonInStock) {
    await send(`BandwagonHost product is in stock! ${bandwagonUrl}`);
  }

  if (dmitInStock) {
    await send(`DMIT product is in stock! ${dmitUrl}`);
  }
}

async function isBandwagonInStock() {
  try {
    const response = await fetch(bandwagonUrl);
    const text = await response.text();
    const isInStock = !text.includes('Out of Stock');
    return isInStock;
  } catch (error) {
    console.error('Error checking BandwagonHost stock:', error);
    return false;
  }
}

async function isDmitInStock() {
  try {
    const encodedUrl = encodeURIComponent(dmitUrl);
    const proxyDmitUrl = `https://proxy.scrapeops.io/v1/?api_key=****&url=${encodedUrl}&render_js=true`;
    const response = await fetch(proxyDmitUrl);
    const text = await response.text();
    const isInStock = !text.includes('Out of Stock');
    return isInStock;
  } catch (error) {
    console.error('Error checking DMIT stock:', error);
    return false;
  }
}

async function send(message) {
  const telegramUrl = `https://api.telegram.org/bot${botToken}/sendMessage`;

  try {
    const response = await fetch(telegramUrl, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        chat_id: chatId,
        text: message
      })
    });

    const data = await response.json();
    if (!response.ok) {
      console.error(`Error sending message to Telegram: ${data.description}`);
    } else {
      console.log('Message sent successfully to Telegram');
    }
  } catch (error) {
    console.error('Error sending message to Telegram:', error);
  }
}

getStock();
```
