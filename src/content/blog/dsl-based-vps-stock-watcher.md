---
author: 小王爷
pubDatetime: 2024-10-29T09:30:00+08:00
modDatetime: 2024-10-29T09:30:00+08:00
title: "VPS 库存监控工具"
draft: false
tags:
  - DSL
  - Node.js
  - Puppeteer
  - VPS
description:
  "用 DSL 和 Node.js 实现的一个优雅的 VPS 库存监控工具"
---

今天写了一个有趣的小工具 - VPS Watcher。这个项目最大的特点是设计了一套简单的 DSL (领域特定语言) 来描述监控任务，让配置变得更加优雅和直观。

## 用 DSL 和 Node.js 实现一个优雅的 VPS 库存监控工具

### 为什么要用 DSL？

相比传统的 JSON 配置，DSL 能提供：

- 更好的可读性
- 更强的表达能力
- 更少的重复代码

## DSL 示例

看看这个监控规则有多简洁：

```dsl
test "Check BandwagonHost Stock" {
    open "https://bandwagonhost.com/cart.php?a=add&pid=145"
    assert "stock" contains "Out of Stock"
}
```

## DSL 实现原理

实现一个简单的 DSL 主要需要三个部分：

1. **词法分析器** (Lexer)：将代码文本分解成 token
2. **语法分析器** (Parser)：将 token 转换成 AST
3. **执行器** (Runner)：解释执行 AST

核心的词法分析代码：

```javascript
class Lexer {
    constructor(sourceCode) {
        this.tokenSpec = [
            ['TEST', /test/],
            ['OPEN', /open/],
            ['ASSERT', /assert/],
            ['STRING', /"[^"]*"/],
            ['CONTAINS', /contains/],
            // ... 其他 token 定义
        ];
    }

    tokenize() {
        // 将源代码转换为 token 流
    }
}
```

## 实际运行

项目使用 Puppeteer 执行监控任务：

```javascript
const browser = await puppeteer.launch();
const page = await browser.newPage();

// 执行 DSL 中定义的操作
for (const action of test.actions) {
    if (action.type === 'open') {
        await page.goto(action.url);
    } else if (action.type === 'assert') {
        // 检查库存状态
    }
}
```

## 使用方法

1. 克隆并安装：

```bash
git clone https://github.com/harrisonwang/vps-watcher.git
cd vps-watcher
npm install
```

2. 编写 DSL 规则并运行：

```bash
$ npm start

> vps-watcher@1.0.0 start
> node src/index.js

this.tokens [
  { type: 'TEST', value: 'test' },
  { type: 'STRING', value: '"Check Stock"' },
  { type: 'LBRACE', value: '{' },
  { type: 'OPEN', value: 'open' },
  {
    type: 'STRING',
    value: '"https://www.dmit.io/cart.php?a=add&pid=183"'
  },
  { type: 'ASSERT', value: 'assert' },
  { type: 'STRING', value: '"stock"' },
  { type: 'IDENTIFIER', value: 'contains' },
  { type: 'STRING', value: '"Out of Stock"' },
  { type: 'RBRACE', value: '}' }
]
ast [
  {
    "testName": "Check Stock",
    "actions": [
      {
        "type": "open",
        "url": "https://www.dmit.io/cart.php?a=add&pid=183"
      },
      {
        "type": "assert",
        "selector": "stock",
        "expected": "Out of Stock"
      }
    ]
  }
]
Running test: Check Stock
Opening URL: https://www.dmit.io/cart.php?a=add&pid=183
等待3秒...
dmit.io 库存状态文本: Out of Stock
dmit.io 暂无库存
```

这个项目很好地展示了如何使用 DSL 来简化配置和提升代码可读性。完整代码已开源在 [GitHub](https://github.com/harrisonwang/vps-watcher)。
