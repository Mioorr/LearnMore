# Components

# 1. 组件间通信

组件间的基本通信方式：

| 组件间的基本通信方式   | 用途                                                         |
| ---------------------- | ------------------------------------------------------------ |
| WXML 数据绑定          | 父组件向子组件的指定属性设置数据，仅能兼容 JSON 数据         |
| 事件                   | 用于子组件向父组件传递数据，可以传递任意数据                 |
| `this.selectComponent` | 如果以上两种方式都不足以满足需要，父组件可以通过这个方法获取子组件实例对象，这样就可以直接访问组件的任意数据和方法。 |

## 1.1 WXML 数据绑定



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

## 1.2 事件通信

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







