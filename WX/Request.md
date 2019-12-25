# 请求

`wx.request()`

# 相关说明

## 1. 服务器域名配置

每个微信小程序需要实现设置通讯域名，<font color="#03a9f4">小程序只可以与指定的域名进行网络通信</font>。包括 HTTPS 请求(`wx.request`)、上传文件(`wx.upload`)、下载文件(`wx.downloadFile`)和 WebSocket 通信(`wx.connectSocket`)。

(基础库`2.4.0+`) 网络接口允许与局域网 IP 通信，但<font color="#03a9f4">不允许与本机IP通信</font>。

**配置流程：**

在「小程序后台 - 开发 - 开发设置 - 服务器域名」中配置，配置时需要注意：

- 域名只支持`http`（`wx.request`、`wx.uploadFile`、`wx.downloadFile`）和`wxss`（`wx.connectSocket`）协议；
- 域名不能使用 IP 地址（小程序的局域网 IP 除外）或 localhost；
- 可以配置端口，但配置后，只能向配置的端口发请求，否则请求会失败；
- 如果不配置端口，那么请求的 URL 中也不能包含端口号。
- 域名必须经过 ICP 备案；
- `api.weixin.qq.com`不能被配置为服务器域名，相关 API 也不能在小程序内调用。
- 对于每个接口，分别可以配置最多 20 个域名。

## 2. 网络请求

**超时时间：**

- 默认超时时间和最大超时时间都是 60s；
- 超时时间可以在`app.json`或`game.json`中通过`networktimeout`配置。

**使用限制：**

- 网络请求的`referer` header 不可设置，格式固定为

  `https://servicewechat.com/{小程序id}/{小程序版本号}/page-frame.html`，版本号=0表示开发、体验、审核版，版本号=devtools表示开发者工具，其余值为正式版。









# 1. wx.request(Object object)

用法与AJAX类似。

**data 参数说明**

最终发送给服务器的数据是 String 类型，如果传入的 data 不是 String 类型，会被转换成 String。转换规则：

- 对于`GET`方法的数据，会将数据转换成 query string（`encodeURIComponent(k)=encodeURIComponent(v)&encodeURIComponent(k)=encodeURIComponent(v)...`）；
- 对于`POST`方法且`header['content-type']`

```js
var requestTask = wx.request({
  url: 'https://test.xxx.com',
  data: {
    x: '',
    y: ''
  },
  header: {
    'content-type': 'application/json'
  },
  success (res) {
  	console.log(res.data)
	}
})
requestTask.abort() // 取消请求任务
```

**RequestTask方法**

- `RequestTask.abort()`：中断请求任务；
- `RequestTask.onHeadersReceived(function callback)`：监听 HTTP Response Header 事件，比请求完成事件更早。
- `RequestTask.offHeadersReceived(function callback)`：取消监听 HTTP Response Header 事件。

# 2. wx.uploadFile(Object object)

客户端发起一个 HTTPS POST 请求，`content-type = multipart/form-data`。

| 属性     | 类型     | 必填 | 说明                                                         |
| -------- | -------- | ---- | ------------------------------------------------------------ |
| url      | String   | Y    | 开发者服务器地址                                             |
| filePath | String   | Y    | 要上传文件资源的路径（网络路径）                             |
| name     | String   | Y    | 文件对应的 key，开发者在服务端可以通过这个 key 获取文件的二进制内容 |
| header   | Object   | N    | HTTP 请求 Header，Header 中不能设置 Referer                  |
| formData | Object   | N    | HTTP 请求中其他额外的 form data                              |
| success  | Function | N    | 接口调用成功的回调                                           |
| fail     | Function | N    | 接口调用失败的回调                                           |
| complete | Function | N    | 接口调用结束的回调（成功&失败都执行）                        |



```js
wx.chooseImage({
  success (res) {
    const tempFilePaths = res.tempFilePaths
    wx.uploadFile({
      url: 'https://text.xxx.com',
      filePath: tempFilePaths[0],
      name: 'file',
      formData: {
        'user': 'test'
      },
      success (res) {
        const data = res.data
      }
    })
  }
})
```





