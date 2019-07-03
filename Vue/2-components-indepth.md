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

**在模块系统中局部注册**

需要在局部注册之前导入要使用的组件：

```javascript
import ComponentA from './ComponentA'
import ComponentB from './ComponentB'

export default {
  components: {
    ComponentA,
    ComponentB
  },
  // ...
}
```

**基础组件的自动化全局注册**

有些通用的组件，要在很多组件中被频繁用到。如果用 webpack 包管理工具，那就可以使用`require.context`只全局注册这些非常通用的组件。

```javascript
import Vue from 'Vue'
import upperFirst from 'lodash/upperFirst'
import camelCase from 'lodash/camelCase'

const requireComponent = require.context(
  './components', // 组件目录的相对路径
  false, // 是否查询子目录
  /Base[A-Z]\w+\.(vue|js)$/ // 匹配基础组件文件名的正则表达式 
)

requireComponent.keys().forEach(fileName => {
  // 获取组件配置
  const componentConfig = requireComponent(fileName)
  // 获取组件的 PascalCase 命名
  const componentName = upperFirst(
    camelCase(
      // 获取和目录深度无关的文件名
      fileName
        .split('/')
        .pop()
        .replace(/\.\w+$/, '')
    )
  )
  // 全局注册组件
  Vue.component(
    componentName,
    // 如果这个组件选项是通过 `export default` 导出的，
    // 那么就会优先使用 `.default`，否则回退到使用模块的根。
    componentConfig.default || componentConfig
  )
})
```

> <font color=pink>Remember：</font>全局注册的行为必须在根 Vue 实例创建==之前==发生。

---

## 2. Prop

### 2.1 Prop 的大小写

HTML 的特性名是大小写不敏感的，浏览器会把所有大写自读解释为小写字符。所以当使用 DOM 中的模板时，驼峰命名法的 prop 名需要使用等价的短横线分隔命名：

```javascript
Vue.component('blog-post', {
  props: ['postTitle'], // 在 JS 中是驼峰式
  template: '<h3>{{ postTitle }}</h3>'
})
```

```html
<!-- 在HTML中是短横线分隔式 -->
<blog-post post-title="hello"></blog-post>
```

> 如果使用字符串模板，就没有这个限制了。

### 2.2 Prop 类型

Prop 可以以字符串数组形式列出：

```javascript
props: ['title', 'likes', 'commentIds']
```

也可以以对象形式列出来指定每个 prop 值的类型，这些属性的名称和值分别是 prop 各自的名称和类型：

```javascript
props: {
  title: String,
  likes: Number,
  commentIds: Array
}
```

### 2.3 传递静态或动态 Prop

**传递静态 prop**

```html
<blog-post title="My journey with Vue"></blog-post>
```

**传递动态 prop**

通过`v-bind`动态赋值。

```html
<!-- 动态赋予一个变量的值 -->
<blog-post v-bind:title="post.title"></blog-post>
<!-- 动态赋予一个复杂表达式的值 -->
<blog-post
  v-bind:title="post.title + ' by ' + post.author.name"
></blog-post>
```

#### 2.3.1 传入一个数字

```html
<!-- 虽然42是静态的，仍然需要v-bind来告诉Vue这是一个 JS 表达式而不是一个字符串 -->
<blog-post v-bind:likes="42"></blog-post>
<!-- 用一个变量进行动态赋值 -->
<blog-post v-bind:likes="post.likes"></blog-post>
```

#### 2.3.2 传入一个布尔值

```html
<!-- 包含 prop 没有值的情况在内，都意味着true -->
<blog-post is-published></blog-post>
<!-- false是静态的，仍然需要v-bind告诉Vue这是一个JS表达式 -->
<blog-post v-bind:is-published="false"></blog-post>
<!-- 用一个变量进行动态赋值 -->
<blog-post v-bind:is-published="post.isPublished"></blog-post>
```

#### 2.3.3 传入一个数组

```html
<!-- 数组是静态的，仍需要v-bind告诉Vue这是一个JS表达式 -->
<blog-post v-bind:comment-ids="[234, 266, 273]"></blog-post>
<!-- 用一个变量进行动态赋值 -->
<blog-post v-bind:comment-ids="post.commentIds"></blog-post>
```

#### 2.3.4 传入一个对象

```html
<blog-post
  v-bind:author="{
    name: 'Vernoica',
    company: 'Veridian Dynamics'
  }"
></blog-post>
<!-- 用一个变量进行动态赋值 -->
<blog-post v-bind:author="post.author"></blog-post>
```

#### 2.3.5 传入一个对象的所有属性

如果要将一个对象的所有属性都作为 prop 传入，可以使用不带参数的`v-bind`（取代`v-bind:prop-name`）。

```javascript
post: {
  id: 1,
  title: 'My Journey with Vue'
}
```

```html
<blog-post v-bind="post"></blog-post>
<!-- 等价于 -->
<blog-post v-bind:id="post.id" v-bind:title="post.title"></blog-post>
```

### 2.4 单向数据流

所有的 prop 都使得其父子 prop 之间形成了一个**单向下行绑定**：父级 prop 的更新会向下流动到子组件中，但是反过来则不行。这样能防止从子组件意外改变父级组件的状态，从而导致应用的数据流向难以理解。

另外，每次父级组件发生更新时，子组件中所有的 prop 都将会刷新为最新的值。因此，不应该在一个子组件内部改变 prop，否则 Vue 会在浏览器的控制台中发出警告。

有两种常见的试图改变一个 prop 的情形：

1. **prop 用来传递一个初始值这个子组件接下来希望将其作为一个本地的 prop 数据来使用**。

   这种情况下，最好定义一个本地的 data 属性并将这个 prop 用作其初始值：

   ```javascript
   props: ['initialCounter'],
   data: function () {
     return {
       counter: this.initialCounter
     }
   }
   ```

2. **prop 以一种原始的值传入且需要进行转换**。

   这种情况下，最好使用这个 prop 的值来定义一个计算属性：

   ```javascript
   props: ['size'],
   computed: {
     normalizedSize: function () {
       return this.size.trim().toLowerCase()
     }
   }
   ```

> <font color=orange>Notice：</font>在 JS 中对象和数组是通过引用传入的，所以对于一个数组或对象类型的 prop 来说，在子组件中改变这个对象或数组本身==会==影响到父组件的状态。

### 2.5 Prop 验证

为了定义 prop 的验证方式，可以为`props`中的值提供一个带有验证需求的对象，而不是一个字符串数组。

```javascript
Vue.component('my-component', {
  props: {
    // 基础的类型检查
    propA: Number,
    // 多个可能的类型
    propB: [String, Number],
    // 必填的字段
    propC: {
      type: String,
      require: true
    },
    // 带有默认值的数字
    propD: {
      type: Number,
      default: 100
    },
    // 带有默认值的对象
    propE: {
      type: Object,
      default: function () {
        return { message: 'hello' }
      }
    },
    // 自定义验证函数
    propF: {
      validator: function (value) {
        // 这个值必须匹配下列字符串中的一个
        return ['success', 'warning', 'danger'].indexOf(value) !== -1
      }
    }
  }
})
```

当 prop 验证失败的时候，（开发环境构建版本的）Vue 会产生一个控制台的警告。

> <font color=orange>Notice：</font>prop 会在一个组件实例创建==之前==进行验证，所以实例的属性（如`data`、`computed`等）在`default`或`validator`函数中是不可用的。

#### 类型检查

type 可以是这些原生构造函数中的一个：`String`、`Number`、`Boolean`、`Array`、`Object`、`Date`、`Function`、`Symbol`。

type 还可以是一个自定义的构造函数，并通过`instanceof`来进行检查确认。

```javascript
function Person (firstName, lastName) {
  this.firstName = firstName
  this.lastName = lastName
}

Vue.component('blog-post', {
  props: {
    author: Person // 验证author的prop是否验证通过new Person创建的
  }
})
```

### 2.6 非 Prop 的特性

一个非 prop 特性是指传向一个组件，但该组件并没有定义相应 prop 的特性。

假设你通过一个 Bootstrap 插件使用了一个第三方的`<bootstrap-date-input>`组件，这个插件需要在其`<input>`上用到一个`data-date-picker`特性，可以将这个特性添加到组件的实例上：

```html
<bootstrap-date-input data-date-picker="activated" ></bootstrap-date-input>
```

然后这个 `data-date-picker="activated"` 特性就会自动添加到 `<bootstrap-date-input>` 的根元素上。

#### 2.6.1 替换 / 合并已有特性

假设`<bootstrap-date-input>`的模板是这样的：

```html
<input type="date" class="form-control">
```

组件实例调用时，添加了一个类名：

```html
<bootstrap-date-input
  data-date-picker="activated"
  class="date-picker-theme-dark"
></bootstrap-date-input>
```

这样出现了2个class：

- `form-control`，这是在组件的模板内设置好的；
- `date-picker-theme-dark`，这是从组件的父级传入的。

对于大多数特性来说，外部提供给组件的值会替换掉组件内部设置好的值。所以如果传入`type="text"`就会替换掉`type="date"`，但`class`和`style`特性两边的值会被合并，从而得到最终的值。

#### 2.6.2 禁用特性继承

通过在组件的选项中设置`inheritAttrs: false`，来阻止组件根元素的继承特性。

```javascript
Vue.component('my-component', {
  inheritAttrs: false,
  // ...
})
```

实例的`$attrs`属性包含了传递给一个组件的特性名和特性值，如：

```javascript
{
  required: true,
  placeholder: 'Enter your username'
}
```

结合 `inheritAttrs: false` 和 `$attrs`，就可以手动决定这些特性会被赋予哪个元素。

```javascript
Vue.component('base-Vue.component('base-input', {
  inheritAttrs: false,
  props: ['label', 'value'],
  template: `
    <label>
      {{ label }}
      <input
        v-bind="$attrs"
        v-bind:value="value"
        v-on:input="$emit('input', $event.target.value)"
      >
    </label>
  `
})
```

> `inheritAttrs: false`选项不会影响`style`和`class`的绑定。

这个模式允许在使用基础组件的时候更像是使用原始的 HTML 元素，而不会担心那个元素是真正的根元素。

```html
<base-input
  v-model="username"
  required
  placeholder="Enter your username"
></base-input>
```

---

## 3. 自定义事件

### 3.1 事件名

事件名不存在任何自动化的大小写转换，触发的事件名需要匹配监听这个事件所用的名称。

事件名不会被用作一个 JS 变量名或属性名。并且`v-on`事件监听器在 DOM 模板中会被自动转换为全小写（因为 HTML 是大小写不敏感的），因此使用驼峰式命名法，会造成 HTML 中的大写变成小写，而导致 DOM 中的事件名与 JS 中的事件名不匹配，导致无法被监听到，因此不要使用驼峰式命名事件，推荐使用短横分隔式。

### 3.2 自定义组件的 v-model

一个组件上的`v-model`默认会利用名为`value`的 prop 和名为`input`的事件，但像单选框、复选框等类型的输入控件可能会将`value`特性用于不同目的。`model`选项可以用于避免这样的冲突。

```javascript
Vue.component('base-checkbox', {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: { checked: Boolean },
  template: `
    <input
      type="checkbox"
      v-bind:checked="checked"
      v-on:change="$emit('change', $event.target.checked)"
    >
  `
})
```

```html
<!-- 现在在这个组件上使用v-model -->
<base-checkbox v-model="lovingVue"></base-checkbox>
<!-- lovingVue的值会传入名为checked的prop，当<base-checkbox>触发一个change事件并附带一个新的值时，这个lovingVue的属性会被更新 --> 
```

> 仍然需要在组件的`props`选项里声明`checked`这个prop。

### 3.3 将原生事件绑定到组件

通过使用`v-on`的`.native`修饰符，在一个组件的根元素上直接监听一个原生事件。

```html
<base-input v-on:focus.native="onFocus"></base-input>
```

但这个方法在监听一个类似`<input>`的非常特定的元素时，并不一定成功。比如`<base-input>`组件可能做了重构，根元素实际上是一个`<label>`：

```html
<label>
  {{ label }}
  <input
    v-bind="$attrs"
    v-bind:value="value"
    v-on:input="$emit('input', $event.target.value)"
  >
</label>
```

这时父级的`.native`监听器将失效，它不会产生任何报错，但`onFocus`处理函数不会如期调用。

Vue 提供了一个`$listeners`属性来解决这个问题，它是一个对象，里面包含了作用在这个组件上的所有监听器。

```javascript
{
  focus: function (event) { /* ... */ },
  input: function (value) { /* ... */ }
}
```

有了这个`$listeners`属性，就可以配合`v-on="listeners"`将所有的事件监听器指向这个组件的某个特定的子元素。

```javascript
Vue.component('base-input', {
  inheritAttrs: false,
  props: ['label', 'value'],
  computed: {
    inputListeners: function () {
      var vm = this
      // Object.assign()将所有的对象合并为一个新对象
      return Object.assign({},
        this.$listeners, // 从父级添加所有的监听器
        // 添加自定义监听器, 或覆写一些监听器的行为
        { // 确保组件配合v-model的工作
          input: function (event) {
            vm.$emit('input', event.target.value)
          }
        }
      )
    }
  },
  template: `
    <label>
      {{ label }}
      <input
        v-bind="$attrs"
        v-bind:value="value"
        v-on="inputListeners">
    </label>
  `
})
```

这样`<base-input>`组件就可以完全像一个普通的`<input>`元素一样使用了。

### 3.4 .sync 修饰符

有时候需要对一个 prop 进行”双向绑定“，但真正的双向绑定不便维护，因为子组件可以修改父组件，且在父组件和子组件都没有明显的改动来源。

建议以`update:myPropName`的模式触发事件来实现双向绑定。

```javascript
Vue.component('sync-demo', {
  props: ['title'],
  data () {
    return {
      newTitle: 'This is new title'
    }
  },
  template: `
    <div>
      <p>{{ title }}</p>
      <button @click="$emit('update:title', newTitle)">Update</button>
    </div>
  `
})
```

然后父组件可以监听那个事件并根据需要更新一个本地的数据属性。

```html
<text-document
  v-bind:title="doc.title"
  v-on:update="doc.title = $event"
></text-document>
```

为方便起见，Vue 为这种模式提供了一个缩写：`.sync`修饰符。

```html
<text-document v-bind:title.sync="doc.title"></text-document>
```

> <font color=orange>Notice：</font>带有`.sync`修饰符的`v-bind`==不能==和表达式一起使用，只能提供想要绑定的属性名。

当用一个对象同时设置多个 prop 时，可以将 `.sync`修饰符和`v-bind`配合使用：

```html
<text-document v-bind.sync="doc"></text-document>
<!-- 这样会把doc对象中的每个属性都作为一个独立的prop传入，然后各自添加用于更新的v-on监听器 -->
```

> <font color=orange>Notice：</font>将`v-bind.sync`用在一个字面量的对象上（如`v-bind.sync="{ title: doc.title }"`）是无法正常工作的，因为在解析一个这样的复杂表达式的时候，有很多边缘情况需要考虑。

---

## 4. 插槽

### 4.1 插槽内容

Vue 将`<slot>`元素作为承载分发内容的出口。

```html
<!-- 组件navigation-link的模板 -->
<a
  v-bind:href="ur"
  class="nav-link"
>
  <slot></slot>
</a>
<!-- 在父组件中这样合成组件 -->
<navigation-link url="/profile">
  Your Profile
</navigation-link>
<!-- 组件渲染的时候，slot元素会被替换成Your Profile -->
```

插槽内可以包含任何模板代码，包括 HTML 甚至其它组件。

如果组件定义模板内没有包含`<slot>`元素，则该组件起始标签和结束标签之间的任何内容都会被抛弃。

插槽可以访问到当前作用域的实例属性，但不能访问这个组件的作用域。

> **<font color=pink>Remember：</font>**父级组件模板里的所有内容都是在父级作用域中编译的；子模板里的所有内容都是在子作用域中编译的。

### 4.2 后备内容

插槽的后备内容，只会在没有提供内容的时候被渲染。

```html
<!-- 组件模板 -->
<button type="submit">
  <!-- 设置后备内容 -->
  <slot>Submit</slot>
</button>
<!-- 在父级组件中时候用这个组件时不提供任何插槽内容 -->
<submit-button></submit-button>
<!-- 这个时候后备内容就会被渲染 -->
```

但是如果在插槽中提供内容，后备内容就会被取代。

### 4.3 具名插槽

有些情况下需要多个插槽，对于这种情况，`slot`元素有一个特殊的特性`name`，这个特性可以用来定义额外的插槽。

```html
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```

一个不带`name`的`<slot>`出口会带有隐含的名字“default”。

在向具名插槽提供内容的时候，可以在一个`<template>`元素上使用`v-slot`指令，并以`v-slot`的参数的形式提供其名称：

```html
<base-layout>
  <template v-slot:header>
    <h1>Here might be a page title</h1>
  </template>
  <p>A paragraph for the main content.</p>
  <p>And another one.</p>
  <template v-slot:footer>
    <p>Here's some contact info</p>
  </template>
</base-layout>
```

这样`<template>`元素中的所有内容都会被传入响应的插槽，任何没有被包裹在带有`v-slot`的`<template>`中的内容都会被视为默认插槽的内容。

> <font color=orange>Notice：</font>`v-slot`只能添加在一个`<template>`上（提供的内容只有默认插槽的情况除外）。

### 4.4 作用于插槽

为了让子组件作用域的数据能在父级的插槽内容可用，可以将子组件的数据作为`<slot>`元素的一个特性绑定上去：

```html
<span>
  <slot v-bind:user="user">
    {{ user.lastName }}
  </slot>
</span>
```

绑定在`<slot>`元素上的特性被称为**插槽 prop**。在父级作用域中，可以给`v-slot`带一个值来定义提供给插槽 prop 的名字：

```html
<current-user>
  <template v-slot:default="slotProps">
    {{ slotProps.user.firstName }}
  </template>
</current-user>
```

#### 4.4.1 独占默认插槽的缩写语法

当被提供的内容只有默认插槽时，组件的标签可以被当做插槽的模板来使用，这样就可以把`v-slot`直接用在组件上：

```html
<current-user v-slot:default="slotProps">
  {{ slotProps.user.firstName }}
</current-user>
```

还可以更简化，不带参数的`v-slot`被假定对应默认插槽：

```html
<current-user v-slot="slotProps">
  {{ slotProps.user.firstName }}
</current-user>
```

> **<font color=orange>Notice：</font>**默认插槽的缩写语法不能和具名插槽混用，因为它会导致作用域不明确。

#### 4.4.2 解构插槽 prop

作用域插槽的内部工作原理是将你的插槽内容包括在一个传入单个参数的函数里：

```javascript
function (slotProps) {
  // 插槽内容
}
```

`v-slot`的值实际上可以是任何能够作为函数定义中的参数的 JavaScript 表达式。所以在支持的环境下（单文件组件或现代浏览器），也可以使用解构来传入具体的插槽 prop。

```html
<current-user v-slot="{ user }">
  {{ user.firstName }}
</current-user>
```

它也开启了 prop  重命名等其他可能：

```html
<current-user v-slot="{ user: person }">
  {{ person.firstName }}
</current-user>
```

也可以定义后备内容，用于插槽 prop 是 undefined 的情况：

```html
<current-user v-slot="{ user = { firstName: 'Guest' } }">
  {{ user.firstName }}
</current-user>
```

### 4.5 动态插槽名

动态指令参数也可以用在`v-slot`上来定义动态的插槽名：

```html
<base-layout>
  <template v-slot:[dynamicSlotName]>
    <!-- ... -->
  </template>
</base-layout>
```

### 4.6 具名插槽的缩写

`v-slot:`的缩写是：`#`。该参数只在其有参数时才可用，而且必须要使用明确的插槽名（动态变量也是可以的，只要其值是明确的）。

---

## 5. 动态组件 & 异步组件

### 5.1 在动态组件上使用`keep-alive`

用于将失活的组件缓存。

比如当在多标签界面切换时，在一个标签进行了一些操作，再切换至一个新的标签，当切换回原来的标签时，原来的标签页刷新回初始状态，而不是展示原来操作过的状态。为了能够将这个更改过的状态缓存下来，再次回到这个标签页的时候能够展示离开时的状态，可以使用`keep-alive`元素将其动态组件包裹起来。

```html
<keep-alive>
  <component v-bind:is="currentTabComponent"></component>
</keep-alive>
```

> <font color=orange>Notice：</font>`<keep-alive>`要求被切换到的组件都有自己的名字，不管是通过组件的`name`选项还是局部 / 全局注册的。

### 5.2 异步组件

Vue 允许以一个工厂函数的方式定义组件，这个工厂函数会异步解析组件的定义。Vue 只有在这个组件需要被渲染的时候才会触发这个工厂函数，而且会把结果缓存起来供之后重渲染。

```javascript
Vue.component('async-example', function (resolve, reject) {
  setTimeout(function () { // 这里的setTimeout为演示用
    // 向resolve回调传递组件定义
    resolve({
      template: '<div>I am async!</div>'
    })
  }, 1000)
})
```

这个工厂函数会收到一个 `resolve` 回调，这个回调函数会在从服务器得到组件定义时被调用。也可以调用 `reject(reason)` 来表示加载失败。

推荐将异步组件和 webpack 的 code-splitting 功能一起配合使用：

```javascript
Vue.component('async-webpack-example', function (resolve) {
  // require会告诉webpack自动将构建代码切割成多个包
  // 这些包会通过Ajax请求加载
  require(['./my-async-component'], resolve)
})
```

也可以在工厂函数中返回一个 Promise，所以把 webpack2 和 ES2015 语法加在一起，可以写成这样：

```javascript
Vue.component(
  'async-webpack-example',
  // 这个import函数会返回一个Promise对象
  () => import('./my-async-component')
)
```

使用局部注册的时候也可以直接提供一个返回 Promise 的函数：

```javascript
new Vue({
  // ...
  components: {
    'my-component': () => import('./my-async-component')
  }
})
```

#### 5.2.1 处理加载状态

这里的异步组件工厂函数也可以返回一个这样格式的对象：

```javascript
const AsyncComponent = () => ({
  // 需要加载的组件（应该是个Promise对象）
  component: import('./MyComponent.vue'),
  // 异步加载时使用的组件
  loading: LoadingComponent,
  // 加载失败时使用的组件
  error: ErrorComponent,
  // 展示加载时组件的延时，默认是200ms
  delay: 200,
  // 若提供了超时时间且组件加载超时了，则使用加载失败时使用的组件，默认为Infinity
  timeout: 3000
})
```

---

## 6.处理边界情况

### 6.1 访问元素 & 组件

多数情况下，最好不要触达另一个组件实例内部或手动操作 DOM 元素。

#### 6.1.1 访问根实例

在每个`new Vue`实例的子组件中，其根实例可以通过`$root`属性进行访问，所有的子组件都可以将这个实例作为一个全局 store 来访问或使用。

```javascript
// Vue根实例
new Vue({
  data: {
    foo: 1
  },
  computed: {
    bar: function () { /* ... */ }  
  },
  methods: {
    baz: function () { /* ... */ }
  }
})

// 子组件
this.$root.foo // 获取根组件的数据
this.$root.foo = 2 // 写入根组件的数据
this.$root.bar // 访问根组件的计算属性
this.$root.baz() // 调用根组件的方法
```

#### 6.1.2 访问父级组件实例

`$parent`属性可以用来从一个子组件访问父组件的实例。通过这个属性，可以在后期随时触达父级组件替代将数据以 prop 形式传入子组件的方式。

但是这种方式构建出来的组件很容易出现问题。

#### 6.1.3 访问子组件实例或子元素

通过`ref`特性为一个子组件赋予一个  ID 引用：

```html
<base-input ref="usernameIput"></base-input>
```

然后在这个组件里通过`$refs`来访问这个组件的实例：

```javascript
this.$refs.usernameInput
```

如，程序化地从一个父级组件聚焦这个输入框。

```html
<input ref="input">
```

然后可以在其组件定义方法：

```javascript
methods: {
  focus: function () {
    this.$refs.input.focus() // 用来从父级组件聚焦输入框
  }
}
```

这样就允许父级组件通过下面的代码聚焦`<base-input>`里的输入框：

```javascript
this.$refs.usernameInput.focus()
```

#### 6.1.4 依赖注入

`provide`选项允许指定要提供给后代组件的数据 / 方法。

```java
provide: function () {
  return {
    getMap: this.getMap
  }
}
```

然后在任何后代组件里都可以使用`inject`选项来接收，而不需要暴露这个组件实例：

```javascript
inject: ['getMap']
```

可以把依赖注入看做一部分“大范围有效的 prop“，除了：

- 祖先组件不需要知道哪些后代组件使用它提供的属性；
- 后代组件不需要知道被注入的属性来自哪里。

> **<font color=orange>依赖注入的负面影响：</font>**它将应用程序中的组件与它们当前的组织方式耦合起来，使重构变得更加困难。

### 6.2 程序化的事件侦听器

除了`$emit`，Vue 实例在其事件借口中提供了其他方法：

- 通过`$on(eventName, eventHandler)`侦听一个事件；
- 通过`$once(eventName, eventHandler)`一次性侦听一个事件；
- 通过`$off(eventName, eventHandler)`停止侦听一个 事件。

可以用于在一个组件实例上手动侦听事件，也可用于代码组织工具。

> **<font color=red>Notice：</font>**Vue 的事件系统不同于浏览器的 EventTarget API，`$emit`、`$on`、`$off`并不是`dispatchEvent`、`addEventListener`、`removeEventListener`的别名。

### 6.3 循环引用

**递归组件**

组件可以在它自己的模板中调用自身，但只能通过`name`选项来做这件事。

当使用`Vue.component`全局注册一个组件时，这个全局的 ID 会自动设置为该组件的`name`选项。

稍有不慎，递归组件就可能导致无限循环：

```javascript
name: 'stack-overflow',
template: '<div><stack-overflow></stack-overflow></div>'
```

**组件之间的循环引用**

指 A 组件的子模板引用了 B 组件，而 B 组件的子模板又引用了 A 组件。

```html
<!-- tree-folder组件 -->
<p>
  <tree-contents></tree-contents>
</p>
<!-- tree-contents组件 -->
<div>
  <tree-folder></tree-folder>
</div>
```

这样导致两个组件在渲染树中互为对方的后代和祖先，这样形成了一个 Paradox，当通过`Vue.component`全局注册组件的时候，这个 Paradox 会被自动解开。但如果使用一个模板系统依赖 / 导入组件，就会遇到一个错误。

为了解决这个问题，需要给模板系统一个点，在这个点“A 反正是需要 B 的，但是不需要先解析 B”。

可以等执行生命周期钩子`beforeCreate`时去注册那个不需要先解析的 B 组件。

```javascript
// 把tree-folder组件设为这个点，那么产生Paradox的子组件是tree-contents
beforeCreate: function () {
  this.$options.components.TreeFilderContents = 
    require('./tree-contents.vue').default
}
```

或者在本地注册的时候，使用 webpack 的异步`import`：

```javascript
components: {
  TreeFolderContents: () => import('./tree-contents.vue')
}
```

### 6.4 模板定义的替代品

#### 6.4.1 内联模板

当`inline-template`这个特性出现在一个子组件上，这个组件将会使用里面的内容作为模板而不是将其作为被分发的内容。

```html
<my-component inline-template>
  <div>
    <p>These are compiled as the component's own template.</p>
    <p>Not parent's transclusion content.</p>
  </div>
</my-component>
```

内联模板需要定义在 Vue 所属的 DOM 元素内。

> <font color=orange>Notice：</font>`inline-template`会让模板的作用域变得更加难以理解。所以建议在组件内有限选择`template`选项或`.vue`文件里的一个`<template>`元素来定义模板。

#### 6.4.2 X-Template

另一个定义模板的方式是在一个`<script>`元素中，并为其带上`text/x-template`的类型，然后通过一个 id 将模板引用过去。

```html
<script type="text/x-template" id="hello-world-template">
	<p>Hello !</p>
</script>
```

```javascript
Vue.component('hello-world', {
  template: '#hello-world-template'
})
```

`x-template`需要定义在 Vue 所属的 DOM 元素外。

> <font color=orange>Notice：</font>这些可以用于模板特别大的 demo 或极小型的应用，但是其它情况下要避免使用，因为这会将模板和该组件的其它定义分离开。

### 6.5 控制更新

#### 6.5.1 强制更新

> 如果你发现你自己需要在 Vue 中做一次强制更新，99.9% 的情况，是你在某个地方做错了。

可以通过`$forceUpdate`]来做这件事。

#### 6.5.2 通过`v-once`创建低开销的静态组件

渲染普通的 HTML 元素在 Vue 中是非常快速的，但有的时候你可能有一个组件，这个组件包含了大量静态内容。在这种情况下，你可以在根元素上添加 `v-once` 特性以确保这些内容只计算一次然后缓存起来。

```javascript
Vue.component('terms-of-service', {
  template: `
    <div v-once>
      <h1>Terms of Service</h1>
      ... a lot of static content ...
    </div>
	`
})
```

> 不要过度使用这个模式。当你需要渲染大量静态内容时，极少数的情况下它会给你带来便利，除非你非常留意渲染变慢了，不然它完全是没有必要的——再加上它在后期会带来很多困惑。













































