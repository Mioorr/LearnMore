---
SVG（Scalable Vector Graphics） 是一种基于 XML 语法的图像格式。其他图像都是基于像素的，SVG 则是属于对图像的形状描述，所以它本质上是文本文件，且体积小，不论放大多少倍都不会失真。
---

## 1. 概述

SVG 的使用：

1. SVG 代码可以直接插入网页，成为 DOM 的一部分，然后用 JS 和 CSS 进行操作。

   ```html
   <svg id="mysvg" xmlns="https://××××">
     <!-- ... -->
   </svg>
   ```

2. SVG 代码可以直接写在一个`.svg`文件中，然后用`<img>`、`<object>`、`<embed>`、`<iframe>`等标签插入网页。

   ```html
   <img src="circle.svg">
   <object id="object" data="circle.svg" type="image/svg+xml"></object>
   <embed id="embed" src="icon.svg" type="image/svg+xml">
   <iframe id="iframe" src="icon.svg"></iframe>
   ```

3. CSS 也可以引入`.svg`文件。

   ```css
   .logo {
     background: url(logo.svg);
   }
   ```

4. SVG 文件还可以转为 BASE 64 编码，然后作为 Data URI 写入网页。

   ```html
   <img src="data:image/svg+xml;base64,[data]" >
   ```

