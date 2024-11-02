---
author: 小王爷
pubDatetime: 2024-10-30T11:09:00+08:00
modDatetime: 2024-10-30T11:09:00+08:00
title: "jpg 和 png 转 webp"
draft: false
tags:
  - webp
  - sharp
description:
  "使用 sharp 将 jpg 和 png 转换为 webp 格式图片"
---

博客站点所在的机房位于美西，因主要是国内用户访问，为了加快文章页面的访问速度，所以需要将图片进行压缩以提升页面加载速度，搜索后发现 webp 格式是一种兼顾高压缩率和高质量的图像格式，故编写了这样一个图片转换程序。

## jpg 和 png 转 webp 格式

### 初始化项目

创建一个目录：

```bash
mkdir -p /opt/image2webp
```

切换至目录：

```bash
cd /opt/image2webp
```

初始化项目：

```bash
npm init -y
```

安装 sharp 依赖包：

```bash
npm i sharp
```

启用 ES 模块支持：

```diff
{
  "name": "image2webp",
  "version": "1.0.0",
  "main": "index.js",
+  "type": "module",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "description": "",
  "dependencies": {
    "sharp": "^0.33.5"
  }
}
```

### jpg 转 webp

创建文件：

```bash
touch jpg2webp.js
```

编辑文件添加以下内容：

```js
import sharp from "sharp";

const inputPath = "input.jpg";
const outputPath = "output.webp";

async function jpgToWebp(input, output) {
  try {
    const info = await sharp(input)
      .toFormat("webp") // 设置输出格式为 WebP
      .toFile(output); // 保存为 WebP 文件
    console.log("转换成功:", info);
  } catch (error) {
    console.error("转换出错:", error);
  }
}

jpgToWebp(inputPath, outputPath);
```

任意准备一张名为 `input.jpg` 的图片，执行下面的命令进行转换后的输出如下：

```bash
$ node jpg2webp.js
转换成功: {
  format: 'webp',
  width: 1888,
  height: 4096,
  channels: 3,
  premultiplied: false,
  size: 105580
}
```

### png 转 webp

创建文件：

```bash
touch png2webp.js
```

编辑文件添加以下内容：

```js
import sharp from "sharp";

const inputPath2 = "input.png";
const outputPath2 = "output2.webp";

async function pngToWebp(input, output) {
  try {
    const info = await sharp(input)
      .webp({
        lossless: true, // 使用无损压缩
        quality: 100, // 设置最高质量
        alphaQuality: 100, // 设置透明通道的最高质量
        effort: 6, // 使用最高压缩努力级别（0-6）
      })
      .toFile(output);
    console.log("转换成功:", info);
  } catch (error) {
    console.error("转换出错:", error);
  }
}

pngToWebp(inputPath2, outputPath2);
```

任意准备一张名为 `input.png` 的图片，执行下面的命令进行转换后的输出如下：

```bash
$ node png2webp.js
转换成功: {
  format: 'webp',
  width: 2160,
  height: 1187,
  channels: 3,
  premultiplied: false,
  size: 35274
}
```
