## 简介
由微软开发的一款开源的 JavaScript 超集语言。

JavaScript 超集：当前任何 JavaScript 都是合法的 TypeScript 代码。

TypeScript 主要是为 JavaScript 提供了类型系统和 ES6 语法的支持。

TypeScript 有自己的编译工具，写好的 TypeScript 代码最终会通过编译器编译成 JavaScript 代码进行运行。

### 静态类型的好处
1. 尽早发现代码中的 bug
2. 提高代码的可读性
3. 便于代码重构
4. 增强 IDE 的功能

### 静态类型的问题说明
1. 会增加代码量
2. 需要花时间掌握类型
3. 可能会降低开发效率

## 安装
选择全局安装或者在项目中进行安装，这里推荐在项目中进行安装。

全局安装
```bash
npm install typescript ts-node -g // ts-node 可以让我们直接在 node 环境中直接运行 ts 的代码
```

项目中安装
```bash
npm install typescript ts-node -D
```

## 配置文件
一般在项目中会结合配置文件来使用。使用 `npx tsc --init` 即可生成一个默认的配置文件。

配置文件的使用方式：`tsc -p ./tsconfig.json`。

### 相关配置项
- target，将 ts 代码转换成哪个版本的 JS 代码（ES5、ES3）
- module，将 ts 代码转换成 JS 代码后所使用的模块化标准
- outDir，ts 转换为 JS 后存放的位置
- rootDir，将哪个目录下的 ts 代码进行转换
- strict，是否将 ts 转换为严格的 JS 代码

