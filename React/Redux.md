# Redux

## 1. 概述

**什么是 Redux ？**

Redux 是负责组织 state 的工具。

**什么场景下适合使用 Redux ？**

- 有大量的、随时间变化的数据；
- state 需要有一个单一可靠的数据来源；
- 把所有 state 放在最顶层组件中已经无法满足需要了。

**安装**

```shell
# 安装稳定版
npm install --save redux
# 安装 React 绑定库
npm install --save react-redux
# 安装开发者工具
npm install --save-dev redux-devtools
```

```javascript
import { createStore } from 'redux'
/**
 * 这是一个 reducer，形式为 (state, action) => state 的纯函数。
 * 描述了 action 如何把 state 转变成下一个 state。
 *
 * state 的形式取决于你，可以是基本类型、数组、对象、
 * 甚至是 Immutable.js 生成的数据结构。惟一的要点是
 * 当 state 变化时需要返回全新的对象，而不是修改传入的参数。
 *
 * 下面使用 switch 语句和字符串来做判断，但你可以写帮助类(helper)
 * 根据不同的约定（如方法映射）来判断，只要适用你的项目即可。
 */
function counter(state = 0, action) {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1
    case 'DECREMENT':
      return state - 1
    default:
      return state
  }
}

// 创建 Redux store 来存放应用的状态。
// API 是 { subscribe, dispatch, getState }。
let store = createStore(counter)

// 可以手动订阅更新，也可以事件绑定到视图层。
store.subscribe(() => console.log(store.getState()))

// 改变内部 state 惟一方法是 dispatch 一个 action。
// action 可以被序列化，用日记记录和储存下来，后期还可以以回放的方式执行
store.dispatch({ type: 'INCREMENT' })
// 1
store.dispatch({ type: 'INCREMENT' })
// 2
store.dispatch({ type: 'DECREMENT' })
// 1
```

### 1.1 要点

`state`：一个组件的数据状态。

`action`：一个用来描述发生了什么的普通 JS 对象。

`reducer`：一个接收 state 和 action，并返回新的 state 的函数。

所有的 state 都以一个对象树的形式存储在一个单一的 store 中，其他的代码不能随意修改它。

唯一能改变 state 中数据的方法就是发起一个 action。

为了把 action 和 state 串起来，开发了一些`reducer`函数。

### 1.2 三大原则

随着应用不断变大，你应该把根级的 reducer 拆成多个小的 reducers，分别独立地操作 state 树的不同部分，而不是添加新的 stores。这就像一个 React 应用只有一个根级的组件，这个根组件又由很多小组件构成。

1. **单一数据源：**

   整个应用的 state 被存储在一棵对象树中，且这个对象树只存在于唯一一个 store 中。

2. **State 是只读的：**

   唯一改变 state 的方法就是触发 action。这样确保视图和网络请求都不能直接修改 state。

3. **使用纯函数来执行修改：**

   为了描述如何改变 state Tree，需要编写 reducer 函数。随着应用变大，可以把它拆成多个小的 reducers，分别独立地操作 state Tree 的不同部分。

---

## 2. 基础

### 2.1 Action

**Action** 是把数据从应用（这些数据有可能是服务器响应，用户输入或其它非 view 的数据 ）传到 store 的有效载荷。它是 store 数据的**唯一**来源。

添加新 todo 任务的 action 是这样的：

```javascript
{
  type: 'ADD_TODO',
  text: 'Build my first Redux app'
}
```

Action 本质上是 JS 普通对象。除了`type`字段必须是一个字符串外，action 对象的结构完全由你自己决定。多数情况下，`type` 会被定义成字符串常量。当应用规模越来越大时，最好用单独的模块或文件来存放 action。

我们还需要再添加一个 action index 来表示用户完成任务的动作序列号。因为数据是存放在数组中的，所以我们通过下标 `index` 来引用特定的任务。而实际项目中一般会在新建数据的时候生成唯一的 ID 作为数据的引用标识。

 **尽量减少在 action 中传递数据。**

#### Action 创建函数

**Action 创建函数** —— 生成 action 的方法。

```javascript
function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  }
}
// 把 action 创建函数的结果传给 dispatch() 即可发起一次 dispatch 过程
dispatch(addTodo(text));
// 创建一个被绑定的 action 创建函数可以自动 dispatch
const boundAddTo = text => dispatch(addTodo(text));
boundAddTo(text); // 然后直接调用它们
```

store 里能直接通过`store.dispatch()`调用`dispatch()`。

Action 创建函数也可以是异步非春函数。

### 2.2 Reducer

**Reducer** 指定了应用状态的变化如何响应 actions 并发送到 store。

#### 设计 State 结构

在写之前，先想一下这个对象的结构，如何才能以最简的形式描述出应用的 state 对象。

尽量把 UI 相关的 state 与其他数据分开。

```javascript
{
  visibilityFilter: 'SHOW_ALL', // UI 相关的 state
  todos: [
    {
      text: 'Consider using Redux',
      completed: true,
    }, {
      text: 'Keep all state in a single tree',
      completed: false
    }
  ]
}
```

> **处理 Reducer 关系时的注意事项：**
>
> 开发复杂的应用时，不可避免会有一些数据相互引用。尽可能地把 state 范式化，不存在嵌套。把所有数据放到一个对象里，每个数据以 ID 为主键，不同实体或列表间通过 ID 相互引用数据。把应用的 state 想像成数据库。

#### Action 处理

Reducer 一定要保持纯净。不要在 Reducer 中做这些操作：

- 修改传入参数；
- 执行有副作用的操作（如 API 请求和路由跳转）；
- 调用非纯函数（如`date.now()`或`Math.random()`）。

Redux 首次执行时，state 为`undefined`，可以借机设置并返回应用的初始 state。

```javascript
import { SET_VISIBILITY_FILTER, VisibilityFilters } from './actions';
const initialState = {
  visibilityFilter: VisibilityFilters.SHOW_ALL,
  todos: []
}
function todoApp(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return Object.assign({}, state, {
        visibilityFilter: action.filter
      })
    default: return state; // 暂不处理任何 action，仅返回传入的 state
  }
}
```

> **不要直接修改 state：**
>
> 使用`Object.assign()`时，不能这样使用`Object.assign(state, actions...)`，因为这个方法会改变第一个参数的值，所以必须将第一个参数设置为空对象。
>
> **在 default 情况下返回旧的 state：**
>
> 遇到未知的 action 时，一定要返回旧的 state。

有多个 action 需要处理时，通过扩展 reducer 的 case 去处理。当需要修改数组中指定的数据项而又不希望发生*突变*（指直接修改引用所指向的值而引用本身保持不变），最好的做法是创建一个新的数据后，将那些无需修改的项原封不动移入，接着对需修改的项用新生成的对象替换：

```javascript
todos: [ // 创建新数组, 新的 todos 相当于旧的 todos 在末尾加上新内容
  ...state.todos,
  {
    text: action.text,
    completed: false
  }
]
```

#### 拆分 Reducer

有时候，可以把数据更新的逻辑拆分到一个单独的函数里：

```js
const { SHOW_ALL } = VisibilityFilters;
function todos(state = [], action) { // state 现在变成了一个数组
  switch (action.type) {
    case ADD_TODO:
      return [
        ...state,
        {
          text: action.text,
          completed: false
        }
      ];
    case TOGGLE_TODO:
      return state.map((todo, index) => {
        if (index === action.index) {
          return Object.assign({}, todo, {
            completed: !todo.completed
          });
        }
        return todo;
      });
    default: return state;
  }
}
function visibilityFilter(state = SHOW_ALL, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return action.filter;
    default: return state;
  }
}
function todoApp(state = {}, action) {
  return {
    visibilityFilter: visibilityFilter(state.visibilityFilter, action),
    todos: todos(state.todos, action)
  };
}
```

> 每个 Reducer 只负责管理全局 state 中它负责的一部分，每个 Reducer 的 state 参数都不同，分别对应它管理的那部分 state 数据。

#### conbineReducers()

Redux 提供了`combineReducers()`工具类来做上面`todoApp`做的事情。

```javascript
import { combineReducers } from 'redux';
const todoApp = combineReducers({
  visibilityFilter,
  todos
});
export default todoApp;
```

也可以给它们设置不同的 key，或者调用不同的函数。

```javascript
const reducer = combineReducers({
  a: doSomethingWithA,
  b: processB,
  c: c
});
// 等价于
function reducer(state = {}, action) {
  return {
    a: doSomethingWithA(state.a, action),
    b: processB(state.b, action),
    c: c(state.c, action)
  };
}
```

`combineReducers()` 生成一个调用 reducer 的函数。每个 reducer 根据它们的 key 来筛选出 state 中的一部分数据并处理，然后这个生成的函数再将所有 reducer 的结果合并成一个大的对象。

## 2.3 Store

Redux 应用只有一个单一 store。

Store 的职责：

- 维持应用的 state；
- 提供`getState()`方法获取 state；
- 提供`dispatch(action)`方法更新 state；
- 通过`subscribe(listener)`注册监听器；
- 通过`subscribe(listener)`返回的函数注销监听器。

```javascript
import {createStore} from 'redux';
// 创建Store
let store = createStore(todoApp);
// 每次更新state打印日志
const unsubscribe = store.subscribe(() => console.log(store.getState()));
// 更新state
store.dispatch(addTodo('Learn about actions'));
// 通过执行subscribe返回的函数停止监听state更新
unsubscribe();
```

## 2.4 数据流

Redux 架构设计的核心是**严格的单向数据流**。

Redux 应用中数据的生命周期遵循下面 4 个步骤：

1. 调用`store.dispatch(action)`。
2. Redux store 调用传入的 reducer 函数。
3. 把多个子 reducer 输出合并成一个单一的树。
4. Redux store 保存了根 reducer 返回的完整的 state 树。























