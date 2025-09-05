### Webpack 常用 10 个 Loader 与作用说明

下列为实际项目中最常用的 10 种 loader，包含作用、典型配置与注意点（标注 Webpack5 的替代方案）。

1) babel-loader
- 作用：使用 Babel 转译现代 JS/TS 特性到更广泛兼容的 JS 语法，可结合 @babel/preset-env 按需引入 polyfill。
- 典型配置：
```js
{
  test: /\.(js|mjs|jsx)$/,
  exclude: /node_modules/,
  use: {
    loader: 'babel-loader',
    options: {
      cacheDirectory: true
    }
  }
}
```
- 注意：结合 `@babel/preset-env` 的 `useBuiltIns: 'usage'` 与 `core-js` 做按需 polyfill；开启 `cacheDirectory` 加速二次构建。

2) ts-loader
- 作用：使用 TypeScript 官方编译器把 `.ts/.tsx` 编译为 JS，保留更严格的类型检查能力。
- 典型配置：
```js
{
  test: /\.(ts|tsx)$/,
  use: [
    { loader: 'ts-loader', options: { transpileOnly: true } }
  ],
  exclude: /node_modules/
}
```
- 注意：`transpileOnly` 提升速度，类型检查可交由 `fork-ts-checker-webpack-plugin` 异步完成。

3) vue-loader
- 作用：解析 `.vue` 单文件组件，使其脚本、模板、样式等片段可被后续 loader 处理（Vue2/3 对应不同版本）。
- 典型配置：
```js
const { VueLoaderPlugin } = require('vue-loader');
{
  module: {
    rules: [
      { test: /\.vue$/, loader: 'vue-loader' }
    ]
  },
  plugins: [new VueLoaderPlugin()]
}
```
- 注意：务必同时注册 `VueLoaderPlugin`；Vue3 使用 `vue-loader@next`。

4) css-loader
- 作用：将 CSS 转为 JS 模块，支持 `@import` 与 `url()` 解析；可启用 CSS Modules 隔离样式作用域。
- 典型配置：
```js
{
  test: /\.css$/,
  use: [
    'style-loader',
    { loader: 'css-loader', options: { importLoaders: 1, modules: false } }
  ]
}
```
- 注意：`importLoaders` 指定在 `css-loader` 之前需要经过的 loader 数量（如 postcss、sass）。

5) style-loader
- 作用：在开发环境将样式通过 `<style>` 标签注入到页面，支持 HMR；生产环境常用 `mini-css-extract-plugin` 抽离 CSS 替代。
- 典型配置：
```js
{
  test: /\.css$/,
  use: ['style-loader', 'css-loader']
}
```
- 注意：仅建议开发阶段使用；生产建议抽取独立 CSS 以利用浏览器缓存。

6) sass-loader
- 作用：将 SCSS/SASS 编译为 CSS，配合 `css-loader` 与 `style-loader`/`MiniCssExtractPlugin.loader` 使用。
- 典型配置：
```js
{
  test: /\.(scss|sass)$/,
  use: [
    'style-loader',
    'css-loader',
    'postcss-loader',
    'sass-loader'
  ]
}
```
- 注意：需要安装 `sass`（Dart Sass）作为底层编译器。

7) less-loader
- 作用：将 LESS 编译为 CSS。
- 典型配置：
```js
{
  test: /\.less$/,
  use: [
    'style-loader',
    'css-loader',
    { loader: 'less-loader', options: { lessOptions: { javascriptEnabled: true } } }
  ]
}
```
- 注意：某些 UI 库（如 antd）自定义主题需启用 `javascriptEnabled`。

8) postcss-loader
- 作用：基于 PostCSS 对 CSS 做自动补全、现代特性转换等（如 Autoprefixer）。
- 典型配置：
```js
{
  test: /\.css$/,
  use: [
    'style-loader',
    'css-loader',
    'postcss-loader'
  ]
}
```
- 注意：项目根目录添加 `postcss.config.js`，启用 `autoprefixer`、`postcss-preset-env` 等插件。

9) url-loader（Webpack5 可用 Asset Modules: asset/inline 替代）
- 作用：将小体积资源转为 base64 内联，减少 HTTP 请求；大资源回退到 `file-loader` 输出文件。
- 典型配置：
```js
{
  test: /\.(png|jpe?g|gif|svg)$/i,
  use: {
    loader: 'url-loader',
    options: { limit: 8 * 1024, name: 'img/[name].[hash:8].[ext]' }
  }
}
```
- 注意：Webpack5 推荐使用内置 Asset Modules，功能更统一、无需额外 loader。

10) file-loader（Webpack5 可用 Asset Modules: asset/resource 替代）
- 作用：将资源文件复制到输出目录并返回其 URL。
- 典型配置：
```js
{
  test: /\.(woff2?|eot|ttf|otf|png|jpe?g|gif|svg)$/i,
  loader: 'file-loader',
  options: { name: 'assets/[name].[hash:8].[ext]' }
}
```
- 注意：Webpack5 使用 `type: 'asset/resource'` 可替代；统一到 Asset Modules 更易维护。

附：与上述 loader 常搭配的工具
- thread-loader：在耗时 loader 前启用多进程并行（如 babel/ts/sass），提升构建速度。
- mini-css-extract-plugin：生产环境抽离 CSS 成独立文件，以获得更好的缓存与并行加载。
