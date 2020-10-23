# Promise

所谓`Promise`，简单说，是一个容器，里面保存着某个未来才会结束的事件（通常是异步操作）的结果；从语法上说，Promise 是一个对象，通过它可以获取异步操作的消息。

**Promise 对象有以下两个特点：**

1. 对象的状态不受外界影响。
   Promise 对象代表一个异步操作，有 3 种状态：
   ① `pending`（进行中）
   ② `fulfilled`（已成功）
   ③ `rejected`（已失败）
   只有异步操作的结果可以决定当前是哪种状态，任何其他操作都无法改变这个状态。
2. 一旦状态改变，就不会再变，任何时候都可以得到这个结果。
   `Promise`对象状态改变只有 2 种可能：
   ① `pending` → `fulfilled`
   ② `pending` → `rejected`
   只要这两种情况发生，状态就会凝固不变，这时就称 **resolved**（已定型）。如果改变已经发生，再对`Promise`对象添加回调函数也会立即得到这个结果。

---

## 1. 基本用法

`Promise`对象是一个构造函数，用来生成`Promise`实例。

`Promise`构造函数接受一个函数作为参数，该函数的两个参数分别是`resolve`和`reject`。它们是两个函数，由 JavaScript 引擎提供，不用自己部署。

```javascript
const promise = new Promise(function(resolve, reject) {
  // ...some code
  if (/* 异步操作成功 */) {
    resolve(value);
  } else {
    reject(error);
  }
});
```

`resolve`函数的作用是，将`Promise`对象的状态从 pending 变为 resolved，在异步操作成功时调用，并将异步操作的结果作为参数传递出去；
`reject`函数的作用是，将`Promise`对象的状态从 pending 变为 rejected，在异步操作失败时调用，并将异步操作的报错作为参数传递出去。

`Promise`实例生成后，可以用`then()`分别指定`resolved`状态和`rejected`状态的回调函数。

```javascript
promise.then(function(value) {
  // success
}, function(error) {
  // failure
});
```

`then()`方法可以接受两个回调函数作为参数。第一个回调函数是`Promise`对象的状态变为`resolved`时调用，第二个（可选参数）回调函数是`Promise`对象的状态变为`rejected`时调用。

**Promise 新建后就会立即执行**，所以有些时候将`Promise`对象放在函数中返回，调用函数的时候才会执行。

**调用`resolve`或`reject`并不会终结 Promise 的参数函数的执行**，但是一般来说，调用`resolve`或`reject`以后，Promise的使命就完成了。

