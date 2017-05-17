---
title: 安装
contributors:
  - pksjce
  - bebraw
  - simon04
sort: 10
---

### 环境要求

在安装 `webpack` 之前，需要确认您已经安装 [Node.js](https://nodejs.org/en/) 的最新版本, 当然 我们更推荐安装 `LTS` 版本,如果使用旧版，在使用过程中可能遇到各种问题，比如 `webpack` 的依赖package丢失或者无法兼容等问题。

接下来介绍如何在项目环境中安装 `webpack`

### 本地安装

W> 本地和全局是相对概念，本地一般是指项目目录环境，而全局则为操作系统环境 -- 译者注

目前webpack的最新发布版本为: [![GitHub release](https://img.shields.io/github/release/webpack/webpack.svg?style=flat-square)](https://github.com/webpack/webpack/releases)

``` bash
npm install --save-dev webpack

npm install --save-dev webpack@<version>
```

如果你在项目中使用npm命令来执行webpack，npm会优先从项目中的 `node_modules` 下去查找当前项目下的 `webpack` 模块包

W> 将 webpack的依赖信息显示的声明在项目代码的 `package.json` 的`devDependencies` 字段以便项目环境迁移后不会应为全局的 `webpack` 版本与你预期环境的版本产生冲突，这个应该在研发规范里面设定 --译者注

```json
"scripts": {
	"start": "webpack --config mywebpack.config.js"
}
```

以上是截取自 `package.json` 的一段配置，我们推荐在 `package.json` 中的 `scripts` 中添加命令来运行 `webpack` 命令，并显示的指定 `webpack.config.js` 配置文件的位置

T> 当你在本地安装 webpack 后，你能够从 node_modules/.bin/webpack 访问它的 bin 版本。


### 全局安装

W> 不推荐。这会锁定 webpack 到指定版本，并且在使用不同的 `webpack` 版本的项目中可能会导致构建失败。

``` bash
npm install --global webpack
```

全局安装完成后 `webpack` 命令现在可以任意的地方执行

### 最新版本

如果你想体验最新版本的 `webpack` （不稳定的版本），你可以直接从 `webpack` 的仓库中安装：

``` bash
npm install webpack/webpack#<tagname/branchname>
```
