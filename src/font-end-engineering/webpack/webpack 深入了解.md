# Webpack 深入了解

### 1.HTML 中 img 标签的图片资源处理
使用时只需要在 html 中正常引入即可， webpack 会找到对应的资源进行打包，并修改 html 中的引入路径。

配置
```JavaScript
....
  module: {
    ....
    rules: [
      test: /\.(htm|html)$/,
      use: 'html-withimg-loader',
    ]
    ....
  }
....
```

注意：该 loader 不能与 copy-webpack-plugin 插件同时使用。

### 2.多页应用打包
在很多业务场景中需要打包为多页面的应用。此时，我们需要修改 webpack 配置。
```JavaScript
....
// 修改入口和出口配置
entry: {
  index: '.src/index.js',
  other: '/src/other.js',
},

....

// 多入口无法对应一个固定的出口，这里需要将 filename 修改为动态变量的形式，目标是为了相应的输出各自的打包文件
output: {
  filename: '[name].js',
}

....

// 多页面也就需要多个 html 文件与之相对应
plugins: [
  new HtmlWebpackPlugin({
    filename: 'index.html',
    template: 'index.template.html',
    chunks: ['index'], // 指定该页面也引入的 JS 文件，否则会全部引入
  }),

  new HtmlWebpackPlugin({
    filename: 'other.html',
    template: 'other.template.html',
    chunks: ['other'],
  }),
]

....
```

### 3.第三方库的两种引入方式
这里以引入 jquery 为例。

- expose-loader 进行全局变量的注入。在具体模块中使用还需要 `import` 或 `require` 相关库。
- 内置插件 `webpack.ProviderPlugin` 对每个模块的闭包空间注入一个变量，自动加载模块。

**配置使用 expose-loader**
```JavaScript
....
module: {
  rules: [
    ....

    {
      test: require.resolve('jquery'), // require.resolve 用于获取模块的绝对路径
      use: {
        loader: 'expose-loader',
        options: {
          exposes: ['$', 'jQuery'],
        }
      }
    },

    ....
  ]

....
```

**配置使用 webpack.ProviderPlugin**
```JavaScript
....

plugins: [
  ....
  new webpack.ProviderPlugin({
    $: 'jquery',
    jQuery: 'jquery',
  }),
  ....
],

....
```

### 4.development/production 不同配置文件打包
项目开发一般需要使用两套配置方案，分别用于开发阶段打包（不压缩代码、不优化代码、增加效率）、上线阶段打包（压缩代码、优化代码、打包后直接上线使用）。

可分为以下 3 个配置文件：
- webpack.base.js // 公用配置项
- webpack.dev.js  // 开发相关配置
- webpack.prod.js // 生产环境相关配置

可按照以下步骤完成不同环境下的配置：
1. 将开发环境、生产环境中共用的配置都放到 `webpack.base.js` 配置文件中，专属配置写到各自的配置文件中（例如，mode）。
2. 然后通过 `webpack-merge` 工具将 `webpack.base.js` 中的配置合并到各自的专属配置文件中。
3. 修改 `package.json` 中的脚本参数，通过 `--config` 指定打包命令所要使用的配置文件。

**如何使用 webpack-merge**
```JavaScript
....
const baseConfig = require('./webpack.base.js')
const webpackMerge = require('webpack-merge')
module.exports = webpackMerge(baseConfig, {
  // 这里写专属配置
})
....
```

**配置文件归类**

现在的配置文件都放在根目录下，一般会将这些配置文件归类到根目录下的 `config` 或 `build` 目录中。具体操作如下：
1. 归类到新的目录下，配置文件中使用到了绝对路径的地方需要做相应的调整。
2. 将 package.json 中所指定的配置文件的路径修改为调整后的配置文件路径。


### 5. 定义环境变量
通过内置插件 `webpack.DefinePlugin` 插件可以定义环境变量，可以用来区分开发环境和生产环境。

具体配置
```JavaScript
....
plugins: [
  ....
  new webpack.DefinePlugin({
    IS_DEV: JSON.stringify(true), // 代码中可以直接使用定义的环境变量：IS_DEV
  }),
  ....
]
....
```

应用案例：
- 通过环境变量实现开发阶段与上线阶段的 API 地址自动切换
