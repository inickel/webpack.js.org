---
title: 开始
sort: 0
contributors:
  - bebraw
  - varunjayaraman
  - cntanglijun
  - chrisVillanueva
  - johnstew
  - simon04
  - aaronang
  - jecoopr
---

首先，webpack 是一个用来构建应用程序（通常是web）的构建工具，在[安装](/guides/installation)完 `webpack` 后，分别可以用 [cli](/api/cli) 或 [api](/api/node) 来使用 `webpack` 。 `webpack` 通过快速建立应用程序的依赖关系并以正确的顺序打包它们来简化你的工作流。 你还能够针对具体情况来对 `webpack` 进行自定义的优化配置，比如为生产环境拆分 vendor/css/js 代码，运行一个开发服务, 来实现无刷新热重载(hot-reload)等很多酷(zhuang)炫(bi)的特性。了解更多请阅读 [为什么要使用webpack](/guides/why-webpack)

## 创建一个bundle

先创建一个示demo目录来 使用 `wepback` 。并 [安装 webpack](/guides/installation) 到本地环境。

Create a demo directory to try out webpack. [Install webpack](/guides/installation).

```bash
mkdir webpack-demo && cd webpack-demo
npm init -y
npm install --save-dev webpack
```

```bash
./node_modules/.bin/webpack --help # Shows a list of valid cli commands
.\node_modules\.bin\webpack --help # For windows users
webpack --help # If you installed webpack globally
```

再创建一个子目录 `app` 并在其下创建一个 `index.js` 文件。

__app/index.js__

```javascript
function component () {
  var element = document.createElement('div');

  /* lodash is required for the next line to work */
  element.innerHTML = _.join(['Hello','webpack'], ' ');

  return element;
}

document.body.appendChild(component());
```

创建一个入口页面 `index.html`，以便观察其上面的代码的执行效果.

__index.html__

```html
<html>
  <head>
    <title>webpack 2 demo</title>
    <script src="https://unpkg.com/lodash@4.16.6"></script>
  </head>
  <body>
    <script src="app/index.js"></script>
  </body>
</html>
```

在示例中，两个 `<script>` 标签之间其实存在隐式依赖关系。

`index.js` 在执行过程中会依赖于在前面引用的 `loadash` ，之所以说是隐式依赖，是因为 `index.js` 中没有显式的声明其对 `lodash` 有依赖关系，只是假设已经在全局变量中存在一个 `_` 的变量。

这种代码的管理很显然是有问题

- 如果依赖不存在，或者是引入的顺序错误，比如会导致程序报错
- 如果引入了该模块，但未被使用，而浏览器又会去下载不需要的代码。

运行 index.js 会依赖于页面中提前引入的 lodash。之所以说是隐式的是因为 index.js 并未显式声明需要引入 lodash，只是假定推测已经存在一个全局变量 `_`。

使用这种方式去管理 JavaScript 项目会有一些问题：

如果依赖不存在，或者引入顺序错误，应用程序将无法正常运行。
如果依赖被引入但是并没有使用，那样就会存在许多浏览器不得不下载的无用代码。
要在 index.js 中打包 lodash 依赖，首先我们需要安装 lodash。

将 `lodash` 和它的依赖文件`index.js` 打包到一起，首先需要安装 `loadash`

```bash
npm install --save lodash
```

修改 `index.js` 中的代码

__app/index.js__

```diff
+ import _ from 'lodash';

function component () {
  ...
```
然后修改 `index.html` 的代码，引用打包后的js文件

```diff
 <html>
   <head>
     <title>webpack 2 demo</title>
-    <script src="https://unpkg.com/lodash@4.16.6"></script>
   </head>
   <body>
-    <script src="app/index.js"></script>
+    <script src="dist/bundle.js"></script>
   </body>
 </html>
```

在这个例子中，`index.js` 显式要求引入的 `lodash` 必须存在，然后将它以 `_` 的别名绑定（不会造成全局范围变量名污染）。

通过声明模块所需的依赖， `webpack` 能够利用这些信息去构建依赖关系，然后使用这个关系表生成一个优化过的，会以正确代码顺序被运行的 bundle。并且没有用到的依赖将不会被 bundle 引入。

在此目录下运行 `webpack` ，其中 `index.js` 是入口文件，`bundle.js` 是已打包所需的所有代码的输出文件。

```bash
./node_modules/.bin/webpack app/index.js dist/bundle.js

Hash: ff6c1d39b26f89b3b7bb
Version: webpack 2.2.0
Time: 385ms
    Asset    Size  Chunks                    Chunk Names
bundle.js  544 kB       0  [emitted]  [big]  main
   [0] ./~/lodash/lodash.js 540 kB {0} [built]
   [1] (webpack)/buildin/global.js 509 bytes {0} [built]
   [2] (webpack)/buildin/module.js 517 bytes {0} [built]
   [3] ./app/index.js 278 bytes {0} [built]
```

T> 输出结果可能会稍有不同。但只要构建成功，那么你就可以继续。

T> Output may vary. If the build is successful then you are good to go.

在浏览器中打开 `index.html`，查看构建成功后的 `bundle` 的结果。你能看到带有以下文本的页面：'Hello webpack'。

## 在 webpack 中使用 ES2015 模块

你肯能已经注意到了，在 `app/index.js` 中使用的 [ES2015 module import](https://developer.mozilla.org//en-US/docs/Web/JavaScript/Reference/Statements/import) (alias ES2015, *harmony*)  尽管 `import/export` 等关键字在浏览器中还未被支持，你也可以正常的使用，因为 `webpack` 会将其替换为 ES5 兼容的代码。你可以检查 `dist/bundle.js` 的代码确实是已经转译成ES5的代码。

注意，webpack 不会更改你的代码中除 import/export 以外的部分。如果你在使用其它 ES2015 特性，请确保你使用了一个像是 Babel 或 Bublé 的转译器。
Noticed the use of [ES2015 module import](https://developer.mozilla.org//en-US/docs/Web/JavaScript/Reference/Statements/import) (alias ES2015, *harmony*) in `app/index.js`? Although `import`/`export` statements are not supported in browsers (yet), using them is fine since webpack will replace those instructions with an ES5 compatible wrapper code. Inspect `dist/bundle.js` to convince yourself.

注意， `webpack` 不会更改你的代码中除 `import/export` 以外的部分。 如果你需要使用[ES2015的特性](http://es6-features.org/)，你可能需要查阅一下信息来确保代码能在浏览器上正常运行 [Babel](https://babeljs.io/) 或 [Bublé](https://buble.surge.sh/guide/) 的转译器。

## webpack的配置

在实际项目中，很显然，在命令行直接赋予 CLI 命令参数的方式并不能满足我们的业务需求， 因此对于更复杂的配置，我们可以创建一个配置文件： `webpack.config.js`，`webpack` 根据配置信息来打包代码。创建一个配置文件后，你可以使用如下的配置设置来达到上述 CLI 命令的同样效果。

__webpack.config.js__

```javascript
var path = require('path');

module.exports = {
  entry: './app/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
};
```

可以使用下面的命令来运行指定配置文件的 `webpack` 命令

```bash
./node_modules/.bin/webpack --config webpack.config.js

Hash: ff6c1d39b26f89b3b7bb
Version: webpack 2.2.0
Time: 390ms
    Asset    Size  Chunks                    Chunk Names
bundle.js  544 kB       0  [emitted]  [big]  main
   [0] ./~/lodash/lodash.js 540 kB {0} [built]
   [1] (webpack)/buildin/global.js 509 bytes {0} [built]
   [2] (webpack)/buildin/module.js 517 bytes {0} [built]
   [3] ./app/index.js 278 bytes {0} [built]
```

T> 如果在当前的执行目录下有 `webpack.config.js` 文件，`webpack` 命令将默认使用当前执行目录下的这个文件，（如果没有指定配置文件）

T> 如果在上面 “创建一个bundle” 章节，已经成功创建过 `dist/bundle.js` 文件，可以删除 dist 子目录来验证通过 `webpack.config.js` 的设置所输出的内容是否符合预期。

通过使用配置文件的方式， webpack 更加灵活。可以向配置文件中添加 loader 、plugins、 resolve 等选项进行打包。

通过配置文件可以最灵活地使用 webpack。我们可以通过向配置文件添加 loader 规则(loader rules)、插件(plugins)、解析选项(resolve options)以及许多其他增强功能，来进行打包。

## 配合 npm 使用

在命令行中直接调用webpack并不是很方便，其实有种更快捷的方式：

修改 *package.json* 如下:

```json
{
  ...
  "scripts": {
    "build": "webpack"
  },
  ...
}
```

现在可以直接运行 `npm run build` 命令来执行文件打包。

T>  你可以通过向 `npm run build` 命令添加两个中横线， 给 `webpack` 传递自定义参数，例如：`npm run build -- --colors`。

You can now achieve the same as above by using `npm run build` command. npm picks up the scripts through it and patches the environment temporarily so that it contains the bin commands. You will see this convention in a lot of projects out there.

T> You can pass custom parameters to webpack by adding two dashes to the `npm run build` command, e.g. `npm run build -- --colors`.


## 小结

现在你已经学习完基本的构建过程，你可以深入 `webpack`  [基本概念](/concepts) 和 [配置信息](/configuration) 来更好地理解其设计。也可以查看 [指南](/guides) 来学习如何处理常见问题。[API](/api) 章节则是对底层的功能进行深入。

Now that you have a basic build together, you should dig into the [basic concepts](/concepts) and [configuration](/configuration) of webpack to better understand its design. Also check out the [guides](/guides) to learn how to approach common problems. The [API](/api) section digs into the lower level features.
