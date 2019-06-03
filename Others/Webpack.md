# 1. 概念

webpack 是一个现代 JS 应用程序的静态模块打包器。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。

1. **loader**：让 webpack 能够去处理非 JS 文件（webpack 自身只理解 JS）。loader 可以将所有类型的文件转换为 webpack 能处理的有效模块，然后就可以利用 webpack 的打包能力对它们进行处理。
2. **plugins**：插件可以用于执行范围更广的任务。插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量。
3. **mode**：通过选择`development`和`production`中的一个设置`mode`参数启用响应模式下的 webpack 内置的优化。

## 1.1 entry points

entry 指示 webpack 应该使用哪个模块作为构建其内部依赖图的开始。在 webpack 配置中有多种方式定义 `entry` 属性。

### 1.1.1 单个 entry (简写)

用法：`entry: string|Array<string>`

```javascript
// webpack.config.js
const config = {
  entry: {
    main: './path/to/my/entry/file.js'
  }
};
// webpack.config.js 简写
const config = {
  entry: './path/to/my/entry/file.js'
};
module.exports = config;
```

### 1.1.2 对象语法

用法：`entry: {[entryChunkName: string]: string|Array<string>}` 这是应用程序中定义入口最可扩展的方式。

```javascript
const config = {
  entry: {
    app: './src/app.js',
    vendors: './src/vendors.js'
  }
}
```

多页应用中（每当页面跳转时），服务器会获取一个新的 HTML 文档，页面重新加载新文档，并且资源被重新下载。

这样就可以使用`commonsChunkPlugin`为每个页面间的应用程序共享代码创建 bundle。由于入口起点增多，多页应用能够复用入口起点之间的大量代码 / 模块。

## 1.2 output

output 控制 webpack 在哪里向硬盘写入编译文件。

> <font color=orange>Notice：</font>entry point 可以存在多个 ，但只能指定一个 output 配置。

基本上，整个应用程序结构都会被编译到指定的输出路径的文件夹中。

### 1.2.1 用法

在 webpack 中配置`output`属性的最低要求是，将它的值设置为一个对象，包括：

- `filename`用于输出文件的文件名；
- 目标输出目录`path`的绝对路径。

```javascript
const config = {
  output: {
    // 将一个单独的文件输出到Path目录中
    filename: 'bundle.js',
    path: '/home/proj/public/assets'
  }
};
module.exports = config;
```

**多个 entry point 情况**

如果配置创建了多个单独的“chunk”，则应该使用占位符来确保每个文件具有唯一的名称。

```javascript
{
  entry: {
    app: './src/app.js',
    search: './src/search.js'
  },
  output: {
    filename: '[name].js',
    path: __dirname + '/dist'
  }
}
// 写入到硬盘：./dist/app.js、./dist/search.js
```

**高级进阶**

使用 CDN 和 hash 的复杂示例：

在编译时不知道最终输出文件的`publicPath`情况下，`publicPath`可以留空，且在入口起点文件运行时动态设置。

```javascript
output: {
  path: '/home/proj/cdn/assets/[hash]',
  publicPath: 'http://cdn.example.com/assets/[hash]/'
}
```

也可以先忽略它，在入口起点设置`__webpack_public_path__`：

```javascript
__webpack_public_path__ = myRuntimePublicPath
// 剩余的应用程序入口
```

## 1.3 mode

























