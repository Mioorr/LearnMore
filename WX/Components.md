# Components

# 1. Component 构造器

`Component`构造器可用于定义组件，调用`Component`构造器时可以指定组件的属性、数据、方法等。

[Component 属性值](https://developers.weixin.qq.com/miniprogram/dev/reference/api/Component.html)。

## 1.1 使用构造器构造页面

小程序的页面也可以视为自定义组件，因此页面也可以用`Component`构造器构造，拥有与普通组件一样的定义段与实例方法，但此时要求对应 json 文件中包含 `usingComponents` 定义段。

```json
{
  "usingComponents": {}
}
```

```js
Compoennt({
  properties: {
    paramA: Number,
    paramB: String,
  },
  methods: {
    onLoad: function() {
      this.data.paramA // 页面参数 paramA 的值
      this.data.paramB // 页面参数 paramB 的值
    }
  }
})
```

使用`Component`构造器来构造页面的一个好处是可以使用`behaviors`来提取页面中公用的代码段。 

如：在所有页面被创建和销毁时都要执行同一段代码，就可以把这段代码提取到`behaviors`中。

```js
// page-common-behavior.js
module.exports = Behavior({
  attatched: function() {
    // 页面创建时执行
  },
  detached: function() {
    // 页面销毁时执行
  }
})
```

```js
/* 页面A */
var pageCommonBehavior = require('./page-common-behavior')
Component({
  behaviors: [pageCommonBehavior],
  data: { /* ... */ },
  methods: { /* ... */ }
})
/* 页面B */
var pageCommonBehavior = require('./page-common-behavior')
Component({
  behaviors: [pageCommonBehavior],
  data: { /* ... */ },
  methods: { /* ... */ }
})
```

---

# 2. 组件生命周期

## 2.1 组件的生命周期

| 生命周期    | 参数           | 执行时间                           |
| ----------- | -------------- | ---------------------------------- |
| `created`   | 无             | 组件实例刚被创建时                 |
| `attatched` | 无             | 组件实例进入页面节点树时           |
| `ready`     | 无             | 组件在视图层布局完成后             |
| `moved`     | 无             | 组件实例被移动到节点树另一个位置时 |
| `detached`  | 无             | 组件实例被从页面节点树移除时       |
| `error`     | `Object Error` | 每当组件方法抛出错误时             |



## 2.2 组件所在页面的生命周期



---

# 3. properties & data

## 3.1 properties 定义

| 定义段        | 类型 \| 是否必填 | 描述                       |
| ------------- | ---------------- | -------------------------- |
| type          | _ _ \| Yes       | 属性的类型                 |
| optionalTypes | Array \| No      | 属性的类型（可以指定多个） |
| value         | _ _ \| No        | 属性的初始值               |
| observer      | Function \| No   | 属性值变化时的回调函数     |

属性值的改变情况可以使用 observer 来监听，在新版本基础库中不推荐使用这个字段，而是使用 Component 构造器的 `observers` 字段代替，它更加强大且性能更好。

```js
Component({
  properties: {
    min: {
      // 设置这个属性可以是Number String Object三种类型中的一种
      type: Number,
      optionalTypes: [String, Object],
      value: 0,
      observer: function(newVal, oldVal) {
        //属性值变化时执行
      }
    }
  }
})
```

属性的类型可以为 String、Number、Boolean、Object、Array 其一，也可以为 `null` 表示不限制类型。

> 属性名要避免以 data 开头，因为在WXML中，`data-xzy=""`会被作为节点 dataset 来处理，而不是组件属性。

## 3.2 data

使用`this.data`可以获取内部数据和属性值，但直接修改它不会将变更应用到界面上，要用`setData`修改。

在一个组件的定义和使用时，组件的属性名和 data 字段相互间都不能冲突。

对象类型的属性和 data 字段中可以包含函数类型的子字段，即可以通过对象类型的属性字段来传递函数（基础库`2.0.9+`）。

---

# 4. 数据监听器

数据监听器可以用于监听和响应`properties`和`data`的变化。

```js
Component({
  properties: { /* 组件的对外属性 */ },
  data: { /* 组件的模板渲染 */ },
  observers: { /* 组件数据字段监听器 */ },
})
```

当一些数据字段被`setData`设置时，需要执行一些操作时使用。

- 一次`setData`最多触发每个监听器一次；
- 可以同时监听多个；
- 监听器可以监听子数据字段。

```js
Component({
  observers: {
    'numberA, numberB': function(numberA, numberB) {
      // 在 numberA 或者 numberB 被设置时，执行
      this.setData({
        sum: numberA + numberB
      })
    },
    'some.subfield': function(subfield) {
  		// 使用setData设置this.data.some.subfield时触发
  		// 使用setData设置this.data.some时也会触发
  		subfield === this.data.some.subfield
		},
    'arr[12]': function(arr12) {
      // 使用setData设置this.data.arr[12]时触发
      // 使用setData设置this.data.arr时也会触发
      arr12 === this.data.arr[12]
    }
  }
})
```

如果需要监听所有自数据字段的变化，可以使用通配符（`**`）。

```js
Component({
  observers: {
    'some.field.**': function(field) {
      // 使用setData设置this.data.some.field或其下任何子数据字段时触发
      // 使用setData设置this.data.some也会触发）
      field === this.data.field
    }
  }
})
```

仅使用通配符`**`可以监听全部`setData`。

```js
Component({
  observers: {
    '**': function() {
      // 每次setData都触发
    }
  }
})
```

> <font color="red">☆</font>
>
> - 数据监听器监听的是`setData`涉及到的数据字段，即使这些数据字段的值没有发生变化，数据监听器依然会被触发。
> - 如果在数据监听器函数中使用`setData`设置本身监听的数据字段，可能会导致死循环。
> - 数据监听器和属性的`observer`相比，数据监听器更强大，且通常具有更好的性能。

---

# 5. 组件间通信

组件间的基本通信方式：

| 组件间的基本通信方式   | 用途                                                         |
| ---------------------- | ------------------------------------------------------------ |
| WXML 数据绑定          | 父组件向子组件的指定属性设置数据，仅能兼容 JSON 数据         |
| 事件                   | 用于子组件向父组件传递数据，可以传递任意数据                 |
| `this.selectComponent` | 如果以上两种方式都不足以满足需要，父组件可以通过这个方法获取子组件实例对象，这样就可以直接访问组件的任意数据和方法。 |

## 5.1 WXML 数据绑定

```jsx
<!-- 引用组件的页面 -->
<view>
  <component-tag-name prop-a="{{dataA}}" prop-b="{{dataB}}">
    <view>插入到组件slot的内容</view>
  </component-tag-name>
</view>
```

组件的属性`prop-a`和`prop-b`将会收到页面传递的数据。页面可以通过`setData`来改变绑定的数据字段。

> <font color="red">☆ </font>这样的数据绑定只能传递 JSON 兼容数据。自基础库版本 2.0.9 开始，还可以在数据中包含函数（但这些函数不能在 WXML 中直接调用，只能传递给子组件）。

## 5.2 事件通信

**监听事件**

自定义组件可以触发任意事件，引用组件的页面可以监听这些事件。

```jsx
<!-- 当自定义组件触发"myevent"事件时调用"onMyEvent"方法 -->
<component-tag-name bindmyevent="onMyEvent" />
<!-- 也可以写成: -->
<component-tag-name bind:myevent="onMyEvent" />
```

```js
Page({
  onMyEvent: function(e) {
    e.detail // 自定义组件触发时间时提供的detail对象
  }
})
```

**触发事件**

自定义组件触发事件时需要使用`trigger`方法，指定事件名、`detail`对象和事件选项。

```jsx
<!-- 自定义组件中 -->
<button bind:tap="onTap">点击这个按钮将触发"myevent"事件</button>
```

```js
Component({
  properties: {},
  methods: {
    onTap: function() {
      var myEventDetail = {}
      var myEventOption = {}
      this.triggerEvent('myevent', myEventDetail, myEventOption)
    }
  }
})
```

触发事件的选项（event option）有（都是布尔值，非必填字段，默认值都为`false`）：

| 选项名       | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| bubbles      | 事件是否冒泡                                                 |
| composed     | 事件是否可以穿越组件边界？<br />`false` => 事件只能在引用组件的节点树上触发，不进入其他组件内部。 |
| capturePhase | 事件是否拥有捕获阶段                                         |

---

# 6. behaviors

每个`behavior`可以包含一组属性、数据、生命周期函数和方法。**组件引用它时，它的属性、数据和方法会被合并到组件中，生命周期函数也会在对应时机被调用**。每个组件可以引用多个 `behavior` ，`behavior` 也可以引用其他 `behavior`。

## 6.1 组件使用

组件引用时，将它们逐个列出即可。

```js
var myBehavior = require('my-behavior')
Component({
  behaviors: [myBehavior], // 引入'my-behavior'
  properties: {
    myProperty: {
      type: String
    }
  },
  data: {
    myData: {}
  },
  attached: function() {},
  method: {
    myMethod: function() {}
  }
})
```

## 6.2 字段的覆盖和组合规则

组件和它引用的`behavior`中可以包含同名的字段，对这些字段的处理方法为：

- 如果有同名的属性或方法，组件本身的属性或方法会覆盖`behavior`中的属性或方法，如果引用了多个`behavior`，在定义段中靠后`behavior`中的属性或方法会覆盖靠前的属性或方法。
- 如果有同名的数据字段，如果数据是对象类型，会进行对象合并，如果是非对象类型，会进行相互覆盖。
- 生命周期函数不会相互覆盖，而是在对应触发时机被逐个调用。如果同一个`behavior`被一个组件多次引用，它定义的生命周期函数只会被执行一次。

## 6.3 内置 behaviors

自定义组件可以通过引用内置的`behavior`来获得内置组件的一些行为。

```js
Component({
  // 'wx://form-field'代表一个内置behavior
  behaviors: ['wx://form-field']
})
```

内置`behavior`往往会为组件添加一些属性，没有特殊说明时，组件可以覆盖这些属性来改变其`type`或添加`observer`。

`wx://component-export`使自定义组件支持`export`定义段，它可以用于指定组件被`selectComponent`调用时的返回值。未使用这个定义段时， `selectComponent` 将返回自定义组件的 `this` （插件的自定义组件将返回 `null` ）。使用这个定义段时，将以这个定义段的函数返回值代替。

```jsx
<!-- 使用自定义组件 -->
<my-component id="the-id" />
```

```js
// 自定义组件my-component内部
Component({
  behaviors: ['wx://component-export'],
  export() {
    return { myField: 'myValue' }
  }
})
// 调用
this.selectComponent('#the-id') // => { myField: 'myValue' }
```

# 7. 组件间关系

## 7.1 定义

组件间有相互的关系，可在组件定义时加入`relations`定义段。

```jsx
<custom-ul>
  <custom-li>item 1</custom-li>
  <custom-li>item 2</custom-li>
</custom-ul>
```

```js
Component({
  relations: {
    './custom-li': {
      type: 'child', // 关联的目标接地单应为子节点
      // target是该节点的实例对象，触发在该节点attatch生命周期之后
      linked: function(target) {
        // 每次有有custim-li被插入时执行，target
      }
    }
  }
})
```



