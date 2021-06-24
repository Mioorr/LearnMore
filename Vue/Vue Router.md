# Vue Router

## Vue Router 的导航守卫

### 1. 全局前置守卫 `router.beforeEach()`

当一个导航触发时，全局前置守卫按照创建顺序调用。守卫是一步解析执行，此时导航在所有守卫 resolve 完之前一直处于等待中。

```javascript
router.beforeEach((to, from, next) => {
  // 返回false取消导航
  return false;
  // 确保next在任何给定的导航守卫中都被严格调用一次，否则钩子永远不会被解析或报错
  next();
})
```

- `to`：即将要进入的目标路由
- `from`：当前导航正要离开的路由
- `next`：resolve 导航守卫

### 2. 全局解析守卫 `router.beforeResolve()`

这和全局前置守卫类似，因为它在每次导航时都会触发，但是确保在导航被确认之前，同时在所有组件内守卫和异步路由组件被解析之后，解析守卫就被正确调用。

```javascript
router.beforeResolve(async to => {
  if (to.meta.requiresCamera) {
    try {
      await askForCameraPermission()
    } catch (error) {
      if (error instanceof NotAllowedError) {
        // ...处理错误，然后取消导航
        return false;
      } else {
        // 意料之外的错误，取消导航并把错误传给全局处理器
        throw error;
      }
    }
  }
})
```

`router.beforeResolve`是一个理想位置，可以在用户无法进入页面的情况下，获取数据或进行任何其他需要避免的操作。

### 3. 全局后置钩子

全局后置钩子和守卫不同的是：这些钩子不会接受`next`函数，也不会改变导航本身。

```javascript
router.afterEach((to, from, failure) => {
  if (!failure) sendAnalytics(to, fullPath);
});
```

另外将 navigation failures 作为第三个参数。

它们对于分析、更改页面标题、声明页面等辅助功能以及许多其他事情都很有用。

### 4. 路由独享的守卫

直接在路由配置上定义`beforeEnter`守卫，它**只在进入路由时触发**，不会在`params`、`query`或`hash`改变时触发。

```javascript
const routes = [
  {
    path: '/user/:id',
    component: UserDetails,
    beforeEnter: (to, from) => {
      // Reject the navigation
  		return false;
  	}
  }
]
```

也可以将一个函数数组传递给`beforeEnter`，这在为不同的路由重用守卫时很有用：

```javascript
function removeQueryParams(to) {
  if (Object.keys(to.query).length)
    return { path: to.path, query: {}, hash: to.hash }
}
function removeHash(to) {
  if (to.hash) return { path: to.path, query: to.query, hash: '' }
}

const routes = [
  {
    path: '/users/:id',
    component: UserDetails,
    beforeEnter: [removeQueryParams, removeHash],
  },
  {
    path: '/about',
    component: UserDetails,
    beforeEnter: [removeQueryParams],
  },
]
```

### 5. 组件内的守卫

可以在路由组件内直接定义路由导航守卫。

#### 可用的配置API

可以为路由组件添加以下配置：

- `beforeRouteEnter`
- `beforeRouteUpdate`
- `beforeRouteLeave`

```javascript
const UserDetails = {
  template: `...`,
  beforeRouteEnter(to, from) {
    // 在渲染该组件的对应路由被验证前调用，不能获取组件实例 this
    // 因为当守卫执行时，组件实例还没被创建！
  },
  beforeRouteUpdate(to, from) {
    // 在当前路由改变但该组件被复用时调用
    // 如：对于一个带有动态参数的路径/users/:id，在/users/1和/users/2之间跳转的时，由于会渲染同样的组件，因此组件实例会被复用。
    // 而这个钩子就会在这个情况下被调用。
    // 因为在这种情况发生时，组件已挂载好，导航守卫可以访问组件实例this
  },
  beforeRouteLeave(to, from) {
    // 在导航离开渲染该组件的对应路由时调用
    // 与beforeRouteUpdate一样，它可以访问组件实例this
  },
}
```

#### 使用组合API

如果正在使用组合API和`setup`函数来编写组件，可以通过`onBeforeRouteUpdate`和`onBeforeRouteLeave`分别添加 update 和 leave 守卫。

## 完整的导航解析流程

1. 导航被触发。
2. 在失活的组件里调用`beforeRouteLeave`守卫。
3. 调用全局的`beforeEach`守卫。
4. 在重用的组件里调用`beforeRouteUpdate`守卫。
5. 在路由配置里调用`beforeEnter`。
6. 解析异步路由组件。
7. 在被激活的组件里调用`beforeRouteEnter`。
8. 调用全局的`beforeResolve`守卫。
9. 导航被确认。
10. 调用全局的`afterEach`钩子。
11. 触发DOM更新。
12. 调用`beforeRouteEnter`守卫中传给`next`的回调函数，创建好的组件实例会作为回调函数的参数传入。

