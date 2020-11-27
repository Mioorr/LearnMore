# Webpack入门

随着Web应用变得庞大复杂，纯靠直接编写JS、CSS、HTML根本应对不了这种发展，因此出现了很多其他的规范、框架，但我们最终需要将这些编译成浏览器能真正认识的JS、CSS、HTML，这时候就需要有个工具帮助我们编译，Webpack做的就是这样的事情，但不止于此。

简而言之，**Webpack是一个JS应用程序的静态模块打包器**。

## 1. 安装

查看 Webpack 安装请移步：[安装Webpack](https://blog.csdn.net/Mirror_r/article/details/90670892)。

## 2.核心概念

Webpack 有以下几个核心概念：

1. `Entry`：入口，Webpack 执行构建的第一步从 `Entry` 开始。
2. `Module`：模块，在 Webpack 里一切皆是模块，一个模块对应着一个文件。Webpack 会从配置的 Entry 开始递归找出所有依赖的模块。
3. `Chunk`：代码块，一个 Chunk 由多个模块组合而成，用于代码合并与分割。
4. `Loader`：模块转换器，用于把模块原内容按需求转换成新内容。
5. `Plugin`：扩展插件，在 Weboack 构建流程中的特定时机注入扩展逻辑来改变构建结果或做你想要的事情。
6. `Output`：输出结果，在 Webpack 经过一系列处理并得出最终想要的代码后输出结果。

Webpack 启动 => 从 Entry 里配置的 Module 开始递归解析 Entry 依赖的所有 Module => 每找到一个 Module，就根据配置的 Loader 去找出对应的转换规则对 Module 进行转换 => 转换 Module 后，解析出当前 Module 依赖的 Module（这些模块会以 Entry 为单位进行分组，一个 Entry 和其所有依赖的 Module 被分到一个组，也就是一个 Chunk） => Webpack 把所有 Chunk 转换成文件输出。
在整个流程中 Webpack 会在恰当的时机执行 Plugin 里定义的逻辑。

## 3. 配置

配置 Webpack 的方式有2种：

1. 通过一个 JS 文件描述配置；
2. 执行 Webpack 可执行文件时通过命令行参数传入。

### 3.1 Context

Webpack 在寻找相对路径文件时会以`context`为根目录，`context`默认为执行启动 Webpack 时所在的工作目录。
如果想改变`context`的默认配置，则可以在配置文件里设置：

```javascript
module.exports = {
  context: path.resolve(_dirname, 'app');
};
```

> <font color="orange">Notice：</font>`context`必须是一个绝对路径的字符串。

也可以通过在启动 Webpack 时带上参数`webpack --context`来设置`context`。

`entry` 的路径和其依赖的模块的路径可能采用相对于`context`的路径来描述，`context`会影响到这些相对路径所指向的真实文件。

### 3.1 Entry <font color="red">*</font>

`entry` 是配置模块的入口，Webpack 执行构建的第一步从`entry`开始搜寻及递归解析出所有 Entry 依赖的模块。

`entry`配置是**必填**的，否则将导致 Webpack 报错退出。

**Entry 类型**

| 类型   | 例子                                                         | 含义                                   |
| ------ | ------------------------------------------------------------ | -------------------------------------- |
| String | `./app/entry`                                                | 入口模块的文件路径<br />可以是相对路径 |
| Array  | `['./app/entry1', './app/entry2']`                           | 入口模块的文件路径<br />可以是相对路径 |
| Object | `{a: './app/entry-a', b: ['./app/entry-b1', './app/entry-b2']}` | 配置多个入口，每个入口生成一个 Chunk   |

如果是`Array`类型，在搭配`output.library`配置项使用时，只有数组里的最后一个入口文件的模块会被导出。



