---
title: 模块热替换(HMR) - React
contributors:
  - jmreidy
  - jhnns
  - sararubin
  - aiduryagin
  - rohannair
  - joshsantos
  - drpicox
---

W> __模块热替换__ 并非在生产环境中所使用的技术方案。 此方案的配置或环境都必须是用于开发过程中的。如果你现在需要知道如何在生产环境打包，请查阅[生产环境打包](/guides/production-build)的相关信息。

正如在[概念章节](/concepts/hot-module-replacement) 提到的，__模块热替换(HMR)__ 的作用是，在应用运行时，无手动需刷新页面，便能替换、增加、删除必要的模块。

__HMR__ 对于那些由单一状态树构成的应用非常有用。因为这些应用的组件是 "dumb" (相对于 "smart") 的，所以在组件的代码更改后，组件的状态依然能够正确反映应用的最新状态。

以下方案采用 `Babel` 和 `React` 举例, 但 __HMR__ 可以使用其他工具以各种其他方式实现。

The approach described below specifically uses Babel and React, but HMR can be done in a variety of other ways, using other tools.

T>  如果想查看其他配置方式的示例，可以告诉我们， 或者[提一个PR](https://github.com/webpack/webpack.js.org).

## 项目配置

首先，安装好将会使用的依赖包:

```bash
npm install --save-dev webpack webpack-dev-server
npm install --save-dev babel-core babel-loader babel-preset-es2015 babel-preset-react
npm install --save-dev style-loader css-loader
```

接下来需要安装React, ReactDOM, react-hot-loader (请使用@next版本)

W> 并未解释为何要使用@next版本

```bash
npm install --save react react-dom react-hot-loader@next
```

### Babel 配置

在项目根目录创建一个`.babelrc`文件，其内容配置信息如下：

__.babelrc__

```json
{
  "presets": [
    ["es2015", { "modules" : false }],
    // webpack 内置可以识别 import 的语法，并为其使用 tree shaking
    "react"
    // 将 React Components 转译为 JavaScript
  ],
  "plugins": [
    "react-hot-loader/babel"
    // 开起React代码中的HRM.
  ]
}
```

W> 参见 [tree shaking](https://www.zhihu.com/question/41922432)，译者认为，tree-shaking 可以从字面意思去理解，“摇晃树，使无用的叶子掉落”，也就过滤掉无用代码的一个具体实现。

我们需要使用 ES2015 模块来使 HMR 正常工作。 为此，在我们的 es2015 preset 设置中，将 `module` 的值设置为 false。我们可以使用 `babel-preset-env` 做类似的事情：

```json
["env", {"modules": false}]
```

将 Babel 的模块插件设置为 false，可以绕过很多问题（查看从 [v1 迁移到 v2](/guides/migrating/#mixing-es2015-with-amd-and-commonjs)和 [webpack-tree-shaking](http://www.2ality.com/2015/12/webpack-tree-shaking.html)）

W> 这段解释过于模糊

Setting Babel's module plugin to false helps fix many issues (see [Migrating from v1 to v2](/guides/migrating/#mixing-es2015-with-amd-and-commonjs) and [webpack-tree-shaking](http://www.2ality.com/2015/12/webpack-tree-shaking.html)).


注意：Node.js 还不支持 ES2015 模块，并且在webpack2 的配置文件中使用 ES2015 模块将造成 [问题](https://github.com/webpack/webpack.js.org/issues/154)。


W> 在现有的lts版本中，nodejs支持大部分的ES2015的语法，但不能使用 `import` `export` 等语法，其实V8还不支持。 -- 译者注

为此，你将需要创建两个 .babelrc 文件，来分别地 编译配置 和 应用程序代码 ：

- 1.一个放置在项目根目录中，并且在配置中使用 "presets": ["es2015"]
- 2.另一个放置在应用程序代码的源代码目录中

W> demo呢？

### webpack 配置

对于这个例子， 我们将使用一个 webpack 配置文件， 并假设：

- 所有应用程序源代码都位于 `<root>/src` 文件夹中
- 应用程序的入口起点位于 `/src/index.js`

目录结构

- --root
- --root/src
- --root/src/index.js

T> 请查看 [webpack-dev-server](/configuration/dev-server) 和 [概念](/concepts) 以熟悉下面的概念

__webpack.config.js__

```js
const { resolve } = require('path');
const webpack = require('webpack');

module.exports = {
  context: resolve(__dirname, 'src'),
  // webpack 解析模块的上下文目录
  entry: [
    'react-hot-loader/patch',
    // 开起React的HRM
    'webpack-dev-server/client?http://localhost:8080',
    // bundle the client for webpack-dev-server
    // and connect to the provided endpoint
    'webpack/hot/only-dev-server',
    // bundle the client for hot reloading
    // only： 只有更新成功时在执行热重载
    // only- means to only hot reload for successful updates
    './index.js'
    // 程序入口
  ],
  output: {
    filename: 'bundle.js',
    // 输出文件名
    path: resolve(__dirname, 'dist'),
    // 输出位置
    publicPath: '/'
    // 必须指定，HMR才之道在哪里加载更新的模块
    // necessary for HMR to know where to load the hot update chunks
  },

  devtool: 'inline-source-map',
  // source-map
  devServer: {
    hot: true,
    // 开起HMR服务
    contentBase: resolve(__dirname, 'dist'),
    // match the output path
    publicPath: '/'
    // 发布目录
  },

  module: {
    rules: [
      {
        test: /\.jsx?$/,
        use: [ 'babel-loader', ],
        exclude: /node_modules/
      },
      {
        test: /\.css$/,
        use: [ 'style-loader', 'css-loader?modules', ],
      },
    ],
  },

  plugins: [
    new webpack.HotModuleReplacementPlugin(),
    // 全局开起HMR
    new webpack.NamedModulesPlugin(),
    // HMR更新时在浏览器的console中显式可读性较好的模块名称
    // prints more readable module names in the browser console on HMR updates
  ],
};
```

W> If you are using the `options` key for your babel configuration, make sure you turn babel's `modules` feature off as mentioned in the [config section](https://webpack.js.org/guides/hmr-react/#babel-config).

### App code

Let's set up our React app:

__src/index.js__

```js
import React from 'react';
import ReactDOM from 'react-dom';

import { AppContainer } from 'react-hot-loader';
// AppContainer is a necessary wrapper component for HMR

import App from './components/App';

const render = (Component) => {
  ReactDOM.render(
    <AppContainer>
      <Component/>
    </AppContainer>,
    document.getElementById('root')
  );
};

render(App);

// Hot Module Replacement API
if (module.hot) {
  module.hot.accept('./components/App', () => {
    render(App)
  });
}
```

__src/components/App.js__

```js
import React from 'react';
import styles from './App.css';

const App = () => (
  <div className={styles.app}>
    <h2>Hello, </h2>
  </div>
);

export default App;
```

__src/components/App.css__

```css
.app {
  text-size-adjust: none;
  font-family: helvetica, arial, sans-serif;
  line-height: 200%;
  padding: 6px 20px 30px;
}
```

Important to note:

1. Setting set `devServer: { hot: true }` causes webpack will expose the `module.hot` API to our code

2. We use the `module.hot` hook to enable HMR for specific resources (`App.js` in this example). The most important property here is `module.hot.accept`, which specifies how to handle changes to specific dependencies.

3. Whenever `src/components/App.js` or its dependencies are changed `module.hot.accept` will fire the `render` method. The `render` method will even fire when `App.css` is changed because it is included in `App.js`.

__dist/index.html__

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Example Index</title>
</head>
<body>
  <div id="root"></div>
  <script src="bundle.js"></script>
</body>
</html>
```

We need to put the index.html file in our `dist` folder because `webpack-dev-server` will not run without it.


### Putting it all together

Finally, lets add a start task to `package.json`, that calls the `webpack-dev-server` binary.

__package.json__

```json
{
  ...
  "scripts" : {
    "start" : "webpack-dev-server"
  }
  ...
}
```

When we run `npm start`, it will launch the webpack dev server, causing our code to be transpiled by Babel, and bundled. Open a browser to `http://localhost:8080`, and check the JS console for logs similar to:

```bash
dev-server.js:49[HMR] Waiting for update signal from WDS…
only-dev-server.js:74[HMR] Waiting for update signal from WDS…
client?c7c8:24 [WDS] Hot Module Replacement enabled.
```

When you edit and save your `App.js` file, you should see something like the following in the console, and the App should update with changes.

```bash
[WDS] App updated. Recompiling…
client?c7c8:91 [WDS] App hot update…
dev-server.js:45 [HMR] Checking for updates on the server…
log-apply-result.js:20 [HMR] Updated modules:
log-apply-result.js:22 [HMR]  - ./components/App.js
dev-server.js:27 [HMR] App is up to date.
```

Note that HMR specifies the paths of the updated modules because we're using `NamedModulesPlugin`.
