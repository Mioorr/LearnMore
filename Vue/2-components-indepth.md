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
<blog-post post-title="hello" />
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
<blog-post title="My journey with Vue" />
```

**传递动态 prop**

通过`v-bind`动态赋值。

```html
<!-- 动态赋予一个变量的值 -->
<blog-post v-bind:title="post.title" />
<!-- 动态赋予一个复杂表达式的值 -->
<blog-post
  v-bind:title="post.title + ' by ' + post.author.name"
/>
```

#### 2.3.1 传入一个数字

```html
<!-- 虽然42是静态的，仍然需要v-bind来告诉Vue这是一个 JS 表达式而不是一个字符串 -->
<blog-post v-bind:likes="42" />
<!-- 用一个变量进行动态赋值 -->
<blog-post v-bind:likes="post.likes" />
```

#### 2.3.2 传入一个布尔值

```html
<!-- 包含 prop 没有值的情况在内，都意味着true -->
<blog-post is-published />
<!-- false是静态的，仍然需要v-bind告诉Vue这是一个JS表达式 -->
<blog-post v-bind:is-published="false" />
<!-- 用一个变量进行动态赋值 -->
<blog-post v-bind:is-published="post.isPublished" />
```

#### 2.3.3 传入一个数组

```html
<!-- 数组是静态的，仍需要v-bind告诉Vue这是一个JS表达式 -->
<blog-post v-bind:comment-ids="[234, 266, 273]" />
<!-- 用一个变量进行动态赋值 -->
<blog-post v-bind:comment-ids="post.commentIds" />
```

#### 2.3.4 传入一个对象

```html
<blog-post
  v-bind:author="{
    name: 'Vernoica',
    company: 'Veridian Dynamics'
  }"
/>
<!-- 用一个变量进行动态赋值 -->
<blog-post v-bind:author="post.author" />
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
<blog-post v-bind="post" />
<!-- 等价于 -->
<blog-post v-bind:id="post.id" v-bind:title="post.title" />
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
<bootstrap-date-input data-date-picker="activated" />
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
/>
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
/>
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
<base-checkbox v-model="lovingVue" />
<!-- lovingVue的值会传入名为checked的prop，当<base-checkbox>触发一个change事件并附带一个新的值时，这个lovingVue的属性会被更新 --> 
```

> 仍然需要在组件的`props`选项里声明`checked`这个prop。

### 3.3 将原生事件绑定到组件

通过使用`v-on`的`.native`修饰符，在一个组件的根元素上直接监听一个原生事件。

```html
<base-input v-on:focus.native="onFocus" />
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
this.$emit('update:title', newTitle)
```

然后父组件可以监听那个事件并根据需要更新一个本地的数据属性。

```html
<text-document
  v-bind:title="doc.title"
  v-on:update="doc.title = $event"
/>
```

为方便起见，Vue 为这种模式提供了一个缩写：`.sync`修饰符。

```html
<text-document v-bind:title.sync="doc.title" />
```















