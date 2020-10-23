# 浏览器存储

浏览器的存储方式：`cookie`、`localStorage`、`sessionStorage`。

## cookie

### 什么是 cookie ？

`Cookie` 是存储于计算机上的小文件，可以由 Web 服务器和客户端浏览器访问。
在每一次 Web 请求的时候都会携带在请求头上。大小限制4k。

cookie 可分为：
**会话Cookie**：不需要指定过期时间（`Expires`）或者有效期（Max-Age），浏览器关闭之后它会被自动删除，也就是说它仅在会话期内有效。

> 需要注意的是，有些浏览器提供了会话恢复功能，这种情况下即使关闭了浏览器，会话期Cookie也会被保留下来，就好像浏览器从来没有关闭一样。

**持久性Cookie**：持久性 Cookie 可以指定一个特定的过期时间（`Expires`）或有效期（`Max-Age`）。

### 语法

```javascript
// 设置 cookie
document.cookie = "username=John Doe; expires=Thu, 18 Dec 2013 12:00:00 GMT; path=/";
// 删除 cookie
document.cookie = "username=; expires=Thu, 01 Jan 1970 00:00:00 GMT"; 
```

**expires**——过期时间
过了到期时间，浏览器会自动删除改 cookie ，如果想删除一个 cookie，只需要把它的过期时间设置成过去的时间即可。
如果不设置过期时间，则表示这个 cookie 的生命周期为浏览器会话期间，只要关闭浏览器窗口，cookie 就消失了。

**path**——路径值
可以是一个目录或者是一个路径。

**domain**——主机名
是指同一个域下的不同主机（如：`www.baidu.com`和`map.baidu.com`就是两个不同的主机名）。
默认情况下，一个主机中创建的cookie在另一个主机下是不能被访问的，但可以通过domain参数来实现对其的控制：

````javascript
// 设置所有*.baidu.com的主机都可以访问该cookie
document.cookie = "name=value;domain=.baidu.com";
````

#### cookie 的 Secure 和 HttpOnly 标记

标记为 `Secure` 的 cookie 只能通过被 Https 协议加密过的请求发送给服务端。但即便设置了 `Secure`标记，敏感信息也不应该通过Cookie传输，因为Cookie有其固有的不安全性，`Secure `标记也无法提供确实的安全保障（从 Chrome 52 和 Firefox 52 开始，`http:`站点无法使用 Cookie 的 `Secure`标记。

为避免跨域脚本 (XSS) 攻击，通过JavaScript的 `Document.cookie` API无法访问带有 `HttpOnly` 标记的Cookie，它们只应该发送给服务端。如果包含服务端 Session 信息的 Cookie 不想被客户端 JavaScript 脚本调用，就应该为其设置 `HttpOnly` 标记。

#### cookie 的作用域

`Domain` 和 `Path` 标识定义了 Cookie 的作用域。

`Domain` 标识指定了哪些主机可以接受 Cookie（不指定则默认为当前文档的主机，不包含子域名）。如果指定了`Domain`，一般包含子域名。

> 如：设置 Domain=mozilla.org，则 Cookie 也包含在子域名中（如 developer.mozilla.org）。

`Path` 标识指定了主机下的哪些路径可以接受 Cookie（该URL路径必须存在于请求URL中）。以字符 `%x2F` (`/`) 作为路径分隔符，子路径也会被匹配。

> 如：设置 path=/docs，则 ‘/docs’、‘/docs/Web’、‘/docs/Web/HTTP’ 都会被匹配。

#### SameSite Cookies

`SameSite` Cookie 允许服务器要求某个cookie 在跨站请求时不会被发送，从而可以阻止跨站请求伪造（CSRF）攻击。

### cookie 的优缺点

缺点：由于 Web 请求时都会携带在请求头上，导致流量增大。



## sessionStorage

### 什么是 sessionStorage ？

`sessionStorage` 是 `window` 对象的一个属性，用于存储页面中的数据。

`sessionStorage`属性允许你访问一个对应当前源的`sessionStorage`对象。

### 语法

```javascript
// 保存数据到 sessionStorage
sessionStorage.setItem('key', 'value');
// 从 sessionStorage 获取数据
let data = sessionStorage.getItem('key');
// 从 sessionStorage 删除保存的数据
sessionStorage.removeItem('key');
// 从 sessionStorage 删除所有保存的数据
sessionStorage.clear();
```

### sessionStorage 的特点

- 页面会话在浏览器打开期间一直保持，且重新加载或恢复页面仍会保持原来的页面会话。
- 在新标签或窗口打开一个页面时会复制顶级浏览回话的上下文作为新回话的上下文。
- 打开多个相同的 URL 的 Tabs 页面，会创建各自的 sessionStorage。
- 关闭对应浏览器 tab，会清除对应的sessionStorage。



## 三者区别

|              | cookie | sessionStorage | localStorage |
| ------------ | ------ | -------------- | ------------ |
| 最大存储大小   | 4k     | 一般为5M        | 一般为5M |
| 数据有效期 | 设置了过期时间的，在该时间之前一直有效；<br />未设置过期时间的，当前浏览器关闭前有效 | 仅在当前浏览器窗口关闭前有效 | 始终有效 |
| 作用域 | z | 在不同浏览器页面中不能共享，即使是同一个页面 | 在所有同源窗口中共享 |

