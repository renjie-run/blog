# Webpack 性能优化

## 基本优化
- production 模式打包自带优化。属于 webpack 自带的优化功能。
- tree shaking：打包时移除 JS 中未使用的代码（dead-code）。该优化依赖于 ES6 模块系统中 `import` 和 `export` 的静态结构特性。属于 webpack 自带的优化功能。
- scope hoisting：作用时将模块之间的关系进行推测，可以让 webpack 打包出来的文件更小，运行的更快。原理是分许模块之间的关系，尽可能的把零散的模块合并到一个函数中去，但前提是不会造成代码冗余。属于 webpack 自带的优化功能。
- 代码压缩：将所有的 JS 代码使用 `uglifyjs-webpack-plugin` 插件进行压缩、混淆。

## CSS 优化

### 将 CSS 提取到单独的文件
使用 `mini-css-extract-plugin` 将 CSS 提取为独立的文件，该插件对每个包含 CSS 的 JS 文件都会创建一个 CSS 文件，支持按需加载 CSS 和 sourceMap。

**优点**
- 异步加载
- 不需要重复编译，性能更好
- 更容易使用
- 只针对 CSS

**配置使用**

1. 安装
```bash
npm install mini-css-extract-plugin -D
```

2. 配置
```JavaScript
....
const MiniCssExtractPlugin = require('mini-css-extract-plugin')

plugins: [
  ....
  new MiniCssExtractPlugin({
    filename: 'css/[name]-[hash:4].css', // 支持 placeholder 语法
  })
  ....
],
....
```

3. 需要将 loader 中使用的 `style-loader` 替换为 `MiniCssExtractPlugin.loader`，因为 CSS 都提取到了单独的文件中，不需要再将 CSS 写入到行内。所以不再需要 `style-loader`，而是需要 `MiniCssExtractPlugin` 中的内置 loader 来处理样式。
