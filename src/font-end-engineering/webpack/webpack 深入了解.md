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

### 第三方库的两种引入方式
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
