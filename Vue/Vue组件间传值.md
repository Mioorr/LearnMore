# Vue 组件间传值

由于组件实例的作用域是孤立的，所以组件之间的传值成为了一个门槛，需要通过特定通道才能实现组件之间的传值。

## 父组件→子组件

### 1. 通过`props`向子组件传递数据

通过`props`向子组件传递数据有以下几个步骤：

1. 父组件在引用子组件时，在标签中通过自定义属性名添加属性向子组件下发数据

   ```html
   <Children :attrname="value"></Children>
   ```

2. 然后子组件显式地通过`props`选项声明它的预期数据，从而获取父组件传递的数据

   ```javascript
   props: ['attrname']
   ```

看下面这个例子：

```html
<!-- 子组件 Header.vue -->
<template>
	<header class="header">
    <a class="logo" href="/">{{ logo }}</a>
    <ul class="nav">
      <li v-for="nav in navs">{{ nav.item }}</li>
    </ul>
  </header>
</template>
<script>
export default {
  name: 'Header',
  porps: ['logo'], // 从父组件获取logo值
  data () {
    return {
      navs: [
        {item: 'Home'},
        {item: 'Logs'},
        {item: 'Photos'}
      ]
    }
  }
}
</script>

<!-- 父组件 App.vue -->
<template>
	<div class="app">
    <!-- 通过logo属性传值 -->
    <Header :logo="logoMsg" />
  </div>
</template>
<script>
import Header from './components/Header'
export default {
  components: {
		Header
  }
  data () {
    return {
      logoMsg: 'Site Logo'
    }
  }
}
</script>
```



## 子组件 → 父组件

### 1. 通过事件传递数据

通过事件向父组件传递数据前，我们需要先了解一个Vue的API：

> `vm.$emit(eventName, [...args])`：触发当前实例上的事件。附加参数都会传给监听器回调。
>
> 参数：
>
> - `{string} eventName`要触发的事件名
> - `[...args]` 要传递给监听器回调的附加参数

```html
<!-- 子组件 -->
<template>
	<div>
    <label>
    	<span>User Name:</span>
      <!-- 输入框值改变触发setUser方法 -->
      <input type="text" v-model="username" @change="setUser" />
    </label>
  </div>
</template>
<script>
export default {
  name: 'Login',
  data () {
    return {
      username: ''
    }
  },
  methods: {
    setUser () {
      // 触发实例上的事件，并传递数据
      this.$emit('transferUser', this.username);
    }
  }
}
</script>

<!-- 父组件 -->
<template>
	<div id="app">
    <!-- 给子组件绑定自定义事件 -->
    <Login @transferUser="getUser" />
    <p>User name is: {{ user }}</p>
  </div>
</template>
<script>
import Login from './components/Login';
export default {
  data () {
    return {
      user: ''
    }
  },
  methods: {
    getUser (msg) {
      this.user = 'msg';
    }
  }
}
</script>
```



## 子组件 → 子组件

