---
title: 代码分离 - 第三方库
sort: 32
contributors:
  - pksjce
  - chrisVillanueva
  - johnstew
  - rafde
  - bartushek
  - shaunwallace
---

一个典型的应用程序会使用第三方框架、类库。这些库一般使用特定的稳定版本，稳定版本自然变更频次比较少；业务代码会频繁改变。

把第三方代码和应用本身的代码一起构建是非常low的做法。 最主要的是不能有效的使用 http 缓存。 其次是放到同一个bundle文件中，使其文件体积增加，增加网络开销，对于用户体验来说是非常致命的。

只有当我们把 vendor 和应用代码的 bundle 分离时，才能实现这一点。

举例说明：

比如我们业务中使用了 [momentjs](https://www.npmjs.com/package/moment)。

在项目根目录安装此模块：

``` bash
npm install --save moment
# 纯前端使用此模块，建议是 --save-dev，如果你用 nodejs 做web服务的时候用到模块时用 --save
```
修改index.js 依赖 moment

__index.js__

```javascript
var moment = require('moment');
console.log(moment().format());
```

webpack 构建配置

__webpack.config.js__

```javascript
var path = require('path');

module.exports = function(env) {
    return {
        entry: './index.js',
        output: {
            filename: '[name].[chunkhash].js',
            path: path.resolve(__dirname, 'dist')
        }
    }
}
```

执行 `webpack` 构建后， 查看输出的 `bundle` 的代码， `moment` 和 `index.js` 都被构建到 `bundle.js` 文件中。
但是如果 `index.js` 发生变更，则整个 `bundle` 都需要重新构建， 这样会导致浏览器需要每次都重新加载来非常多并没有变更过的代码。

## 配置多个入口

稍微修改下webpack的配置文件：

__webpack.config.js__

``` javascript
var path = require('path');

module.exports = function(env) {
    return {
        entry: {
            main: './index.js',
            vendor: 'moment'
        },
        output: {
            filename: '[name].[chunkhash].js',
            path: path.resolve(__dirname, 'dist')
        }
    }
}
```

执行完 `webpack` 命令后， 会产生新的两个 `bundle` 文件， 查看源码后发现 `moment` 的代码在两个 `bundle` 文件中都存在，原因是 moment 被 index.js依赖， 同时每个入口都会将自己的依赖模块构建起来。

为此， 官方提供 [CommonsChunkPlugin](/plugins/commons-chunk-plugin) 来处理公共模块的抽取工作

## 公共模块插件

这插件非常复杂， 它允许我们从不同的 `bundle` 中提取所有的公共模块， 并将他们加入公共 `bundle` 中。

修改 `webpack` 配置文件如下:

__webpack.config.js__

```javascript
var webpack = require('webpack');
var path = require('path');

module.exports = function(env) {
    return {
        entry: {
            main: './index.js',
            vendor: 'moment'
        },
        output: {
            filename: '[name].[chunkhash].js',
            path: path.resolve(__dirname, 'dist')
        },
        plugins: [
            new webpack.optimize.CommonsChunkPlugin({
                name: 'vendor' // 声明公共模块名称
            })
        ]
    }
}
```

重新执行 `webpack` ， 查看构建后的代码， 发现 `moment` 的代码只在 `vendor` 的 文件中存在， 这样就达到来我们的预期效果。

```html
···
<script src="vendor.[chunkhash].js"></script>
<script src="main.[chunkhash].js"></script>
···
```

## 隐式公共模块

你可以将 `CommonsChunkPlugin` 配置为只接受第三方库。

__webpack.config.js__

```javascript
var webpack = require('webpack');
var path = require('path');

module.exports = function() {
    return {
        entry: {
            main: './index.js'
        },
        output: {
            filename: '[name].[chunkhash].js',
            path: path.resolve(__dirname, 'dist')
        },
        plugins: [
            new webpack.optimize.CommonsChunkPlugin({
                name: 'vendor',
                minChunks: function (module) {
                  // 引入的 vendor 存在于 node_modules 目录中
                   return module.context && module.context.indexOf('node_modules') !== -1;
                }
            })
        ]
    };
}
```

__index.html__

```html
···
<script src="vendor.[chunkhash].js"></script>
<script src="main.[chunkhash].js"></script>
···
```

## Manifest 文件

新的问题又来来： 每次构建的时候， `vendor` 的 `hash` 都会改变， 即使把 `vendor` 和 `main` 的 `bundle` 隔离，但 `vendor bundle` 也会随着业务代码的变更而变更。 因为每次构建都会变更 vendor 的 hash 值，这使得我们无法有效的利用浏览器的缓存机制。

这里的问题在于，每次构建时， `webpack` 生成了一些 `webpack runtime` 代码，用来帮助 `webpack` 完成其工作。当只有一个 `bundle` 的时候， `runtime` 代码驻留在其中。但是当生成多个 `bundle` 的时候，运行时代码被提取到了公共模块中，在这里就是 `vendor` 文件。

为了防止这种情况，我们需要将运行时代码提取到一个单独的 `manifest` 文件中。 尽管我们又创建了另一个 `bundle` ，但可以是的第三方代码库可以有效的利用浏览器缓存。

__webpack.config.js__

```javascript
var webpack = require('webpack');
var path = require('path');

module.exports = function(env) {
    return {
        entry: {
            main: './index.js',
            vendor: 'moment'
        },
        output: {
            filename: '[name].[chunkhash].js',
            path: path.resolve(__dirname, 'dist')
        },
        plugins: [
            new webpack.optimize.CommonsChunkPlugin({
                names: ['vendor', 'manifest'] // Specify the common bundle's name.
            })
        ]
    }
};
```

重新打包后，生成了`vendor` 、 `main` 和 `manifest` 三个bundle文件。

至此，已经达成隐式公共模块抽取的任务。

__webpack.config.js__

```javascript
var webpack = require('webpack');
var path = require('path');

module.exports = function() {
    return {
        entry: {
            main: './index.js' //Notice that we do not have an explicit vendor entry here
        },
        output: {
            filename: '[name].[chunkhash].js',
            path: path.resolve(__dirname, 'dist')
        },
        plugins: [
            new webpack.optimize.CommonsChunkPlugin({
                name: 'vendor',
                minChunks: function (module) {
                   // this assumes your vendor imports exist in the node_modules directory
                   return module.context && module.context.indexOf('node_modules') !== -1;
                }
            }),
            //CommonChunksPlugin will now extract all the common modules from vendor and main bundles
            new webpack.optimize.CommonsChunkPlugin({
                name: 'manifest' //But since there are no more common modules between them we end up with just the runtime code included in the manifest file
            })
        ]
    };
}
```

__index.html__

```html
···
<!-- manifest 和 main 的 chunkhash 在每一次发生业务代码变动的时候，都会发生变更 -->
<script src="manifest.[chunkhash].js"></script>
<!-- vendor 的 chunkhash 如果不更新第三方资源库的情况下，不发生变更 -->
<script src="vendor.[chunkhash].js"></script>
<script src="main.[chunkhash].js"></script>
···
```

T> 注意，长效的 bundle 缓存是通过 “content-based hashing” 来实现的 。查阅[缓存](/guides/caching/)。
