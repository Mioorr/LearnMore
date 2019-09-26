# Transform 对元素渲染的影响

CSS 的 `transfrom` 属性真的让人又爱又恨，虽然功能挺不错，但有些情况下，还是常常对页面产生很多莫名其妙的影响。

## 1. Transfrom 提升元素堆叠上下文顺序

> 当元素未设置`float`、`position`都为`static`的情况下，遇到重叠被覆盖的时候，给被覆盖的元素设置`transform`样式，被覆盖的元素层级会被提升，但是会受`opacity`属性的影响。

来看看 HTML 结构：

```html
<div class="left"></div>
<div class="right"></div>
```

在相同的堆叠上下文中，元素的层级按渲染顺序展示，后渲染的元素会在上面。

```scss
.left, .right {
  display: inline-block;
  width: 200px;
  vertical-align: middle;
}
.left {
  height: 120px;
  background-color: #8ed7ff;
}
.right {
  height: 100px;
  margin-left: -50px;
  background-color: pink;
}
```

这个时候在页面上的效果是，right 覆盖在了 left 上：

![right覆盖在left上](./img/transform-stack-1.png)

这个时候，给左边的元素设置`transform`：

```scss
.left {
  transform: translate(0, 0); // +
}
```

<font color=red>！</font>奇迹发生了————————————`transform`可以提升元素的堆叠顺序：

![left 在 right 上](./img/transform-stack-2.png)

但是，这个时候给两个元素设置`opacity`会发生什么呢？

```scss
.left, .right {
  opacity: .8; // +
}
```

<font color=red>！</font>这两个元素的堆叠层级恢复了（只给 right 设置`opacity`结果也一样）：

![设置透明度后元素的层级恢复了](./img/transform-stack-3.png)

`opacity`可以恢复因`transform`对元素造成的层级（按渲染顺序的层级）变化。当然，给元素设置`position`和层级也可以达到同样的效果。

## 2.  Transform 限制 position: fixed 的窗口固定

> 设置了`transform`元素使得子元素的`position:fixed`窗口固定效果变成`position: absolute`。

HTML 结构：

```html
<div class="container">
  <div class="child"></div>
</div>
```

一般来说，设置了`position: fixed`的元素都能根据窗口固定：

```scss
.container{
  height: 200vh;
  .child {
    width: 250px; 
    height: 250px;
    background-color: pink;
    position: fixed;
    top: calc(50vh - 125px);
    right: calc(50vw - 125px);
  }
}
```

这个时候，child 会脱离父元素的文本流，无论页面如何滚动，它都固定在窗口的中间。

<font color=red>！</font>但是当父元素设置了`transform`，它的`fixed`就失效了，和`absolute`时候的效果一样，会随着窗口滚动了。

## 3. Transform 改变 overflow 对 absolute 子元素的限制

> 对于`position: absolute`的元素，其父元素的`overflow`对
>
> 设有`transform`样式的容器，当同时设有`overflow: hidden`样式，则其`absolute`的子元素溢出也会隐藏。

来看下 HTML 结构：

```html
<div class="container">
  <div class="child"></div>
</div>
```

当容器设置了溢出隐藏，其子元素溢出容器的部分自然会被隐藏，但是当子元素设置了绝对定位，脱离了正常文本流，在正常文本流中的位置不再存在，容器的溢出隐藏对这个绝对定位的子元素就没有用了（当然，容器设置相对定位的情况另谈）。

```scss
.container {
  width: 200px;
  height: 200px;
  overflow: hidden; // 父元素隐藏
  border: blue;
  .child {
    width: 250px;
    height: 250px;
    background: pink;
    position: absolute; // 子元素绝对定位
  }
}
```

此时页面上渲染出来的效果是这样的：

![子元素溢出](./img/absolute.png)

当容器设置了`transform`属性，这个时候`overflow: hidden`对这个绝对定位的子元素就生效了。

```scss
.container {
  transform: translate(0, 0); // +
}
```

此时页面上，子元素溢出容器的部分就被隐藏了：

![子元素溢出隐藏](./img/transform-absolute.png)

有人可能会想，设一设`z-index`就回去了吧？ 然而不能，无论子元素的层级设多高，都是一样的结果。

你以为就这样结束了吗？ 在支持`transform`属性的 IE 浏览器上会出问题！设置了`translate(0, 0)`的容器会认为是没设，只有当其值不为全0的时候，溢出隐藏才会生效。那怎么办？为了视觉上没差，我使用`translate3d(0, 0, 1px)`对垂直于屏幕的方向设置值，这样就不是全 0 的值了，IE 上的子元素又能继续隐藏了。

## 4. Transform 限制 absolute 的百分比宽度大小

> 一般设置`absolute`元素的百分比宽度大小时，是以上级第一个非`static`值的`position`祖先元素的宽计算，如果没有，就参照 window。但实际上，`transform`也会影响这个参照祖先选取，所以实际上的选取规则应该：是以第一个`position`不为`static`值或设置了`transform`值的祖先元素。

```scss
.container{
  width: 500px;
  height: 500px;
  border: 5px solid $light-blue;
  transform: translate(0, 0);
  .child {
    width: 50%;
    height: 250px;
    background-color: pink;
    position: absolute;
  }
}
```

此时 child 的宽是 container 宽的一半。















