## Webpack 基础

### 安装
在所在项目中安装
```bash
npm i webpack webpack-cli -D // 当前所使用的版本为 5.x
```

### webpack 的使用

#### 1 webpack-cli 的使用
npm 5.2.x 之后提供了 `npx`，帮助执行项目中提供的指令包。

webpack4.0 之后可实现 0 配置打包，即默认打包 `src` 目录下的 `index` 文件.
但是 0 配置一般限制比较多，无法自定义很多配置。
常用的还是使用 webpack 配置文件进行打包构建。

#### 2 webpack 配置
**核心概念**
1. 入口（entry）：程序的入口 js
2. 输出（output）：打包后存放的位置
3. Loader：用于非 JS 模块的源代码进行转换
4. 插件（Plugins）：插件目的用于解决 Loader 无法实现的其他事情
5. 模式（mode）

**基本配置（webpack.config.js）**
```JavaScript
const path = require('path')
module.exports = {
  // 模式: development | production，也就是开发模式和生产模式，区别就是生产模式下会对代码进行压缩和混淆
  mode: 'development'
  // 入口配置
  entry: './src/index.js',
  // 出口配置文件
  output: {
    // 输出路径，webpack2 起就规定必须是绝对路径
    path: path.join(__dirname, 'dist'), // 也可以使用 path.resolve('dist')
    filename: 'bundle.js',
  }
}
```

**将 `npx webpack` 命令配置到 package.json 文件的脚本中**
具体是将命令配置到 `scripts` 字段中，具体使用情况如下：
```JavaScript
....
"scripts": {
  "build": "webpack" // 会默认使用 webpack.config.js 配置文件
  "dev": "webpack --config webpack.config.dev.js" // 指定使用 webpack.config.dev.js 配置文件
}
....
```

这里使用了一个 webpack 的 flag：`--config`，用于提供 webpack 配置文件的路径。

#### 3 开发时自动编译工具
1. watch 模式
2. webpack-dev-server （多数场景下都使用该方式）
3. webpack-dev-middleware

**3.1 watch**
作用是监视本地项目文件的变化，发现有修改的代码会自动编译打包生成输出文件。使用方式有以下两种：
- 配置在 package.json 文件中的 `scripts` 字段中：`webpack --watch`
- 配置在 webpack 配置文件中：`watch: true`

**3.2 webpack-dev-server（推荐）**
devServer 会在内存中生成一个打包好的 `bundle.js`，专供开发时使用，打包效率高，修改代码后会自动重新打包以及刷新浏览器，用户体验较好。
注意：devServer 服务运行在项目的 `public` 目录。webpack4.x 默认运行在项目根目录。

1. 安装
```bash
npm i webpack-dev-server -D // 注意，webpack-dev-server 依赖于 webpack
```

2. 将运行 webpack-dev-server 的指令配置到 package.json 的 scripts：`"dev": "webpack serve"`

3. 将 `index.html` 文件移至 `public` 目录中，同时将文件中的 js 文件路径修改为 `src="bundle.js"`

3. 运行：`npm run dev`

通过配置文件的方式配置 devServer 的其他参数辅助开发
```JavaScript
....
devServer: {
  hot: true,      // 开启支持热模块替换
  open: true,     // 启动服务后自动打开浏览器
  compress: true, // 启用 gzip 压缩
  port: 3000,     // 自定义服务端口，默认为 8080

  // 自定义服务器获取静态资源的路径。默认为 public 目录
  // 4.x 使用 contentBase 进行配置
  static: {
    directory: path.join(__dirname, 'assets'),
    publicPath: '/' // 相对于 directory 来说的路径，决定从哪个目录中提供打包好的文件
  },
}
....
```

**3.3 插件：html-webpack-plugin**
用途
1. 根据模板生成指定名称的 html 文件（类似于 devServer 生成在内存中的 `bundle.js`）
2. 会自动引入 `bundle.js`
3. 打包时会自动生成 html 文件

安装
```bash
npm i html-webpack-plugin
```

配置插件
```JavaScript
....
const HtmlWebpackPlugin = require('html-webpack-plugin');

plugins: [
  new HtmlWebpackPlugin({
    filename: 'index.html',
    template: 'template.html',
    scriptLoading: 'blocking', // 脚本加载方式：blocking | defer(默认) | module
  });
]
....
```

**3.4 webpack-dev-middleware**
`webpack-dev-middleware` 是一个容器（wrapper），他可以把 webpack 处理后的文件传递给一个服务器（server），`webpack-dev-server` 在内部使用了它，同时它也可以作为一个单独的包来使用，以进行更多自定义设置来实现更多的需求。

1. 安装 `express` 和 `webpack-dev-middleware`
```bash
npm i express webpack-dev-middleware -D
```

2. 新建 `server.js`
```JavaScript
const express = require('express')
const webpack = require('webpack')
const webpackDevMiddleware = require('webpack-dev-middleware')
const config = require('./webpack.config.js')

const app = express() // 使用 express 创建一个服务
const compiler = webpack(config)

app.use(webpackDevMiddleware(compiler, {
  publicPath: '/',
}))

app.listen(3000, function() {
  console.log('http://localhost:3000')
})
```

3. 配置 package.json 中 scripts：`"serve": "node server.js"`
4. 运行：`npm run serve`

注意：`webpack-dev-middleware` 必须使用 `html-webpack-plugin` 插件，否则 html 文件无法正确输出到 express 服务的根目录。

3.5 小结
自动编译工具主要都是为了提高开发体验。

#### 4 处理 CSS
处理 CSS 需要 `css-loader` 和 `style-loader` 两个 loader 来完成。

**作用**
- css-loader：解析 CSS 文件
- style-loader：将解析出来的结果加入到 HTML 中，使其生效

**安装**
```bash
npm install css-loader style-loader -D
```

**配置**
`loader` 需要配置在 `module.rules` 中。

```JavaScript
....
module: {
  rules: [
    {
      test: /\.css$/ // 表示用于解析后缀为 .css 的文件
      // webpack 读取 loader 是从右往左读取，会讲 CSS 文件先交给最右侧的 loader 来处理
      // loader 的执行顺序是从右到左以管道的方式链式调用
      use: ['style-loader', 'css-loader']
    }
  ]
}
....
```

#### 5 处理 less 和 sass
**安装**
```bash
npm i less less-loader -D // 处理 less
npm i sass-loader node-sass -D // 处理 sass
```

**配置**
```JavaScript
....
module: {
  rules: [
    ....
    {
      test: /\.less$/,
      use: ['style-loader', 'css-loader', 'less-loader']
    },
    {
      test: /\.s(a|c)ss$/,
      use: ['style-loader', 'css-loader', 'sass-loader']
    }
    ....
  ]
}
....
```
