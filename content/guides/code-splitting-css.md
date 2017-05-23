---
title: 代码分离 - CSS
sort: 31
contributors:
  - pksjce
  - jonwheeler
  - johnstew
  - simon04
  - shinxi
  - tomtasche
---

如果想用 webpack 打包css文件， 需要使用[其它类型模块](/concepts/modules)来引用css文件，并且使用 `css-loader` 加载器来输出处css模块，利用`ExtractTextWebpackPlugin` 外部文本处理插件来处理打包和输出css文件。

## 引入CSS

css的引入方式 和 JavaScript模块的引入方式一样，

```javascript
import 'bootstrap/dist/css/bootstrap.css';
```


## css和style加载器


安装 [`css-loader`](/loaders/css-loader) 和 [`style-loader`](/loaders/style-loader):

``` bash
npm install --save-dev css-loader style-loader
```

给webpack配置文件添加loader配置

```javascript
module.exports = {
    module: {
        rules: [{
            test: /\.css$/,
            use: [ 'style-loader', 'css-loader' ]
        }]
    }
}
```

在打包完成后，css代码被加入到JavaScript的代码中，并在页面完成加载后，插入`<style>`标签。

这样会出现闪屏问题：页面的样式渲染依赖于JavaScript的加载，显然这不符合我们前端优化的准则，因此我们需要把样式文件和javascript文件分开加载。

webpack 提供一个外部文本处理插件 [`ExtractTextWebpackPlugin`](/plugins/extract-text-webpack-plugin) 来帮我们解决此问题

## 外部文本处理插件

安装 [`ExtractTextWebpackPlugin`](/plugins/extract-text-webpack-plugin) 插件

``` bash
npm install --save-dev extract-text-webpack-plugin
```

修改 `webpack` 配置的 `module.rules` 和 添加 `plugins`

```diff
+var ExtractTextPlugin = require('extract-text-webpack-plugin');
module.exports = {
    module: {
         rules: [{
             test: /\.css$/,
-            use: [ 'style-loader', 'css-loader' ]
+            use: ExtractTextPlugin.extract({
+                use: 'css-loader'
+            })
         }]
     },
+    plugins: [
+        new ExtractTextPlugin('styles.css'),
+    ]
}
```
经过修改后打包，所有的css模块都被导入到styles.css文件中，在 `index.html` 中可以直接引用该css文件。
