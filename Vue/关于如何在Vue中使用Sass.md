# 1. 在 Vue 中使用 Sass

> 下面示例的环境是用`vue-cli`搭建，使用 npm 进行包管理，因此目录结构会与其他方式搭建的项目目录结构有所区别。

1. **安装相关依赖**
   Vue 的 webpack 项目中需要安装上 `node-sass`、`sass-loader`、`style-loader`，所以需要先通过命令行安装。

   ```powershell
   # 一次性安装全部
   npm i node-sass sass-loader style-loader -D
   ```

2. **在 webpack 中配置 loader**
   在项目中的`/build/webpack.base.config.js`文件的`module.export`内的`module.rules`中加入解析 scss 文件的 loader：

   ```javascript
   module.exports = {
     // other configurations...
     module: {
       rules: [
         // ...
         {
           test: /\.scss$/,
           loaders: ['style','css','sass']
         }
       ]
     }
   }
   ```

3. **在 `.vue`文件中指定 `<style>`标签的语言为 scss**

   ```html
   <style lang="scss">
   /* scss code */
   </style>
   ```

> 如果以上步骤都是正确完成，运行代码还是报错，可以检查是否是 sass 依赖的版本过高导致的（因为我就遇到过这个问题），试试看 `sass-loader` 安装 `^7.3.1` 的版本能否解决你的问题。

scss 是可以和 `scoped`属性同时使用的。

```html
<style lang="scss" scoped>
/* scss code */
</style>
```

# 2. 在Vue中全局引入scss文件

写 scss 代码的时候会发现，有很多重复的 scss 代码（比如 mixins、常量等）。

如果我们将这些重复的代码单独拎出来放在一个 scss 文件中，每次使用的时候，再引入这个 scss 文件，那么最后，这个 scss 文件会被多次载入（`import`调用了几次，就会被载入几次），这种方式不用都知道问题很大。

```html
<style lang="scss">
  import '../style/style.scss';
</style>
```

那么就需要一种配置，就像`Vue.use()`那样，一经引入，全局可用。这个时候，就要用到`sass-resources-loader`。

1. **安装`sass-resources-loader`依赖。**

   ```powershell
   # 命令行敲入
   npm install sass-resources-loader --save-dev
   ```

2. **配置全局 scss 文件路径。**
   在`/build/utils.js`文件下找到下面指定的这行代码

   ```javascript
   // ...
   exports.cssLoaders = function (options) {
     // ...
     return {
       // ...
       scss: generateLoaders('sass'), /* 找到这行代码 */
       // ...
     }
   }
   ```

   替换成下面所示的内容

   ```javascript
   // ...
   exports.cssLoaders = function (options) {
     // ...
     return {
       // ...
       scss: generateLoaders('sass').concat(
         {
           loader: 'sass-resources-loader',
           options: {
             /* 后面的文件路径就是你所需要引入的全局scss文件路径 */
             resources: path.resolve(__dirname, '../src/style/style.scss')
           }
         }
       ),
       // ...
     }
   }
   ```

   `resources`属性支持数组值，可添加多个`scss`文件。

3. **重启服务**
   `utils.js`文件的修改需要重启服务才能生效，执行`npm run dev`重启。