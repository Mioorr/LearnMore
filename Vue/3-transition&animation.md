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

对于这些在过渡中切换的类名来说，如果使用一个没有名字的 `<transition>`，则 `v-` 是这些类名的默认前缀，若使用了`name`（如 `<transition name="my-transition">`），那么`v-enter`会替换为`name值-enter`（如`my-transition-enter`）。

#### 1.1.2 CSS 过渡

通常的过渡是使用 CSS 过渡。

```css
.fade-enter-active { transition: all .5s ease; }
.fade-leave-active {
  transition: all .5s cubic-bezier(1.0, 0.5, 0.8, 1.0);
}
.fade-enter, .fade-leave-to {
  transform: translateX(10px);
  opacity: 0;
}
```

#### 1.1.3 CSS 动画

CSS 动画用法同 CSS 过渡，区别是在动画中`v-enter`类名在节点插入 DOM 后不会立即删除，而是在`animationed`事件触发时删除。

```css
.bounce-enter-active { animation: bounce-in 1s; }
.bounce-leave-active { animation: bounce-in 1s reverse; }
@keyframes bounce-in {
  0% { transform: scale(0); }
  50% { transform: scale(1.1); }
  100% { transform: scale(1); }
}
```

#### 1.1.4 自定义过渡的类名

可以通过以下特性来自定义过渡类名：

- `enter-class`
- `enter-active-class`
- `enter-to-class`
- `leave-class`
- `leave-active-class`
- `leave-to-class`

他们的优先级高于普通的类名，用于结合 Vue 的过渡系统和其他第三方 CSS 动画库。

```html
<link href="https://cdn.jsdelivr.net/npm/animate.css@3.5.1" rel="stylesheet" type="text/css">

<div id="example-3">
  <button @click="show = !show">
    Toggle render
  </button>
  <transition
    name="custom-classes-transition"
    enter-active-class="animated tada"
    leave-active-class="animated bounceOutRight"
  >
    <p v-if="show">hello</p>
  </transition>
</div>
```

#### 1.1.5 同时使用过渡和动画

Vue 为了知道过渡的完成，必须设置相应的事件监听器。它可以是 `transitionend` 或 `animationend` ，这取决于给元素应用的 CSS 规则。如果用其中一种，Vue 能自动识别类型并设置监听。但在一些场景中，需要给同一个元素同时设置两种过渡动效，`animation`很快的被触发并完成了，而 `transition`效果还没结束。在这种情况下，就需要使用 `type` 特性并设置 `animation` 或 `transition` 来明确声明要 Vue 监听的类型。

#### 1.1.6 显性的过渡持续时间

在很多情况下，Vue 可以自动得出过渡效果的完成时机。默认情况下，Vue 会等待其在过渡效果的根元素的第一个`transitionend`或`animationend`事件。

也可以不这样设定（如，自定义一系列过渡效果，其中一些嵌套的内部元素相比于过渡效果的根元素有延迟的或更长的过渡效果），可以用 `<transition>` 组件上的 `duration` 属性定制一个显性的过渡持续时间 (以毫秒计)：

```html
<transition :duration="1000">...</transition>
```

也可以定制进入和移出的持续时间：

```html
<transition :duration="{ enter: 500, leave: 800 }">...</transition>
```

#### 1.1.7 JS 钩子

可以在属性中声明 JS 钩子。

```html
<transition
  v-on:before-enter="beforeEnter"
  v-on:enter="enter"
  v-on:after-enter="afterEnter"
  v-on:enter-cancelled="enterCancelled"

  v-on:before-leave="beforeLeave"
  v-on:leave="leave"
  v-on:after-leave="afterLeave"
  v-on:leave-cancelled="leaveCancelled"
>
  <!-- ... -->
</transition>
```

```javascript
methods: {
  /* -Enter- */
  beforeEnter: function (el) { /* ... */ },
  enter: function (el, done) {
    // 与 CSS 结合使用时, 回调函数 done 是可选的
    done()
  },
  afterEnter: function (el) { /* ... */ },
  enterCancelled: function (el) { /* ... */ },
	/* -Leave- */
  beforeLeave: function (el) { /* ... */ },
  leave: function (el, done) { /* ... */
    done()
  },
  afterLeave: function (el) { /* ... */ },
  // leaveCancelled 只用于 v-show 中
  leaveCancelled: function (el) { /* ... */ }
}
```

这些钩子函数可以结合 CSS `transitions/animations` 使用，也可以单独使用。

> <font color=orange>Notice：</font>
>
> 当只用 JS 过渡时，在`enter`和`leave`中必须使用`done`进行回调。否则，它们将被同步调用，过渡会立即完成。
>
> 推荐对于仅使用 JavaScript 过渡的元素添加 `v-bind:css="false"`，Vue 会跳过 CSS 的检测。这也可以避免过渡过程中 CSS 的影响。

### 1.2 初始渲染的过渡

可以通过`appear`特性设置在节点在初始渲染的过渡。

```html
<transition appear></transition>
```

这里默认和进入/离开过渡一样，同样也可以自定义 CSS 类名。

```html
<transition
  appear
  appear-class="custom-appear-class"
  appear-to-class="custom-appear-to-class" (2.1.8+)
  appear-active-class="custom-appear-active-class"
>
  <!-- ... -->
</transition>
```

自定义 JS 钩子：

```html
<transition
  appear
  v-on:before-appear="customBeforeAppearHook"
  v-on:appear="customAppearHook"
  v-on:after-appear="customAfterAppearHook"
  v-on:appear-cancelled="customAppearCancelledHook"
></transition>
```

### 1.3 多个元素的过渡

最常见的多标签过渡是一个列表和描述这个列表为空消息的元素：

```html
<transition>
  <table v-if="items.length > 0">
  </table>
  <p v-else>Sorry, no items found.</p>
</transition>
```

> <font color=orange>Notice：</font>当有相同标签名的元素切换时，需要通过`key`特性设置唯一的值来标记以让 Vue 区分它们，否则 Vue 为了效率只会替换相同标签内部的内容。

一些场景中，可以通过给同一个元素的`key`特性设置不同的状态代替`v-if`和`v-else`。

```html
<transition>
  <button v-bind:key="isEditing">
    {{ isEditing ? 'Save' : 'Edit' }}
  </button>
</transition>
```

`transition`的同时生效的进入和离开的过渡不能满足所有要求，所以 Vue 提供了 **过渡模式**

- `in-out`：新元素先进行过渡，完成之后当前元素过渡离开。
- `out-in`：当前元素先进行过渡，完成之后新元素过渡进入。

```html
<transition name="fade" mode="out-in">
  <!-- ... the buttons ... -->
</transition>
```

### 1.4 多个组件的过渡

多个组件的过渡不需要使用`key`特性，需要使**动态组件**：

```html
<transition name="component-fade" mode="out-in">
  <component v-bind:is="view"></component>
</transition>
```
```javascript
data: {
  view: 'v-a'
},
components: {
  'v-a': {
    template: '<div>Component A</div>'
  },
  'v-b': {
    template: '<div>Component B</div>'
  }
}
```
### 1.5 列表过渡

使用`<transitionpgroup>`组件可以渲染整个列表，它的特点：

- 不同于 `<transition>`，它会以一个真实元素呈现：默认为一个 `<span>`，可以通过 `tag`特性更换为其他元素。
- 过渡模式不可用（`mode`特性），因为不相互切换特有的元素。
- 内部元素需要提供唯一的 `key` 属性值。















































