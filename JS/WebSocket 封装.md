# WebSocket 封装

> 以下会使到 ES6 的语法，若是需要项目兼容较低版本浏览器，需要将其进行编译。

```js
// socket.js
export default class WebSocketClass {
  constructor (url) {
    this.connect(url);
  }

  connect (url) {
    this.ws = new WebSocket(_wsDomain + url + _token);
  }

  getMessage (callback) {
    this.ws.onmessage = (e) => {
      const data = JSON.parse(e.data).results;
      callback(data);
    }
  }

  close () {
    this.ws.close();
  }
}
```



