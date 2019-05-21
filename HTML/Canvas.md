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

### 1.2 渲染上下文

canvas 元素创造一个固定大小的画布，公开一个或多个**渲染上下文**，它可以用来绘制和处理要展示的内容。

canvas 开始是空白的。首先脚本要找到渲染上下文，然后在这上面绘制。

canvas 元素有一个`getContext()`方法，用来获取上下文和它的绘画功能。方法只有一个参数：上下文格式。

通过测试`getContext()`方法是否存在，可以检查浏览器对 canvas 的支持性。

```javascript
var canvas = document.getElementById('tutorial');
if (canvas.getContext) {
  var ctx = canvas.getContext('2d');
  // ...
} else {
  // ...
}
```

## 2. 绘制形状

### 2.1 画布栅格

canvas 元素默认被网格覆盖，通常网格中的一个单元相当于 Canvas 中的 1 像素。栅格的起点为左上角(0, 0)，所有图像的位置都相对于原点定位。



![canvas栅格](./img/canvas栅格.png)

### 2.2 绘制矩形

HTML 中的 Canvas 只支持一种原生的图形绘制：矩形。所有其他的图形绘制都需要至少生成一条路径。

Canvas 提供了 3 种方法绘制矩形：

1. `fillRect(x, y, width, height)`：绘制一个填充的矩形。
2. `strokeRect(x, y, width, height)`：绘制一个矩形的边框。
3. `clearRect(x, y, width, height)`：清除指定矩形区域，让清除部分完全透明。

`x`、`y`指定了在 Canvas 画布上所绘制的矩形相对于原点的坐标；`width`和`height`设置矩形尺寸。

### 2.3 绘制路径

图形的基本元素是路径。路径是通过不同颜色和宽度的线段或曲线相连形成的不同形状的点的集合。一个路径，甚至一个子路径都是闭合的。使用路径绘制图形需要一些额外的步骤：

1. 创建路径起始点；
2. 使用画图命令去画出路径；
3. 把路径封闭；
4. 一旦路径生成，就能通过描边或填充路径区域来渲染图形。

要用到的函数：

- `beginPath()`：新建一条路径，生成之后，图形绘制命令被指向到路径上生成路径。
- `closePath()`：闭合路径后图形绘制命令又重新指向到上下文中。
- `stroke()`：通过线条绘制图形轮廓。
- `fill()`：通过填充路径的内容区域生成实心的图形。









```html
<html>
<head>
	<script type="application/javascript">
    function draw() {
      var canvas = document.getElementById("canvas");
      if (canvas.getContext) {
        var ctx = canvas.getContext("2d");
        ctx.fillStyle = "rgb(200,0,0)";
        ctx.fillRect (10, 10, 55, 50);
        ctx.fillStyle = "rgba(0, 0, 200, 0.5)";
        ctx.fillRect (30, 30, 55, 50);
      }
    }
  </script>
</head>
<body onload="draw();">
<!-- 在页面加载完执行draw() -->
  <canvas id="canvas" width="150" height="150"></canvas>
</body>
</html>
```





 





































