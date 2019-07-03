# 过渡 & 动画

## 1. 进入 / 离开 & 列表过渡

Vue 在插入、更新或者移除 DOM 时，提供多种不同方式的应用过渡效果。
包括以下工具：

- 在 CSS 过渡和动画中自动应用 class；
- 可以配合使用第三方 CSS 动画库，如 Animate.css；
- 在过渡钩子函数中使用 JavaScript 直接操作 DOM；
- 可以配合使用第三方 JavaScript 动画库，如 Velocity.js。

### 1.1 单元素 / 组件的过渡

Vue 提供了 `transition` 的封装组件，在下列情形中，可以给任何元素和组件添加进入/离开过渡

- 条件渲染 (使用 `v-if`)
- 条件展示 (使用 `v-show`)
- 动态组件
- 组件根节点

```html
<button @click="show = !show">Toggle</button>
<transition name="fade">
	<p v-if="show">When "show = true", I will show!</p>
</transition>
<style>
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}
</style>
```

```javascript
data: {
  show: true
}
```

当插入或删除包含在 `transition` 组件中的元素时，Vue 将会做以下处理：

1. 自动嗅探目标元素是否应用了 CSS 过渡或动画，如果是，在恰当的时机添加 / 删除 CSS 类名。

2. 如果过渡组件提供了 JavaScript 钩子函数，这些钩子函数将在恰当的时机被调用。

3. 如果没有找到 JavaScript 钩子并且也没有检测到 CSS 过渡/动画，DOM 操作 (插入/删除) 在下一帧中立即执行。

   > <font color=orange>Notice：</font>这里指浏览器逐帧动画机制，和 Vue 的 `nextTick` 概念不同。

#### 1.1.1 过渡的类名

在进入 / 离开的过渡中，有6个 class 切换：

1. `v-enter`：定义进入过渡的开始状态。在元素被插入之前生效，元素被插入之后的下一帧移除。
2. `v-enter-active`：定义进入过渡生效时的状态。在整个进入过渡的阶段中应用，元素被插入之前生效，在过渡/动画完成之后移除。
3. `v-enter-to`：定义进入过渡的结束状态。在元素被插入之后下一帧生效 (与此同时 `v-enter` 被移除)，过渡/动画完成之后移除。
4. `v-leave`：定义离开过渡的开始状态。在离开过渡被触发时立刻生效，下一帧被移除。
5. `v-leave-active`：定义离开过渡生效时的状态。在整个离开过渡的阶段中应用，在离开过渡被触发时立刻生效，在过渡/动画完成之后移除。
6. `v-leave-to`：定义离开过渡的结束状态。在离开过渡被触发之后下一帧生效 (与此同时 `v-leave` 被删除)，在过渡/动画完成之后移除。

![过渡类名示意图](./img/v-enter-leave.png)

对于这些在过渡中切换的类名来说，如果你使用一个没有名字的 `<transition>`，则 `v-` 是这些类名的默认前缀（如使用了 `<transition name="my-transition">`，那么 `v-enter` 会替换为 `my-transition-enter`）。





















