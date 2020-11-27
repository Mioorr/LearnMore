# Computed & Watch

## 计算属性computed

Vue 的**计算属性**是指定义在 Vue 实例`computed`选项中的属性，用于处理一些复杂逻辑的变量。

模板内可以嵌入表达式，但模板中放入过多逻辑会让模板过重且难以维护。

```html
<template>
  <div>
    <p>Oraginal message: "{{ message }}"</p>
    <p>Oraginal message: "{{ message.split('').reverse().join('') }}"</p>
  </div>
</template>

<script>
export default {
  data: {
    message: 'Hello'
  }
}
</script>
```

面对这样复杂逻辑的时候，应当使用计算属性：

```vue
<template>
	<div>
    <p>Oraginal message: "{{ message }}"</p>
    <p>Computed reversed message: "{{ reversedMessage }}"</p>
	</div>
</template>

<script>
export default {
  data () {
    return {
    	message: 'Hello' 
    }
  },
  computed: {
    reversedMessage () {
      return this.message.split('').reverse().join('')
    }
  }
}
</script>
```







计算属性与方法的区别：
**计算属性是基于他们的响应式依赖进行缓存的**，只在相关响应式  依赖发生改变时，它们才会重新求值。
而方法是，每当调用方法，就会再次执行函数。

如果频繁的使用计算属性，而计算属性中有大量的耗时操作，会带来一些性能问题。



## 侦听器watch

Vue 的**侦听器**是定义在 Vue 实例`watch`中的属性，用于在数据变化时执行异步或开销较大的操作。

```html
<template>
  <div id="watch-example">
    <p>
      Ask a question (yes/no):
      <input v-model="question">
    </p>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  data () {
    return {
      question: '',
      answer: 'I can not give you an answer untill you ask a question.'
    }
  },
  watch: {
    question (newQst, oldQst) {
      this.answer = 'You are typing...';
      this.debu
    }
  }
}
</script>
```



