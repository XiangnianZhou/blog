# 工作原理
webpack 基本的功能就是根据输入，进行编译输出。总体分为大体可以分为三个阶段： 
初始化 --> 编译 --> 输出，更具体的步骤如下：

> Webpack 的运行流程是一个串行的过程，从启动到结束会依次执行以下流程：
> 1. 初始化参数：从配置文件和 Shell 语句中读取与合并参数，得出最终的参数；
> 2. 开始编译：用上一步得到的参数初始化 Compiler 对象，加载所有配置的插件，执行对象的 run 方法开始执行编译；
> 3. 确定入口：根据配置中的 entry 找出所有的入口文件；
> 4. 编译模块：从入口文件出发，调用所有配置的 Loader 对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理；
> 5. 完成模块编译：在经过第4步使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系；
> 6. 输出资源：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk，再把每个 Chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会；
> 7. 输出完成：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统。
>
> 在以上过程中，Webpack 会在特定的时间点广播出特定的事件，插件在监听到感兴趣的事件后会执行特定的逻辑，并且插件可以调用 Webpack 提供的 API 改变 Webpack 的运行结果。
> 
> 引自《深入浅出 Webpack》（吴浩麟）

大概原理如上，我们心里先有个底，接下来我们根据源码逐步解析一下。

## 整体流程
首先我们跟着程序执行的脚步，来一探究竟。

从终端输入 `npm run webpack` 或 `npx webpack` 开始：
首先，从 `node_modules/.bin` 目录的 webpack 文件开始使用 node 调用 `node_modules/webpack/bin/webpack.js` 文件， 从此 webpack 和 webpack-cli 便开始运行了。


**webpack/bin/webpack.js：**
这里先有一波对 webpack/webpack-cli 安装判断，最后调用 `webpack-cli/bin/cli.js`。

**webpack-cli/bin/cli.js：**
1. 合并配置文件和命令行参数的配置（调用webpack-cli/utils/convert-argv.js）。
2. 调用 `webpack/lib/webpack.js`， 此文件导出一个 `webpack()` 函数； 执行这个函数得到 compiler 对象； 最后执行 `compiler.run()` 方法。
3. 最后在 `compiler.run()` 的回调中执行 `compiler.emitAssets()` 方法，根据 output 配置，输出文件。

到这里我们可以知道， webpack 的流程可以分为三个阶段： **初始化、编译和输出**。要高清楚如何编译，就得搞明白 `webpack/lib/webpack.js` 导出的函数和函数执行返回的对象， 所以，接下来 盘他～～

**webpack/lib/webpack.js：**
在这个模块里先定义了一个 `webpack()` 函数， 然后给这个导出的 `webpack()` 暴露了一波插件，其中便有我们熟悉的 `DefinePlugin` 和 `DllPlugin` 类似的插件。
联想这些插件的用法 `require('webpack').DefinePlugin` 我们可以猜到， 我们在 `webpack.config.js` 写的 `require('webpack')` 引用便是这个`webpack()` 函数。查看 `webpack\package.json` 的 main 字段可以证实我们的猜想。

闲言少叙，书归正文，我们最主要的是看执行 `webpack()` 函数发生了啥。
1. `compiler  =  new Compiler()` 得到了 compiler 对象。
2. 执行 `compiler.hooks.environment` 和 `compiler.hooks.afterEnvironment` 钩子。
3. 调用 `new WebpackOptionsApply().process()`,这一步的目的，简单理解就是**把 webpack 的配置项以各种方式怼到 compiler 对象里**。处理配置项output, target, externals, devtool等。又挂载了一波 webpack 内置插件（通过各个插件实例提供的 apply 方法，在不同的生命周期钩子（ compiler.hooks） 注册事件。了解更多可查看【链接】webpack 插件 里的介绍）。
4. 返回 compiler 对象。


compiler 对象在 webpack 中很重要，看官方文档：
> `Compiler` 模块是 webpack 的支柱引擎，它通过 [CLI](https://webpack.docschina.org/api/cli) 或 [Node API](https://webpack.docschina.org/api/node) 传递的所有选项，创建出一个 compilation 实例。它扩展(extend)自 `Tapable` 类，以便注册和调用插件。大多数面向用户的插件首先会会在 `Compiler` 上注册。

webpack 支柱引擎! 既然这么重要，我们需要看看 Compiler 类里到底定义了些什么！
- 继承自Tapable 关于 tapable 可以看这里 【链接】
- 定义 hooks 属性 (定义了一堆的生命周期钩子，用于注册插件。插件注册原理可以参考【链接 tapable】)
- 定义更多属性和方法，此处暂且不表(比如：watch,run,runAsChild,emitRecords...)




## 大概的概念
- 插件，就是一个模块
- 插件执行apply 方法，可以向 webpack 生命周期钩子 注册事件
- 通过 call callAsync 可执行 已经注册的事件的回调函数


## 一步步执行 webpack
发现，webpack 主体的执行流程基本是： 调用 compiler 对象的xx方法，调用xx钩子。
xx方法是把整个流程串起来，而钩子是执行挂在各个生命周期的事件回调。
可以认为，整个webpack 就是一个插件系统。

### 具体语法树和抽象语法树
简单理解是:
- 具体语法树，包含完整的源代码字符串信息。
- 相对源码，抽象语法树省略了部分信息，比如一些辅助符号。
- 一般的，在源代码的翻译和编译过程中，语法分析器创建出分
析树，然后从分析树生成抽象语法树。

webpack 使用 [acorn](https://github.com/acornjs/acorn) 进行编译

参考;
- [V8 引擎本用了什么编译技术](https://www.zhihu.com/question/19721167)
- [认识V8](https://zhuanlan.zhihu.com/p/27628685)
- [js](https://cheogo.github.io/learn-javascript/201709/runtime.html)

## 最后打包的文件
Webpack 其最初主要的目的是在浏览器端复用符合CommonJS规范的代码模块。

## 参考
- [webpack源码分析之四：plugin](https://segmentfault.com/a/1190000015836947#articleHeader7)

