# CSS 的特殊选择器

**第一类：元素选择器**

元素选择器很常见，这里就不过多赘述。

CSS 中有很多选择器，功能齐全，但往往会被我们忽视。



1. `*`：选择所有元素。
2. `[attribute]`：选择属性。

**伪类选择器：**

- `:`

**伪元素选择器：**

- `:before (:before)`：在备选元素内容前插入内容。

  ```css
  /* CSS2 语法 */
  p:before { /* style properties */ }
  /* CSS3 语法 */
  p::before { /* style properties */ }
  ```

  因为 CSS 的伪类选择器是以单引号开头的，为了区分伪类和伪元素选择器，CSS3 将伪元素选择器前面的`:`改为`::`。支持 CSS3 的浏览器同时也支持 CSS2 中引入的`:`写法，`:`已过时，仅用来支持 IE8-。

- `::after (:after)`：在被选元素内容后插入内容。

  同`::before`。

- 





MDN 选择器

<https://developer.mozilla.org/zh-CN/docs/Web/CSS/::before>

CSS 选择器参考手册

<http://www.w3school.com.cn/cssref/css_selectors.asp>

CSS 分组

<http://www.w3school.com.cn/css/css_selector_grouping.asp>