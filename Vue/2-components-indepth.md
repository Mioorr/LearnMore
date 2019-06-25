# 深入了解组件

## 1. 组件注册

### 1.1 组件名

在注册一个组件的时候，必须要给它一个名字，组件名就是`Vue.component()`的第一个参数。

```javascript
Vue.component('my-component-name', { /* ... */ })
```

**组件名定义方式**

定义组件名的方式有2种：

1. `component-name`短横线分隔命名方式。

   这种方式命名一个组件时，也必须在引用这个自定义元素时使用一样的名称（`<component-name>`）。

2. `ComponentName`单词首字母大写命名方式。

   这种方式命名一个组件时，在引用这个自定义组件时两种命名法都可以使用（即`<component-name>`和`<ComponentName>`）。但直接在 DOM（非字符串模板）中使用时只有短横线分隔命名方式是有效的。

> <font color=blue>自定义组件命名建议：</font>
>
> 当直接在 DOM 中使用一个组件（而不是在字符串模板或单文件组件）时，建议字母全小写且必须包含一个连字符。

### 1.2 组件注册方式

#### 1.2.1 全局注册

通过`Vue.component()`来创建组件：

```javascript
Vue.component('my-component-name', {
  // ... 选项 ...
})
```

这些组件是全局注册的，它们在注册之后可以用在任何新创建的 Vue 根实例（`new Vue()`）的模板中。所有的子组件也是一样。

#### 1.2.2 局部注册

全局注册的组件会被包含在最终的构建结果中。这造成了用户下载了很多无谓的 JS。

在这些情况下，可以通过一个普通的 JS 对象来定义组件，然后在`components`选项中定义要使用的组件：

```javascript
var ComponentA = { /* options ... */ }
var ComponentB = { /* options ... */ }
new Vue({
  el: '#app',
  components: {
    'components-a': ComponentA,
    'components-b': ComponentB
  }
})
```

对于`components`对象中的每个属性来说，属性名就是自定义元素的名字，其属性值就是这个组件的选项对象。

> <font color=red>Notice：</font>局部注册的组件在其子组件中不可用。
>
> 如，要在`ComponentA`在`ComponentB`中可用，则需要这样写：
>
> ```javascript
> var ComponentA = { /* options ... */ }
> var ComponentB = {
>   components: {
>     'component-a': ComponentA
>   },
>   // ...
> }
> ```
>
> 如果通过 Babel 和 webpack 使用 ES2015 模块，那么代码可以这样：
>
> ```javascript
> import ComponentA from './ComponentA.vue'
> export default {
>   components: {
>     ComponentA
>     // => ComponentA: ComponentA
>   },
>   // ...
> }
> ```

### 1.3 模块系统



















