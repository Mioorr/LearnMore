# CSS 选择器

CSS 中有很多选择器，功能齐全，但往往会被我们忽视。

---

## 一、基本选择器

1. **通配符选择器**

   `*`：选择所有元素，也可以选择某个元素下的所有元素。

2. **元素选择器**

   元素选择器是 CSS 中最基本且最常见的选择器，就是通过 HTML 元素名进行选择，这里就不罗列和赘述。

3. **ID 选择器**

   `#idName`：常用，不赘述。

4. **类选择器**

   `.className`：常用，不赘述。

5. **后代选择器**

   `E F`：用空格，在 E 的后代中选择所有和 F 匹配的元素，也比较常用。

6. **相邻兄弟选择器**

   `E + F`：选择紧接在 E 元素后且和 F 匹配的元素。

7. **通用兄弟选择器**

   `E ~ F`：在 E 后的所有兄弟元素中选择与 F 匹配的元素。

---

## 二、属性选择器

1. `[attr]`：选择有某个属性的元素，不管这个属性值是什么。
2. `[attr="value"]`：选择有某个属性其值为 value 的元素。

以下的功能语法类似正则表达式。

3. `[arrt^="value"]`：选择某个属性的值以 value 开头的元素。

4. `[attr$="value"]`：选择某个属性的值以 value 结尾的元素。
5. `[attr~="value"]`：选择某个属性值的一个或多个词列表中存在 value 值的元素，如果是列表，他们需要用空格隔开，只要属性值中有一个 value 相匹配就可以选中该元素。
6. `[attr*="value"]`：选择某个属性的值中有 value 的元素，这个要区别于`~`的用法。
7. `[attr|="value"]`：这个被称作特定属性选择器，这个选择器会选择 attr 属性值等于`value`或以 `value-`开头的所有元素。

## 三、伪类选择器

1. **UI 元素状态伪类**

   - `:enabled`、`:disabled`、`:checked` 主要针对于 HTML 中的 Form 元素操作。
   - `:link`未被访问的、`:visited`已访问的、`:active`激活的`:hover`指针经过、`:focus`获得焦点的、`:target`当前活动的。
   - `:focus-within`：一个元素获得焦点，该元素的后代元素全部获得焦点。

2. **CSS3 的 :nth 选择器**

   - `:first-child`：选择某个元素的第一个子元素。

   - `:last-child`：选择某个元素的最后一个子元素。

   - `:nth-child`

     `:nth-child(length)`：length 是具体的整数，下同。

     `:nth-child(n)`：参数是 n，执行时 n 会自动从 0 开始计算，下同。

     `:nth-child(length*n)`：选择 n 的倍数。

     `:nth-child(length+n)`：选择 > length 后面的元素。

     `:nth-child(length-n)`：选择 < length 后面的元素。

     `:nth-child(length*n+1)`：表示隔几选1。

   - `:nth-last-child()`：选择某个元素的一个或多个特定的子元素，从这个元素的最后一个子元素开始算。

   - `:nth-of-type()`：选择指定的元素。

   - `:nth-last-of-type()`：选择指定的元素，从元素的最后一个开始计算。

   - `:first-of-type`：选择一个上级元素下的第一个同类子元素。

   - `:last-of-type`：选择一个上级元素下的第一个同类子元素。

   - `:only-child`：选择一个上级元素的最后一个同类子元素。

   - `:only-of-type`：选择一个元素是它的父元素的唯一一个子元素。

   - `:empty`：选择的元素里没有任何内容。

   - `:first-letter`：选择某元素的首字母。

   - `:first-line`：选择某元素的首行。

3. `:fullscreen`：当前处于全屏显示模式的元素。它不仅选择顶级元素，还包括所有已显示的栈内元素。

   > W3C 标准使用不带破折号的单词，但 Webkit 和 Gecko 应用接口各自使用前缀带有破折号的变量`:-webkit-full-screen`、`-moz-full-screen`。微软的 Edge 和 Internet Explorer 各自使用标准语法`:fullscreen`、`:-ms-fullscreen`。

4. `:not(x)`：否定选择器。一个简单的以选择器 x 为参数的功能性标记函数。匹配不符合参数选择器 x 描述的元素，x 不能包含另外一个否定选择器。

   > <font color=red>Notice：</font>
   >
   > -  `:not(*)` 匹配任何非元素的元素，因此这个规则将永远不会被应用。
   > - 利用这个伪类提高规则的优先级。 `#foo:not(#bar)` 和 `#foo` 会匹配相同的元素。 但是前者的优先级更高。
   > - `:not(foo)` 将匹配任何非foo元素，包括html和body。
   > - 这个选择器只会应用在一个元素上，无法用它排除所有父元素。比如， `body :not(table) a` 将会应用在table内部的[`<a>`上, 因为`<tr>`将会被 `:not()`这部分选择器匹配。

## 四、伪元素选择器

1. `:before (:before)`：在备选元素内容前插入内容。

   ```css
   /* CSS2 语法 */
   p:before { /* style properties */ }
   /* CSS3 语法 */
   p::before { /* style properties */ }
   ```

   因为 CSS 的伪类选择器是以单引号开头的，为了区分伪类和伪元素选择器，CSS3 将伪元素选择器前面的`:`改为`::`。支持 CSS3 的浏览器同时也支持 CSS2 中引入的`:`写法，`:`已过时，仅用来支持 IE8-。

2. `::after (:after)`：在被选元素内容后插入内容。

   同`::before`。

3. `::section`：选择被用户选择的元素。

   只有一小部分 CSS 属性可以用于这个选择器：`color`、`background-color`、`cusor`、`caret-color`、`outline`、`text-decoration`、`text-emphasis-color`、`text-shadow`。

   > <font color=red>Notice：</font>`background-image`会和其他属性一样被忽略。

   