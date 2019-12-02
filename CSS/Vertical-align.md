# vertical-align

> 有些时候 `vertical-align` 的渲染结果并不同我们想象的一样。
> 虽然现在已经有很多 CSS 属性可以完美替代 `vertical-align` 的渲染效果，但作为一个Front-End Programmer，怎么能不搞清楚它？

<font color="red">☆</font>**作用对象**：`inline`、`inline-block`子元素。

这个属性失效的情况：

1. 作用对象是块级元素（因为块级元素会霸占整行）；
2. 作用元素设置了浮动、定位等脱离文本流的属性（父元素设置这些属性不影响）；
3. 父元素是`flex`或`grid`布局（作用元素设置这些布局不影响）。

## 1. 属性值

可选参数：

|      值       |                             描述                             |
| :-------------: | :----------------------------------------------------------: |
|  `baseline`   |           (default) 元素放在父元素的基线上。           |
|     `top`     |           元素的顶端与行中最高元素的顶端对齐           |
|  `text-top`   |      元素的顶端与父元素内容区域的顶端(文字顶端)对齐。      |
|   `middle`    | 元素在父元素内垂直居中。 |
|   `bottom`    |          元素底端与行中最低元素底端对齐。           |
| `text-bottom` |           元素底端与父元素文本的底端对齐。           |
|     `sub`     |            降低元素的基线到父元素合适的下标位置。            |
|    `super`    |            升高元素的基线到父元素合适的上标位置。            |
|   `inherit`   |              采用父元素相关属性的相同的指定值。              |
|     长度      |                   通过距离升高或降低元素。                   |
|       %       | 通过相对于`line-height`属性的百分值升高或降低元素。 |

看表格的描述太抽象，还不如来点儿实际的～

HTML 代码：

```html
<div class="container">
  Father Text
  <span class="content">
  	Child Text
  </span>
  <img class="align" src="./img/xx.png" alt="vertical align image">
  <!-- 下面的图片不作特定垂直对齐,作为一个参照 -->
  <img src="./img/**.png" alt="reference image">
</div>
```

1. **`baseline`**：元素放在父元素的基线上。

   实际上文本子元素是文本内容放在父元素基线上。因为这是默认值，其他子元素都是`baseline`。
   
   ![baseline](./img/vertical-align-1.png)
   
2. **`top`**：元素的顶端与行中最高元素的顶端对齐。

   ```css
   .align { vertical-align: top; }
   ```

   从`baseline`看，这里的最高元素是参照图。

   ![top](./img/vertical-align-2.png)

3. **`text-top`**：元素的顶端与父元素字体的顶端对齐。

   ![text-top](./img/vertical-align-3.png)

4. **`middle`**：元素在父元素内垂直居中。

   可以看出，`middle`实际是指元素中部与父元素文本中部对齐。

   ![middle](./img/vertical-align-4.png)

5. **`bottom`**：元素底端与行中最低元素底端对齐。

   ```css
   .align { vertical-align: bottom; }
   ```

   从`baseline`可以看出这里最低元素是 content 元素。

   ![bottom](./img/vertical-align-5.png)

6. **`text-bottom`**：元素底端与父元素文本底端对齐。

   > <font color="red">！</font>这和`baseline`是不一样的。

   ![text-bottom](./img/vertical-align-6.png)

7. **长度值**：使用定长表示元素底部与父元素基线的距离。

   正值升高元素，负值降低元素。0值等同于`baseline`。 

   ```scss
   .align {
     vertical-allign: 10px; // 元素相对于基线向下偏移2px
   }
   ```

   ![10px](./img/vertical-align-7.png)

8. **百分比**：元素底部相对于父元素基线移动相对于元素行高`line-height`的百分比。

   这里百分比是相对于元素的`line-height`值。

   ```scss
   .align { vertical-align: -10%; }
   ```

   假设`.align`本身未设置行高，继承的行高是`20px`，这里的`-10%`代表的实际值是`-10% × 20 = -2px`，所以是让元素下降 2 像素。

   ![-2px](./img/vertical-align-8.png)

   设置`.align`行高为`100px`，这里的`-10%`代表的实际值是`-10% × 100 = -10px`，所以是让元素下降 10 像素。

   ![-10px](./img/vertical-align-9.png)
   
   > <font color="red">！</font>IE6 / 7 下的`vertical-align`百分比值不支持小数的`line-height`。


