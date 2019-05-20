# Canvas

## 1.基本用法

```html
<canvas id="tutorial" width="150" height="150"></canvas>
```

`<canvas>`实际上只有两个属性——`width`和`height`，当没有设置宽高时，`canvas`会初始化宽 300px 高 150px。该元素可以使用 CSS 来定义大小，在绘制时图像会伸缩以适应框架尺寸，如果CSS的尺寸与初始画布的比例不一致，图像会出现扭曲。

> <font color=blue>Notice：</font>如果绘制出来的图像是扭曲的，试着用`width`和`height`属性为 canvas 明确规定宽高。

canvas 元素可以像任何普通图像一样（有`margin`、`border`等属性），这些样式不会影响在 canvas 中的实际图像。没有规定样式的 canvas 是完全透明的。

### 1.1 替换内容

canvas 可以设置替换内容。不支持 canvas 标签的浏览器将会忽略 canvas 容器并在其中渲染后备内容。

```html
<canvas id="clock" width="150" height="150">
  <!-- 不支持canvas的浏览器将会忽略外部canvas,只渲染img标签 -->
  <img src="images/clock.png" width="150" height="150" alt=""/>
</canvas>
```

`</canvas>`结束标签不能省略，否则文档的其余部分会被认为是替代内容，不会显示出来。

## 1.2 渲染上下文

canvas 元素创造一个固定大小的画布，公开一个或多个**渲染上下文**，它可以用来绘制和处理要展示的内容。

canvas 开始是空白的。首先脚本要找到渲染上下文，然后在这上面绘制。

canvas 元素有一个`getContex()`方法，用来获取上下文和它的绘画功能。方法只有一个参数：上下文格式。

```javascript
var canvas = document.getElementById('tutorial');
var ctx = canvas.getContext('2d');
```



 





































