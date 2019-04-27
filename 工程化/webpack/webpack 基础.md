## 概述

### 关于本文

最近有机会系统地总结一下 webpack 相关的东西，写篇文章来记录一下。写这篇文章一半是为了自己看，一半给别人看。如果有人看了这篇文章感觉很有用，真是我莫大的荣幸。
首先这**不是** 《深入浅出 webpack》， 也**不是**《一篇文章带你玩转 webpack》，更**不是** 《学习 webpack 这一篇就够了》。如果你想通过一篇文章就弄懂 webpack，这里不适合你。

### 如何学 webpack

**看文档**是最简单，最直接的方法，不过官方文档有些地方写的不太清楚，这时候，那些文章和博客的作用就比较明显了，毕竟这些是文章的作者们摸索出来的。比如这篇文章没有大篇幅复制官方文档，而是力求多介绍些没有在文档中体现的部分。

### 关于 webpack

webpack 是现在最流行的打包工具，即使现在的 webpack 也可以像 parcel 一样实现零配置打包；vue-cli3 也提供了开箱即用的零配置打包方案。但是，这些方案不一定能满足自己的需求。很多时候还需要我们自己根据自己的需求配置 webpack。

## 安装

-   webpack4.x 需同时安装 webpack-cli
-   推荐局部安装，然后可以配合 `npx` 命令运行局部的 webpack 和 webpack-cli

> 在命令行使用 npx 命令 调用 webpack： `npx webpack`
> npx 会调用 `node_modules/.bin` 下的 `webpack` 文件, 在这个文件里，会先调用 `webpack-cli` 以读取命令行参数。

## 概念

### 四个核心概念

-   入口（entry），指示 webpack 应该使用哪个模块，来作为构建其内部*依赖图*的开始。
-   出口(output)，告诉 webpack 在哪里输出它所创建的 _bundles_，以及如何命名这些文件。
-   loader, 将所有类型的文件，转换为应用程序的依赖图（和最终的 bundle）可以直接引用的模块。
-   插件, 在 webpack 打包过程中增加额外的功能。

### bundle, chunck 和 module

[**官方解释**](https://webpack.docschina.org/glossary) 大概说明了这几个概念；

1. 从先后顺序
   module：模块，webpack 模块（项目的资源）
   chunck： 代码块 (中间产物)
   bundle： 打包后的结果

2. 从“包含” 关系
   module <= chunck <= bundle

### module

在 webpack 中任何资源都被作为模块处理，webpack 支持 js 各种模块化标准的模块，和通过 loader 转化来的模块。

### chunck

**chunkname**, webpack 中每个 chunk 都会有一个名称：

-   如果，entry 是一个 string 或 array， 就只会生成一个 chunk 名称为 main
-   如果，entry 是 object，就可能会出现多个 chunk，chunk 的名称是 object 的键名。
-   如果是运行时产生的 chunk，如 `CommonsChunkPlugin`， `mini-css-extract-plugin` 插件产生的 chunk，需要在插件配置中指定 chunk 的名称。

### bundle

webpack 是个 JavaScript 打包器，bundle 是指 webpack 打包后的结果。

> 本质上，`webpack` 是一个现代 JavaScript 应用程序的静态模块打包器(module bundler)。当 `webpack` 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 `bundle`。

## 基本配置

[官方文档](https://webpack.js.org/concepts)
[中文文档](https://www.webpackjs.com/concepts/)

配置文件为： `webpack.config.js` 或 `webpackfile.js`；

### 入口

基本配置：

```
const config = {
  entry: {
    pageOne: './src/pageOne/index.js',
    pageTwo: './src/pageTwo/index.js',
    pageThree: './src/pageThree/index.js'
  }
};
```

### 出口

基础配置：

```
{
  output: {
    filename: '[name].js',
    path: '/home/proj/public/assets'
  }
}
```

注意： `path` 需为**绝对路径**

### webpack-dev-server

注意，这是第三方的模块, 需要另外安装。
webpack-dev-server 不是构建本地开发环境的唯一选择，其他方式有看[这里](https://webpack.js.org/guides/development/#using-watch-mode)。
其实 webpack-dev-server 会启动一个 express 服务，所以想要更到的自由度，可以直接使用 express 的 webpack-dev-middleware。

[`devServer`](https://webpack.js.org/configuration/dev-server) 选项用来配置 wepack-dev-server，基本配置：

```
 {
  devServer: {
    contentBase: path.join(__dirname, 'dist'), // 提供静态文件的目录
    compress: true,  // gzip 压缩
    publicPath: '/assets/', //  bundle 的可访问路径
    port: 9000
  }
}
```

#### live reloading 和 hot module replacement(HMR)

webpack-dev-server 默认已经实现了 live reloading，无需额外配置。
CSS 的 HMR 可以使用 style-loader。
Js 的 HMR ，需要在项目代码里写重载代码。一般借助开发框架提供的提供的 HMR 方案, 比如 vue 提供的 vue-loader。

### 模式

webpack4 新增了 mode 选项，可以指定开发模式/生产模式。

```
{
  mode: 'production'
}
```

如果值为 `development` 或 `production`， 会启用 webpack 内置的优化策略。

### loader

在 webpack 中使用非 js 模块（或需要特殊处理的 js 模块，比如 ES 模块）时，需要使用 loader 进行转换；使用 `module.rules` 配置 loader 的规则：

```
module: {
    rules: [
        {
            test: /\.html$/,
            use: 'html-withimg-loader'
        }
    ]
}
```

webpack 的配置比较灵活，很多的配置项都可以接受不同数据类型的值，比如，上面的 `test` 属性，可以接受字符串、正则表达式，数组，对象，甚至是函数。
同样，`use` 属性的配置也比较灵活，当只使用一个 loader 时，可以只写一个字符串（如 `css-loader`）；当使用多个 loader 时，可以写一个数组(`['css-loader', 'less-loader']`)，当需要给每个 loader 一些其他的配置时，可以把上面的字符串替换为一个对象：

```
{
    test: /\.(png|jpe?g|gif|svg)$/,
    use: {
        loader: 'url-loader',
        options: {
            limit: 8192,
            fallback: 'file-loader',
            name: 'img/[name]-[hash].[ext]'
         }
     }
}
```

**注意：** 多个 loader 的执行顺序是：自右向左，自下向上；
更多[ loader ](https://webpack.js.org/loaders)。

### 插件

配置 webpack，很大一部分是在配置插件。
基本的做法是，`new` 一个插件实例，传入一些配置信息：

```
{
    plugins: [
        new HtmlWebpackPlugin({
             title: 'My App'
        })
    ]
}
```

更多插件及相关配置 [看这里](https://webpack.js.org/plugins/)。

常见的几个插件；

-   `HtmlWebpackPlugin`，创建 HTML 文件
-   `TerserJSPlugin`，使用 `UglifyJS` 压缩混淆 js 代码
-   `MiniCssExtractPlugin`, 提取 CSS
-   `CleanWebpackPlugin`， 清除文件
-   `webpack.DefinePlugin`, 定义全局常量

注意，随着 webpack 的升级，一些老项目和网上的一些文章的插件已经过时了，比如：

1. 抽离 CSS 为独立文件，使用 [mini-css-extract-plugin](https://webpack.js.org/plugins/mini-css-extract-plugin/)， 而不再建议使用 `extract-text-webpack-plugin`。

2. webpack 默认安装 `terser-webpack-plugin`, 不再需要使用 `uglifyjs-webpack-plugin`。因为后者对[ ES6+ 的处理问题](https://github.com/webpack/webpack/pull/8036)。

3. 抽离公共代码的 `CommonsChunkPlugin` 现在使用功能更强大的 `optimization.splitChunks` 代替。

### 模块解析

webpack 与 Nodejs 的模块解析相似，查找规则基本相同，前者在功能上有所增强。
模块路径解析相关的配置都在 `resolve` 字段下，常用的有：

-   `resolve.alias`, 设置模块别名
-   `resolve.extensions`，自动解析确定的扩展名

```
resolve:  {
    extensions:  ['.js',  '.vue',  '.json'],
    alias:  {
        'service':  path.resolve(__dirname, 'src/js/service'),
        'utils':  path.resolve(__dirname, 'src/js/utils')
    }
}
```

### source map

#### js 的 source map

通过 `devtool` 配置，也可以使用 `SourceMapDevToolPlugin` 进行更细粒度的配置。
关于 `devtool`， [文档](https://www.webpackjs.com/configuration/devtool/) 给出了多种方式，每一种的运算速度都不一样，有的适合开发环境有的适合生产环境，可以根据自己的需求选择合适的配置。

#### css 的 source map

需要在样式处理的各个 loader 中打开 source map 配置。

## 优化

### loader 优化

loader 可以通过 `include` 和 `exclude` 配置来缩小文件的搜索范围。

babel-loader 官方提供了还提供了 `cacheDirectory` 配置，缓存编译结果，避免重复编译。

```JavaScript
{
    test: /\.js$/,
    exclude: /node_modules/,
    use:  {
        loader:  'babel-loader',
        options:  {
            presets:  ['@babel/preset-env'],
            cacheDirectory:  true
        }
    }
}
```

### 模块查找优化

#### resolve.modules

配置 webpack 去哪个目录下寻找第三方模块。其默认值为， `['node_modules']`。
和 Nodejs 的模块查找机制类似，webpack 会先从当前目录的 `./node_modules` 目录开始查找想要的模块，如果没找到，就去 `../node_modules` 中查找，再没有就去 `../../node_modules`， 以此类推。
一般情况下，我们的第三方模块都是安装在根目录的 `node_modules` 目录下的，此时可以通过 `resolve.modules` 指定固定的目录，以减少寻找。

```
modules: [path.resolve(__dirname, 'node_modules')]
```

#### resolve.alias

创建 `import` 或 `require` 的别名，使模块引入变得更简单。

有些库，安装到 `node_modules` 目录下的文件包括两部分：

-   模块化的代码，使用这些代码要经过 webpack 的编译；
-   已经打包好的单独的文件。

当第三方模块的完整性较强且模块入口为模块化代码，我们不希望 webpack 再次编译的时候，可以使用 `resolve.alias` 将导入模块换成使用已打包好的文件。

反之亦然，当只使用模块一部分功能，希望有效利用优化（如 Tree Shaking），且第三方模块入口为已打包的单独文件时，可以使用 `resolve.alias` 将导入的模块换成使用模块化的代码。

#### resolve.extensions

当导入预计没有后缀时，webpack 会自动加上后缀去尝试询问文件是否存在。`resolve.extensions` 便是用于配置在尝试过程中用到的后缀名列表。

-   后缀名列表要尽可能短；
-   频率出现高的文件后缀名放到前面；
-   在源码中，导入语句时，尽可能加上文件后缀。

#### module.noParse

让 webpack 忽略对部分**没有采用模块化** 的文件的递归解析。

### 动态链接库

项目中引入的第三方模块，每一次打包都要进行编译，特别消耗性能。DLLPlugin 提高打包速度的基本原理是，单独打包一次这些第三方模块，生成一个单独的文件，之后在正式打包时就可以直接从这个文件中引入之前打包好的第三方模块。

注意： DLL 常用于打包第三方模块，但不局限于第三方模块。

先从基本原理说起，比如我们要单独打包 A,B 两个库：

1. 单独打包出一个 js（**动态链接库**），暴露出一个变量，这个变量包含模块 A 和 B 以及其依赖模块，这些模块可以通过其在变量中的 ID 访问。
2. 动态链接库打包时，需要一个清单文件来存储这些依赖的和 ID 的对应关系。
3. 在 HTML 中引入单独打包好的 js 文件，使其暴露的变量在浏览器中可访问。
4. 真正打包项目时，需要去 2 中的清单中查找是否已经打包，当遇到已经打包过的模块，则不再打包。

对应的配置：

1. 要单独打包动态链接库 js，一般新建一个单独的 webpack 配置文件。为了便于多人合作，这个配置文件、打包好的动态链接库及其清单文件最好都使用版本管理。
2. 这个单独的 webpack 配置文档，同其它 webpack 配置文档基本一样，最大的不同是要使用 **webpack.DLLPlugin** 生成清单文件。
3. 浏览器中如果要能访问暴露出的变量，需要在 HTML 中使用 `script` 标签引入动态链接库，最笨的方法手写(：。
4. 真正打包项目时，需要通过 **webpack.DllReferencePlugin** 插件来检查模块是否存在动态链接库里。

例子：[官方提供的例子](https://github.com/webpack/webpack/tree/master/examples/dll)

#### 其他玩法

-   可以打包出多个动态链接库。
-   可以打包更多的资源，如 css。
-   <del>结合 npm script 或 webpack-html-plugin 可以实现过程更高的自动化。</del>

### externals

> `externals` 配置选项提供了「从输出的 bundle 中排除依赖」的方法。相反，所创建的 bundle 依赖于那些存在于用户环境(consumer's environment)中的依赖。

`externals`的优势在于，可以从 CND 引入资源，有效利用浏览器缓存。对于 jQuery 这样的库来说 `externals` 是一个不错的选择。

#### 与 DLL（动态链接库） 对比

以 `externals` 最终的打包结果（以 jQuery 为例）：

```
jquery:  function(module,  exports)  {
    eval('module.exports = jQuery;');
}
```

也就是 webpack 对 `externals` 的处理仅仅是把全局变量（`jQuery`）作为一个依赖模块进行管理，所以 `externals` 不具备管理*依赖的依赖*的能力，也就**无法避免依赖的依赖重复打包**。

DLL 也有类似的结果 （假如以`var __dll__vendor` 导出动态链接库）：

```JavaScript
'dll-reference __dll__vendor':  function(module,  exports)  {
    eval('module.exports = __dll__vendor;');
}
```

打开页面，可以在控制台访问到这些打包进动态链接库的模块：

```JavaScript
__dll__vendor.m[id];  // 通过id可以访问已经打包进动态链接库的模块
```

与 `externals` 相同，DLL 也把一个全局的变量作为一个依赖进行管理。不同的是，这个变量的结构和 webpack 运行时模块管理的结构是一样的，其中，依赖和依赖的依赖，都可以通过 ID 访问到，**可以避免重复打包**。

### 提取公共代码

`optimization.splitChunks` 选项通过配置 SplitChunksPlugin 来实现提取公共代码。一般的应用场景是：

-   提取单/多页面公共第三方库代码
-   提取单/多页面公共业务代码

webpack 在生产环境的提取公共代码策略和以下因素有关：

-   代码被多少次共享或是第三方库
-   代码块大小
-   并行按需加载的请求数量
-   页面初始化时的并行请求的数量

更多细节请移步[官方文档](https://webpack.js.org/plugins/split-chunks-plugin/)

我们在做公共代码的提取相关配置的时候，也是从上面的几个角度去配置的。
<del>比如要在一个单页应用中，打包抽离出第三方模块和公共业务代码:</del>

```JavaScript
// 此处缺一坨代码
```

### 按需加载

即懒加载，在初始化页面的时候不一次性加载所有代码，当需要更多内容时，再对用到的内容进行即时加载。

目前可用写法：

-   ES 规范的 `import()` 语法（推荐）
-   早期 webpack 提供的 `require.ensure`

```
import(/* webpackChunkName: "print" */ './print').then(module => {
    // ES6 模块的 import() 方法引入模块时，必须指向模块的 .default 值
     var print = module.default;
});
```

上面代码中，`/* webpackChunkName: "print" */`是以行内注释的方式为动态生成的 chunk 赋一个名称。此处还可以写更多的参数如：

-   `webpackPrefetch` 预取模块
-   `webpackPreload` 预加载模块

关于上面代码中的 `module.default`，需要注意：

> webpack v4 开始，在 import CommonJS 模块时，不会再将导入模块解析为 `module.exports` 的值，而是为 CommonJS 模块创建一个 artificial namespace object(人工命名空间对象)，关于其背后原因的更多信息，请阅读 [webpack 4: import() 和 CommonJs](https://medium.com/webpack/webpack-4-import-and-commonjs-d619d626b655)。

总结来说：

-   webpack4 与 webpack3 的 `import()`，对 CommonJS 模块处理方式不太一样；
-   webpack4 开始， `import()` 时，解析的都是一个命名空间对象了。
