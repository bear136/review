## 一、webpack 的作用

-   **模块打包 ：**将不同模块的文件打包在一起，使得他们之间引用正确，执行有序。
-   **编译兼容：** 通过 webpack 的 loader 机制，可以编译转换 .less .vue .jsx 等浏览器无法识别的格式文件
-   **能力扩展：** 通过 webpack 的 plugin 机制，可以进一步实现按需加载、代码压缩等功能

## 二、webpack 配置

-   entry ：入口
-   output：出口
-   loader: 用于**对模块源码的转换，**loader 描述了 webpack 如何处理非 javascript 模块，并且在 build 中引入这些依赖。loader 可以将文件从不同的语言（如 TypeScript）转换为 JavaScript，或者将内联图像转换为 data URL
-   plugins : 定义需要用的插件 。 目的在于解决 loader 无法实现的其他事，从打包优化和压缩，到重新定义环境变量，功能强大到可以用来处理各种各样的任务
-   mode ： 定义开发环境和生产环境
-   module： 决定结果和处理项目中不同类型的模块

## 三、webpack 的打包过程

-   **`初始化参数`**：从配置文件和 Shell 语句中读取与合并参数，得出最终的参数
-   **`开始编译`**：用上一步得到的参数初始化 Compiler 对象，加载所有配置的插件，执行对象的 run 方法开始执行编译
-   **`确定入口`**：根据配置中的 entry 找出所有的入口文件
-   **`编译模块`**：从入口文件出发，调用所有配置的 Loader 对模块进行翻译，再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理
-   **`完成模块编译`**：在经过第 4 步使用 Loader 翻译完所有模块后，得到了每个模块被翻译后的最终内容以及它们之间的依赖关系
-   **`输出资源`**：根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk，再把每个 Chunk 转换成一个单独的文件加入到输出列表，这步是可以修改输出内容的最后机会
-   **`输出完成`**：在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统

在以上过程中，`Webpack` 会在特定的时间点广播出特定的事件，插件在监听到感兴趣的事件后会执行特定的逻辑，并且插件可以调用 Webpack 提供的 API 改变 Webpack 的运行结果。

简单说

-   初始化：启动构建，读取与合并配置参数，加载 Plugin，实例化 Compiler
-   编译：从 Entry 出发，针对每个 Module 串行调用对应的 Loader 去翻译文件的内容，再找到该 Module 依赖的 Module，递归地进行编译处理
-   输出：将编译后的 Module 组合成 Chunk，将 Chunk 转换成文件，输出到文件系统中

## 四、常见的 loader 有哪些

`raw-loader`：加载文件原始内容（utf-8）

`file-loader`：把文件输出到一个文件夹中，在代码中通过相对 URL 去引用输出的文件 (处理图片和字体)

`url-loader`：与 file-loader 类似，区别是用户可以设置一个阈值，大于阈值会交给 file-loader 处理，小于阈值时返回文件 base64 形式编码 (处理图片和字体)

`source-map-loader`：加载额外的 Source Map 文件，以方便断点调试

`svg-inline-loader`：将压缩后的 SVG 内容注入代码中

`image-loader`：加载并且压缩图片文件

`json-loader` 加载 JSON 文件（默认包含）

`handlebars-loader`: 将 Handlebars 模版编译成函数并返回

`babel-loader`：把 ES6 转换成 ES5

`ts-loader`: 将 TypeScript 转换成 JavaScript

`awesome-typescript-loader`：将 TypeScript 转换成 JavaScript，性能优于 ts-loader

`sass-loader`：将 SCSS/SASS 代码转换成 CSS

`css-loader`：加载 CSS，支持模块化、压缩、文件导入等特性

`style-loader`：把 CSS 代码注入到 JavaScript 中，通过 DOM 操作去加载 CSS

`postcss-loader`：扩展 CSS 语法，使用下一代 CSS，可以配合 autoprefixer 插件自动补齐 CSS3 前缀

`eslint-loader`：通过 ESLint 检查 JavaScript 代码

`tslint-loader`：通过 TSLint 检查 TypeScript 代码

`mocha-loader`：加载 Mocha 测试用例的代码

`coverjs-loader`：计算测试的覆盖率

`vue-loader`：加载 Vue.js 单文件组件

`i18n-loader`: 国际化

`cache-loader`: 可以在一些性能开销较大的 Loader 之前添加，目的是将结果缓存到磁盘里

## 五、常见的 plugin 有哪些

`define-plugin`：定义环境变量 (Webpack4 之后指定 mode 会自动配置)

`ignore-plugin`：忽略部分文件

`html-webpack-plugin`：简化 HTML 文件创建 (依赖于 html-loader)

`web-webpack-plugin`：可方便地为单页应用输出 HTML，比 html-webpack-plugin 好用

`uglifyjs-webpack-plugin`：不支持 ES6 压缩 (Webpack4 以前)

`terser-webpack-plugin`: 支持压缩 ES6 (Webpack4)

`webpack-parallel-uglify-plugin`: 多进程执行代码压缩，提升构建速度

`mini-css-extract-plugin`: 分离样式文件，CSS 提取为独立文件，支持按需加载 (替代 extract-text-webpack-plugin)

`serviceworker-webpack-plugin`：为网页应用增加离线缓存功能

`clean-webpack-plugin`: 目录清理

`ModuleConcatenationPlugin`: 开启 Scope Hoisting

`speed-measure-webpack-plugin`: 可以看到每个 Loader 和 Plugin 执行耗时 (整个打包耗时、每个 Plugin 和 Loader 耗时)

`webpack-bundle-analyzer`: 可视化 Webpack 输出文件的体积 (业务组件、依赖第三方模块)

## 六、说一说 Loader 和 Plugin 的区别

**Loader** 本质上是一个函数，在该函数中对接收到的内容进行转换，返回转换后的结果，即对其他类型的资源进行预处理。

**Loader：** 在 module.rules 中进行配置，作为类型的解析规则，类型为数组。每一项都是 Object，包括了 test(类型文件) 、loader、options 等属性

**Plugin** 就是插件，基于事件流框架，可以拓展 webpack 功能。在 webpack 运行周期内会广播许多事件，Plugin 可以监听这些事件，再合适的时机通过 webpack 提供的 api 修改输出结果

**plugin** 在plugins中单独配置，类型为数组，每一项都是一个plugin的实例，参数通过构造函数传入

## 七、使用webpack开发时，你用过哪些可以提高效率的插件？

## 七、使用 webpack 开发时，你用过哪些可以提高效率的插件？

-   **webpack-dashboard :** 可以更好的展示相关打包信息

-   **webpack-merge:** 提取公共配置，减少重复代码

-   **speed-measure-webpack-plugin**：简称 SMP，分析出 Webpack 打包过程中 Loader 和 Plugin 的耗时，有助于找到构建过程中的性能瓶颈。

-   **size-plugin**：监控资源体积变化，尽早发现问题

-   **HotModuleReplacementPlugin**：模块热替换

## 八、webpack 热更新原理

webpack 的热更新也叫做热替换 （Hot Module Replacement） 简称 HMR 。 这个机制可以做到不用刷新浏览器而更将新变更的模块替换掉旧的模块

**wds：基于 node.js 的使用了 express 的 http 服务器**

客户端从服务端拉取更新后的文件，就是代码块需要更新的那部分，实际上 wds 与浏览器之间维护了一个**websocket**，当本地资源发生变化时，wds 会向浏览器推送更新，并且**带上构建时的 hash**，让客户端与上一次资源进行比对，客户端对比出差异后**会向 wds 发起 ajax 请求来获取更改内容（文件内容，hash）**，这样客户端就可以**借助这些信息继续向 wds 发起 jsonp 请求来获取 chunk 的增量更新**。后续由 hot module plugin 来完成后续工作，比如拿到增量更新如何处理，那些状态应该保留，哪些又需要更新；hotmoduleplugin 提供了相关 api 供开发者针对自身场景进行处理。

## 九、babel 的原理

-   **解析：** 将代码转化成 AST
    -   词法分析：将代码（字符串）分割正 token 流，即语法单元组成的数组
    -   语法分析 ： 分析 token 流并生成 AST
-   **转换：** 访问 AST 的节点进行变换操作生成新的 AST
-   **生成：** 以新的 AST 为基础生成代码

# Vite

## 一、Vite 的优点

-   快速的冷启动
-   即时的模块热更新
-   真正的按需编译

## 二、 Vite 和 Webpack 的区别

-   处理流程对比：

    -   webpack 会将整个应用进行打包，再将打包后的代码提供给 dev server

    -   Vite 直接将源码交给浏览器，实现 dev server 秒开。当浏览器需要相关模块时，再向 dev server 发送请求，服务器简单处理后，将请求的模块返回给浏览器，实现按需加载

-   热更新更快

    `Webpack`: 重新编译，请求变更后模块的代码，客户端重新加载

    `Vite`: 请求变更的模块，再重新加载

    `Vite` 通过 `chokidar` 来监听文件系统的变更，只用对发生变更的模块重新加载， 只需要精确的使相关模块与其临近的 `HMR`边界连接失效即可，这样`HMR` 更新速度就不会因为应用体积的增加而变慢，而 `Webpack` 还要经历一次打包构建

    -   创建一个`websocket`服务端和`client`文件，启动服务

    -   通过`chokidar`监听文件变更

    -   当代码变更后，服务端进行判断并推送到客户端

    -   客户端根据推送的信息执行不同操作的更新
