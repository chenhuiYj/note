### Webpack 常见面试题（20题）

1.什么是 Webpack，它解决了什么问题？

Webpack 是一个模块打包器，支持 JS/CSS/图片等资源模块化，进行依赖分析、打包、压缩与优化，解决浏览器不支持模块规范、资源散乱、体积大、重复请求等问题。

2.Webpack 的核心概念有哪些？

entry（入口）、output（输出）、loader（资源转换）、plugin（功能扩展）、module（模块）、chunk（代码块）、bundle（产物）、asset（资源）、mode（模式）、devServer（开发服务）。

3.loader 与 plugin 有何区别？
答：loader 负责把某一类资源转换为模块（如 Babel 转译 JS、处理样式/图片），本质是函数；plugin 基于 Tapable 的事件钩子扩展编译流程，能做打包阶段的任意能力增强（优化、注入、产物处理）。

4.Webpack 的构建流程（高层）是什么？

初始化参数 → 创建编译器 → 执行编译（从入口解析依赖图，调用 loader 转换）→ 生成 chunk → 调用 plugin 进行优化/生成资源 → 输出文件到 output 目录。

5.HMR（热模块替换）原理是什么？

运行时与开发服务器建立连接（WebSocket/EventSource），当模块变化时只推送变更的模块 diff，运行时用新模块替换旧模块并触发模块级更新，无需整页刷新，状态可保持。

6.什么是 Tree Shaking，如何生效？

利用 ES Module 静态分析删除未引用的导出。需使用 ESModule 语法、在 production 模式、开启压缩，配置 `sideEffects` 告知无副作用文件可安全删除。

7.如何进行代码分割（Code Splitting）？

- 动态导入：`import()` 按需加载
- 多入口：多个 entry 输出多 bundle
- 公共依赖抽取：`optimization.splitChunks` 抽出共享模块

8.如何做浏览器长期缓存（Long-term Caching）？

使用 `[contenthash]` 命名、分离 runtime：`optimization.runtimeChunk: 'single'`、稳定 moduleId（`deterministic`）、只在内容变更时变更文件名，配合 CDN 缓存策略。

9.Source Map 有哪些类型，生产环境如何选择？

常见有 `eval`, `inline`, `cheap`, `module`, `hidden`, `nosources` 等组合。开发建议 `cheap-module-source-map`；生产常用 `source-map`（外链）或 `hidden-source-map`/`nosources-source-map`（兼顾安全）。

10.Babel 与 Webpack 如何集成且优化？

用 `babel-loader` 转译，开启 `cacheDirectory` 缓存，必要时配合 `thread-loader` 并行；生产使用按需 polyfill（`@babel/preset-env` + `useBuiltIns: 'usage'`）。

```js
// webpack.module.rules 示例
{
  test: /\.m?js$/,
  exclude: /node_modules/,
  use: [
    { loader: 'thread-loader', options: { workers: 2 } },
    { loader: 'babel-loader', options: { cacheDirectory: true } }
  ]
}
```

11.如何提升构建速度？

- 开启持久化缓存（webpack5 `cache: { type: 'filesystem' }`）
- 减少解析范围（`include/exclude`、`resolve.extensions`、`alias`）
- 并行/多进程（`thread-loader`, `Terser` 并行）
- 使用 `oneOf` 减少 rule 匹配次数
- 合理分包与按需编译

12.图片/字体等静态资源如何处理（Webpack5）？

使用 Asset Modules：`asset/resource`（复制文件）、`asset/inline`（base64）、`asset`（自动选择），替代 `file-loader`/`url-loader`。

```js
{
  test: /\.(png|jpg|gif|svg|woff2?)$/i,
  type: 'asset',
  parser: { dataUrlCondition: { maxSize: 8 * 1024 } }
}
```

13.如何管理环境变量与多环境配置？

分环境配置文件（`webpack-merge` 合并）、使用 `DefinePlugin` 注入构建时常量，配合 `cross-env` 设置 `NODE_ENV`。

```js
const { merge } = require('webpack-merge');
const base = require('./webpack.base');
module.exports = merge(base, { mode: 'production' });
```

14.Tree Shaking 常见“失效”原因？

使用 CommonJS（无法静态分析）、未设置 `sideEffects`、Babel 把 ESModule 转成 CJS（需 `modules: false`）、库模块有副作用、未开启生产优化或压缩器未做 DCE。

15.如何实现懒加载并提示浏览器预取/预加载？

使用动态导入并添加魔法注释 `webpackPrefetch`/`webpackPreload` 改善资源调度。

```js
import(/* webpackChunkName: 'user', webpackPrefetch: true */ './User');
```

16.`optimization.splitChunks` 关键配置有哪些？

`chunks`（all/async/initial）、`minSize`、`minChunks`、`maxInitialRequests`、`maxAsyncRequests`、`cacheGroups`（如 vendors、common 自定义规则）。

```js
optimization: {
  splitChunks: {
    chunks: 'all',
    cacheGroups: {
      vendors: { test: /[\\/]node_modules[\\/]/, name: 'vendors', priority: -10 },
      common: { minChunks: 2, name: 'common', priority: -20, reuseExistingChunk: true }
    }
  }
}
```

17.loader 的执行顺序与原理？

同一 rule 的 `use` 从右到左（或从下到上）执行；loader 可实现 `pitch` 阶段；返回 JS 模块字符串给下一环处理，最终交给 Webpack 解析依赖。

18.plugin 的工作机制是什么？

基于 Tapable 事件流，plugin 通过在编译生命周期钩子（如 `compile`、`emit`、`done`）上注册回调来读写 compilation、修改模块/资源或注入额外逻辑。

19.Webpack5 有哪些重要新特性？

- 模块联邦（Module Federation）实现跨应用共享运行时模块
- 内置 Asset Modules（替代旧 loader）
- 持久化文件缓存、改进 Long-term Caching
- 更好的 Tree Shaking 与 sideEffects 处理

20.如何减小产物体积（运行时性能优化）？

- 生产模式，启用压缩（`TerserPlugin`、`css-minimizer-webpack-plugin`）
- Scope Hoisting（`ModuleConcatenationPlugin`，生产模式默认）
- 按需加载与代码分割，剔除多余 polyfill
- 使用 `externals` 或 CDN 承载大库
- 移除调试代码与冗余依赖、开启 gzip/br 压缩（服务器侧）

