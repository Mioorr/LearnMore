# 1. 概念

webpack 是一个现代 JS 应用程序的静态模块打包器。当 webpack 处理应用程序时，它会构建一个依赖图，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。

1. **plugins**：插件可以用于执行范围更广的任务。插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量。

2. **mode**：通过选择`development`和`production`中的一个设置`mode`参数启用响应模式下的 webpack 内置的优化。

## 1.1 入口

`entry` 指示 webpack 应该使用哪个模块作为构建其内部依赖图的开始（默认值是 `.src/index.js`）。

在 webpack 配置中有多种方式定义 `entry` 属性。

1. 单个 `entry`语法
   用法：`entry: string|Array<string>`
   **缺点**： 该方法在扩展配置时缺乏灵活性。

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

2. 对象语法
   用法： `entry: {<entryChunkName> string | [string]}`
   **缺点**： 比较繁琐。
   **优点**： 这是应用程序中定义入口的最可扩展方式。

   ```javascript
   // webpack.config.js
   module.exports = {
     entry: {
       app: './src/app.js',
       adminApp: './src/adminApp.js'
     }
   };
   ```

## 1.2 output

output 控制 webpack 在哪里向硬盘写入编译文件。

> <font color=orange>Notice：</font>`entry` 可以存在多个 ，`output` 配置只能指定一个。

在 webpack 中配置`output`属性的最低要求是，将它的值设置为一个对象，包括：

- `filename`：用于输出文件的文件名（默认为`main.js`）；
- `path`：目标输出目录（默认为`./dist`）。

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

**多个入口起点的情况**

如果配置创建了多个“chunk”，则应该使用占位符来确保每个文件具有唯一的名称。

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

在编译时不知道最终输出文件的`publicPath`情况下，`publicPath`可以留空，且在入口起点文件运行时通过`__webpack_public_path__`动态设置。

```javascript
// config.js
module.exports = {
  // ...
  output: {
    path: '/home/proj/cdn/assets/[hash]',
    publicPath: 'http://cdn.example.com/assets/[hash]/'
  }
};
```

也可以先忽略它，在入口起点设置`__webpack_public_path__`：

```javascript
__webpack_public_path__ = myRuntimePublicPath
// 应用程序入口的其余部分
```

## 1.3 loader

loader 的作用：

- 让 webpack 能够处理其他类型的文件（webpack 自身只理解 JS 和 JSON）。
- 用于对模块的源代码进行转换。
- 可以使你在`import` 或加载模块时预处理文件。
- 可以将不同的语言转换为 JS 或将内联图像转换为 data URL。
- 允许你直接在 JS 模块中 `import` CSS文件。

有3种使用 loader 的方式：

1. 配置方式（推荐）：在 webpack.config.js 文件中指定 loader。
   `module.rules` 允许在 webpack 配置中指定多个 loader。

   loader 从下到上地执行。

   ```javascript
   module.exports = {
     module: {
       rules: [
         {
           test: /\.css$/, /* 正则 */
           use: [
             {
               loader: 'style-loader'
             }, {
               loader: 'css-loader',
               options: {
                 modules: ture
               }
             }, {
               loader: 'sass-loader'
             }
           ]
         }
       ]
     }
   };
   // 上面loader的执行顺序是 sass-loader => css-loader => style-loader
   ```

2. 内联方式：在`import`语句或与“import”方法等同的引用方式中指定loader。
   使用`!`将资源中的loader分开，每个部分都会相对于当前目录解析。

   ```javascript
   import Style from 'style-loader!css-loader?moudles!./style.css'
   ```

   使用`!`前缀，将禁用所有已配置的 normal loader。

   ```javascript
   import Styles from '!style-loader!css-loader?modules!./style.css'
   ```

   使用 `!!` 前缀，将禁用所有已配置的 loader。

   ```javascript
   import Styles from '!!style-loader!css-loader?modules!./styles.css';
   ```

   使用 `-!`前缀，将禁用所有已配置的loader。

   ```javascript
   import Styles from '-!style-loader!css-loader?modules!./styles.css';
   ```

3. CLI 方式：在`shell`命令中指定它们。

   ```javascript
   webpack --module-bind pug-loader --module-bind 'css=style-loader:css-loader'
   ```

   这会对`.jade`文件使用`pug-loader`，以及对`.css`文件使用`style-loader`和`css-loader`。

**loader特性**

- loader 支持链式调用。
  一组链式的 loader 将按照相反的顺序执行。
- loader 可以是同步的，也可以是异步的。
- loader 运行在 Node.js 中，且能够执行任何操作。
- loader 可以通过 `options` 对象配置（仍支持使用 `query` 参数来设置选项，但该方式已废弃）。
- 可以通过 `package.json` 的 `main` 来将一个 npm 模块导出为 loader，还可以在 module.rules 中使用 `loader` 字段直接引用一个模块。
- plugin 可以为 loader 带来更多特性。
- loader 能够产生额外的任意文件。

## 1.4 plugin

webpack 插件是一个具有`apply`方法的 JS 对象。`apply`方法会被webpack compiler 调用，并在整个编译生命周期都可以访问 compiler 对象。

```javascript
// ConsoleLogOnBuildWebpackPlugin.js
const plugin = 'ConsoleLogOnBuildWebpackPlugin';
class ConsoleLogOnBuildWebpackPlugin {
  apply(compiler) {
    compiler.hooks.sun.tap(pluginName, compilation) => {
      console.log('webpack 构建过程开始！');
    }
  }
}
module.exports = ConsoleLogOnBuildWebpackPlugin;
```

由于 plugin 可以携带参数，必须在 webpack 配置中，向`plugins`属性传入一个`new`实例。

对应有多种使用插件的方式：

1. 配置方式。

   





























