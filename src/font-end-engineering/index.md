# 前端工程化

## 概述

### 定义
所有能降低成本，并且能提高效率的事情总称为工程化。

软件工程化关注的是性能、稳定性、可维护性等方面，一切以这些为目标的工作都是 “前端工程化”。

### 前端为什么需要工程化？
- 现在前端项目 “太大”、“复杂”
- 不再是后端的附属品

### 工程化解决的问题
- 提高工程师的开发效率
  - 扩展 JavaScript、HTML、CSS 本身的语言能力
  - 解决重复工作
  - 模块化、组件化
  - 解决功能复用和变更问题
- 进行高效的多人协作
- 保证项目的可维护性
- 提高项目的质量

### 发展历程的四个阶段
- 阶段一：库/框架——JQuery、Zepto、Angular、React、Vue
- 阶段二：简单构建优化——grunt、gulp
- 阶段三：JS/CSS模块化开发——AMD/CommonJS/UMD/ES6 Module Less/Sass/Stylus
- 阶段四：组件化开发与资源管理（分治思想）——微信小程序工程化

## 具体内容

### 前端模块化
- 职责
  - 模块管理
  - 资源加载
- 工具
  - NodeJS
  - npm
  - webpack
  - parcel
  - rollup


详情见具体章节：[前端模块化](https://github.com/renjie-run/blog/blob/master/src/font-end-engineering/前端模块化.md)

### 前端组件化（趋势）
- 优势：自由、灵活、可复用
- 相关实践：微信小程序（目录结构）

### 前端规范化
- 工具
  - eslint
  - stylelint

详情见具体章节：[代码规范](https://github.com/renjie-run/blog/blob/master/src/font-end-engineering/代码规范.md)

### 前端自动化
- 自动构建
  - grunt
  - gulp：压缩、校验、资源合并
- 自动化测试
  - 单元测试：Chai、Karma、Mocha
  - UI 测试： Jest、Enzyme、Selenium Webdriver
  - 覆盖率测试：istanbul
  - 性能测试：benchmark（基准测试）
  - 持续测试：codecov、Travis CI
- 自动部署
  - pm2
