# Grid 布局

网格（Grid）布局将网页划分为一个个网格，可以任意组合不同的网格，做出各种各样的布局。

弹性（Flex）布局是轴线布局，只能指定”项目“针对轴线的位置，可以看作**一维布局**；Grid 布局将容器划分为行和列，产生单元格，然后指定”项目“所在的单元格，可以看作**二维布局**。

## 1. 基本概念

### 1.1 容器和项目

和 Flex 类似，采用网格布局的区域称为**容器**（container），容器内部采用网格定位的子元素称为**项目**（item）。

### 1.2 行、列、单元格

容器里的水平区域称为**行**（row），垂直区域称为**列**（column）。

![row&col](./img/row&col.png)

行和列交叉区域称为**单元格**（cell）。正常情况下，`n`行和`m`列会产生`n x m`个单元格。

### 1.3 网格线

划分网格的线，称为**网格线**（grid line）。水平网格线划分出行，垂直网格线划分出列。

正常情况下，`n`行有`n + 1`根水平网格线，`m`列有`m + 1`根垂直网格线。

![grid line](./img/grid-line.png)

---

## 2. 容器属性

### 2.1 display

```css
display: grid; /* 块级网格（默认） */
display: inline-grid; /* 行内网格 */
```

设为网格布局后，容器子元素的`float`、`display: inline-block`、`display: table-cell`、`vertical-align`、`column-*`等设置都将失效。

### 2.2 grid-template-columns & grid-template-rows

`grid-template-columns`定义每列的列宽，`grid-template-rows`定义每行的行高。

```css
grid-template-rows: 100px 100px 100px;
grid-template-columns: 100px 100px 100px;
```

除了使用绝对单位，也可以使用百分比。

1. `repeat()`

   当要写连续的重复值时候可以使用`repeat(num, value)`函数。

   num：重复的次数；value：重复的值。

   ```css
   /* 重复一个值 */
   grid-template-rows: repeat(3, 100px);
   grid-template-columns: repeat(3, 100px);
   /* 重复一组值 */
   grid-template-rows: repeat(3, 100px 50px 200px);
   ```

2. `auto-fill`关键字

   有时，单元格的大小是固定的，但是容器的大小不确定。如果希望每一行（或每一列）容纳尽可能多的单元格，这时可以使用`auto-fill`关键字表示自动填充。

   ```css
   grid-template-columns: repeat(auto-fill, 100px);
   /* 表示每列宽 100px，自动填充，直至容器不能放置更多的列 */
   ```

3. `fr`关键字

   `fr`（片段`fraction`），这个值就像 Flex 布局的`flex-grow`属性。

   ```css
   grid-template-columns: 1fr 2fr;
   /* 表示第二列宽是第一列宽的两倍 */
   ```

   `fr`可以与绝对长度单位结合使用。

4. `minmax()`

   `minmax(min, max)`函数产生一个长度范围，表示长度就在`min ~ max`这个范围中。

   ```css
   grid-template-columns: 1fr 1fr minmax(100px, 1fr);
   ```

5. `auto`关键字

   `auto`关键字表示由浏览器自己决定长度。

   ```css
   grid-template-columns: 100px auto 100px;
   ```

6. 网格线的名称

   `grid-template-columns`属性和`grid-template-rows`属性里面，可以使用`[]`指定每一根网格线的名字，方便以后的引用。

   ```css
   grid-template-columns: [c1] 100px [c2] 100px [c3] auto [c4];
   grid-template-rows: [r1] 100px [r2] 100px [r3] auto [r4];
   ```

   网格布局允许同一根线有多个名字，比如`[fifth-line row-5]`。

### 2.3 grid-row-gap & grid-column-gap & grid-gap

`grid-row-gap`设置行间距，`grid-column-gap`设置列间距。

```css
grid-row-gap: 20px;
gird-column-gap: 20px;
```

`grid-gap`是上面两个属性的简写形式。

```css
grid-gap: <grid-row-gap> <grid-column-gap>;
```

如果省略了第二个值，浏览器认为第二个值等于第一个值。

### 2.4 grid-template-areas

网格布局允许指定"区域"（area），一个区域由单个或多个单元格组成。`grid-template-areas`属性用于定义区域。

```css
/* 划分出9个单位 */
grid-template-columns: 100px 100px 100px;
grid-template-rows: 100px 100px 100px;
/* 将其定名为a到i的九个区域 */
grid-template-areas: 'a b c'
                     'd e f'
                     'g h i';
```

多个单元格合并成一个区域：

```css
grid-template-areas: 'a a a'
                     'b b b'
                     'c c c';
```

某些区域不需要利用，则使用`.`表示。

### 2.5 grid-auto-flow

划分网格以后，容器的子元素会按照顺序，自动放置在每一个网格。默认的放置顺序是"先行后列"，即先填满第一行，再开始放入第二行。

```css
grid-auto-flow: row; /* (default)修改放置顺序为"先行后列" */
grid-auto-flow: column; /* 设置放置顺序为"先列后行" */
/* 尽可能紧密填满 */
grid-auto-flow: row dense;
grid-auto-flow: column dense;
```

![Difference between row & row dense](./img/row-dense.png)

 ### 2.6 justify-items & align-items & place-items

`justify-items`设置单元格内容的水平位置，`align-items`设置单元格内容的垂直位置。

这两个属性的写法完全相同，都可以取这些值：

- start：对齐单元格的起始边缘。
- end：对齐单元格的结束边缘。
- center：单元格内部居中。
- stretch：拉伸，占满单元格的整个宽度（默认值）。

![justify-items & align-items](./img/justify&align-items.png)

### 2.7 justify-content & align-content & place-content

`justify-content`是整个内容区域在容器里的水平位置，`align-content`是整个内容区域的垂直位置。

这两个属性可取的值：

- start --- 对齐容器的起始边框；
- end --- 对齐容器的结束边框；
- center --- 容器内部居中；
- stretch --- 项目大小没有指定时，拉伸占据整个网格容器；
- space-around --- 每个项目两侧的间隔相等。所以项目之间的间隔比项目与容器边框的间隔大一倍。
- space-between --- 项目与项目的间隔相等，项目与容器边框之间没有间隔。
- space-evenly --- 项目与项目的间隔相等，项目与容器边框之间也是同样长度的间隔。

`place-content`是这两个属性的合并简写形式。

```css
place-content: <align-content> <space-evenly>;
```

如果省略第二个值，浏览器就会假定第二个值等于第一个值。

### 2.8 grid-auto-columns & grid-auto-rows

有时候，一些项目的指定位置，在现有网格的外部。比如网格只有3列，但是某一个项目指定在第5行。这时，浏览器会自动生成多余的网格，以便放置项目。

`grid-auto-columns`属性 grid-auto-rows`属性用来设置，浏览器自动创建的多余网格的列宽和行高。它们的写法与`grid-template-columns`和`grid-template-rows`完全相同。如果不指定这两个属性，浏览器完全根据单元格内容的大小，决定新增网格的列宽和行高。

```css
grid-auto-rows: 50px; /* 指定新增的行高为 50px */
```

### 2.9 grid-template & grid

`grid-template`是`grid-template-columns`、`grid-template-rows`、`grid-template-areas`这3个属性的合并简写形式。

`grid`属性是`grid-template-rows`、`grid-template-columns`、`grid-template-areas`、 `grid-auto-rows`、`grid-auto-columns`、`grid-auto-flow`这六个属性的合并简写形式。

## 3. 项目属性

### 3.1 grid-column-start & grid-column-end & grid-row-start & grid-row-end

项目的位置是可以指定的，具体方法就是指定项目的4个边框，分别定位在哪根网格线。

- `grid-column-start`：左边框所在的垂直网格线；
- `grid-column-end`：右边框所在的垂直网格线；
- `grid-row-start`：上边框所在的水平网格线；
- `grid-row-end`：下边框所在的水平网格线。

```css
.item-1 {
  grid-column-start: 2; /* 从第2列网格线开始 */
  grid-column-end: 4;   /* 到第4列网格线结束 */
  grid-row-start: 2;    /* 从第2行网格线开始 */
  grid-row-end: 4;      /* 到第4列网格线结束 */
}
```

这 4 个属性的值可以是：

- 数字，表示第几根网格线；

- 网格线的名字；

- `span`关键字，表示“跨越”多少个网格。

  ```css
  grid-column-start: span 2; /* 表示左右跨越2个网格 */
  ```

使用这四个属性，如果产生了项目的重叠，则使用`z-index`属性指定项目的重叠顺序。

#### grid-column & grid-row

`grid-column`属性是`grid-column-start`和`grid-column-end`的合并简写形式，`grid-row`属性是`grid-row-start`属性和`grid-row-end`的合并简写形式。

```css
grid-column: 1 / 3;
grid-row: 1 / 2;
/* 等同于 */
grid-column-start: 1;
grid-column-end: 3;
grid-row-start: 1;
grid-row-end: 2;
/* 斜杠以及后面的部分可以省略，默认跨越一个网格 */
grid-column: 1;
grid-row: 1;
```

### 3.2 grid-area

`grid-area`可指定项目的位置。

1. 指定项目的区域：

   ```scss
   .container {
     grid-template-area: 'a b c'  'd e f'  'g h i';
     .item-1 {
       grid-area: e; // 指定项目放在e区域
     }
   }
   ```

2. 作为`grid-row-start`、`grid-column-start`、`grid-row-end`、`grid-column-end`的合并简写形式，直接指定项目的位置：

   ```scss
   grid-area: <row-start> / <column-start> / <row-end> / <column-end>;
   ```

### 3.3 justify-self & align-self & place-self

`justify-self`属性设置单元格内容的水平位置（左中右），跟`justify-items`属性的用法完全一致，但只作用于单个项目。

`align-self`属性设置单元格内容的垂直位置（上中下），跟`align-items`属性的用法完全一致，也是只作用于单个项目。 

这两个属性都可以取下面四个值：

- start：对齐单元格的起始边缘。
- end：对齐单元格的结束边缘。
- center：单元格内部居中。
- stretch：拉伸，占满单元格的整个宽度（default）。

`place-self`属性是`align-self`属性和`justify-self`属性的合并简写形式。

```css
place-self: <align-self> <justify-self>;
```