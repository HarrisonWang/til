---
author: 小王爷
pubDatetime: 2024-10-30T20:46:00+08:00
modDatetime: 2024-10-30T20:46:00+08:00
title: "批量转换图片为 webp 格式脚本"
draft: false
tags:
  - webp
  - sharp
description:
  "由于博客图片比较多，都是 jpg 和 png 格式，为了加速网页加载，减小图片体积，所以随手糊了一个程序批量将 jpg 和 png 图片批量转为 webp 格式。"
---

由于博客图片比较多，都是 jpg 和 png 格式，为了加速网页加载，减小图片体积，所以随手糊了一个程序批量将 jpg 和 png 图片批量转为 webp 格式。

## 批量将 jpg 和 png 图片转换为 webp 格式

1. 提供待批量转换的图片目录 img。

2. 创建 batch2webp.js 文件，添加以下内容：

```js
import sharp from 'sharp';
import fs from 'fs';
import path from 'path';

// 输入目录路径
const inputDir = "./img";

// 读取目录中的所有文件
fs.readdir(inputDir, (err, files) => {
  if (err) {
    return console.error('无法读取目录:', err);
  }

  // 过滤出 JPG 和 PNG 文件
  const imageFiles = files.filter(file => {
    const ext = path.extname(file).toLowerCase();
    return ext === '.jpg' || ext === '.jpeg' || ext === '.png';
  });

  // 遍历并将每个文件转换为 WebP
  imageFiles.forEach(file => {
    const inputFilePath = path.join(inputDir, file);
    const outputFileName = path.basename(file, path.extname(file)) + '.webp';
    const outputFilePath = path.join(inputDir, outputFileName);

    // 处理 PNG 和 JPG 文件的不同情况
    const ext = path.extname(file).toLowerCase();
    if (ext === '.png') {
      // 如果是 PNG 文件，保留透明度
      sharp(inputFilePath)
        .webp({ lossless: true })  // 保留透明背景，使用无损压缩
        .toFile(outputFilePath)
        .then(info => {
          console.log(`${file} (PNG) 转换成功为 ${outputFileName}`);
        })
        .catch(err => {
          console.error(`转换 PNG 文件 ${file} 时出错:`, err);
        });
    } else if (ext === '.jpg' || ext === '.jpeg') {
      // 对于 JPG 文件，正常转换
      sharp(inputFilePath)
        .webp({ quality: 80 })  // 使用有损压缩，设置质量
        .toFile(outputFilePath)
        .then(info => {
          console.log(`${file} (JPG) 转换成功为 ${outputFileName}`);
        })
        .catch(err => {
          console.error(`转换 JPG 文件 ${file} 时出错:`, err);
        });
    }
  });
});
```

3. 执行 `node batch2webp.js` 命令进行批量转换，转换结果下：

   ```bash
   $ node batch2webp.js
   image-20231104165125190.png (PNG) 转换成功为 image-20231104165125190.webp
   image-20231104165416936.png (PNG) 转换成功为 image-20231104165416936.webp
   image-20231104170322010.png (PNG) 转换成功为 image-20231104170322010.webp
   image-20231104165812848.png (PNG) 转换成功为 image-20231104165812848.webp
   ```
