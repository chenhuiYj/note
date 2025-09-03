

### Webpack 常见配置与作用说明

下面给出一个典型的 Webpack5 配置示例，并解释常见配置项的作用。为简洁起见，示例以单文件形式展示，实际项目可拆分为 `webpack.common.js`、`webpack.dev.js`、`webpack.prod.js` 并通过 `webpack-merge` 合并。

```js
// webpack.config.js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const CssMinimizerPlugin = require('css-minimizer-webpack-plugin');
const TerserPlugin = require('terser-webpack-plugin');
const webpack = require('webpack');

module.exports = (env, argv) => {
  const isProd = argv.mode === 'production';

  return {
    // 1) 运行模式
    mode: isProd ? 'production' : 'development',

    // 2) 入口
    entry: {
      main: path.resolve(__dirname, 'src/index.js')
    },

    // 3) 输出
    output: {
      path: path.resolve(__dirname, 'dist'),
      filename: isProd ? 'js/[name].[contenthash:8].js' : 'js/[name].js',
      chunkFilename: isProd ? 'js/[name].[contenthash:8].chunk.js' : 'js/[name].chunk.js',
      assetModuleFilename: 'assets/[name].[hash:8][ext][query]',
      publicPath: '/',
      clean: true
    },

    // 4) 模块解析
    resolve: {
      extensions: ['.mjs', '.js', '.json', '.jsx', '.ts', '.tsx'],
      alias: {
        '@': path.resolve(__dirname, 'src')
      },
      symlinks: true
    },

    // 5) 模块与 loader
    module: {
      rules: [
        {
          test: /\.(js|mjs|jsx|ts|tsx)$/,
          exclude: /node_modules/,
          use: [
            { loader: 'thread-loader', options: { workers: 2 } },
            {
              loader: 'babel-loader',
              options: { cacheDirectory: true }
            }
          ]
        },
        {
          test: /\.css$/,
          use: [
            isProd ? MiniCssExtractPlugin.loader : 'style-loader',
            {
              loader: 'css-loader',
              options: { importLoaders: 1, modules: false }
            },
            'postcss-loader'
          ]
        },
        {
          test: /\.(scss|sass)$/,
          use: [
            isProd ? MiniCssExtractPlugin.loader : 'style-loader',
            'css-loader',
            'postcss-loader',
            'sass-loader'
          ]
        },
        {
          test: /\.(png|jpe?g|gif|svg)$/i,
          type: 'asset',
          parser: { dataUrlCondition: { maxSize: 8 * 1024 } } // 小图转 base64
        },
        {
          test: /\.(woff2?|eot|ttf|otf)$/i,
          type: 'asset/resource' // 始终输出文件
        }
      ]
    },

    // 6) 插件
    plugins: [
      new HtmlWebpackPlugin({
        template: path.resolve(__dirname, 'public/index.html'),
        filename: 'index.html',
        inject: 'body',
        minify: isProd
          ? {
              removeComments: true,
              collapseWhitespace: true,
              removeRedundantAttributes: true
            }
          : false
      }),
      new webpack.DefinePlugin({
        'process.env.NODE_ENV': JSON.stringify(isProd ? 'production' : 'development')
      }),
      ...(isProd
        ? [new MiniCssExtractPlugin({ filename: 'css/[name].[contenthash:8].css' })]
        : [])
    ],

    // 7) 开发服务器
    devServer: {
      port: 3000,
      open: true,
      hot: true,
      historyApiFallback: true,
      static: { directory: path.resolve(__dirname, 'public') },
      proxy: {
        '/api': {
          target: 'http://localhost:8080',
          changeOrigin: true,
          pathRewrite: { '^/api': '' }
        }
      }
    },

    // 8) Source Map
    devtool: isProd ? 'source-map' : 'cheap-module-source-map',

    // 9) 性能/优化
    optimization: {
      minimize: isProd,
      minimizer: [
        new TerserPlugin({ parallel: true }),
        new CssMinimizerPlugin()
      ],
      splitChunks: {
        chunks: 'all',
        cacheGroups: {
          vendors: {
            test: /[\\/]node_modules[\\/]/,
            name: 'vendors',
            priority: -10
          },
          common: {
            minChunks: 2,
            name: 'common',
            priority: -20,
            reuseExistingChunk: true
          }
        }
      },
      runtimeChunk: 'single',
      moduleIds: 'deterministic'
    },

    // 10) 缓存
    cache: {
      type: 'filesystem'
    },

    // 11) 统计输出
    stats: 'errors-warnings'
  };
};
```

配置项作用说明（对应上文序号）：
- 1) mode：设置构建模式（development/production），影响默认优化与启用的插件。
- 2) entry：打包入口，支持多入口，形成多个初始 chunk。
- 3) output：产物输出目录与命名；`contenthash` 用于长期缓存；`clean` 构建前清理；`publicPath` 为资源引用前缀。
- 4) resolve：模块解析策略；`extensions` 自动解析后缀；`alias` 路径别名；`symlinks` 解析软链。
- 5) module.rules：配置各类资源的转换规则：
  - JS/TS：`babel-loader` 转译，`thread-loader` 并行；`cacheDirectory` 提速。
  - CSS/Sass：开发用 `style-loader` 热替换，生产抽离为独立 CSS，`postcss-loader` 做 Autoprefixer 等。
  - 资源：Webpack5 Asset Modules 统一处理图片/字体；`asset` 自动在 inline/resource 之间选择。
- 6) plugins：
  - `HtmlWebpackPlugin` 生成入口 HTML 并注入脚本/样式。
  - `DefinePlugin` 注入编译时常量。
  - `MiniCssExtractPlugin` 生产环境抽离 CSS 便于缓存。
- 7) devServer：本地开发服务；`hot` 热替换；`historyApiFallback` 支持前端路由；`proxy` 解决跨域联调。
- 8) devtool：Source Map 策略；开发选择构建快、定位准确的选项；生产用 `source-map` 或更安全的变体。
- 9) optimization：
  - `minimize/minimizer`：JS/CSS 压缩与并行。
  - `splitChunks`：抽取公共与第三方依赖，减少重复与首屏体积。
  - `runtimeChunk`：分离运行时代码，配合 `moduleIds: 'deterministic'` 稳定哈希，提升长期缓存。
- 10) cache：文件系统缓存，加速二次构建。
- 11) stats：精简控制台输出，聚焦错误与警告。
