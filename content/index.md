---
title: webpack
---

## 示例代码

<div class="homepage__wrap">
<div class="homepage__left">

**app.js**

```js
import bar from './bar';

bar();
```

</div><div class="homepage__right">

**bar.js**

```js
export default function bar() {
  //
}
```

</div>
</div>


## webpack打包代码

<div class="homepage__wrap">
<div class="homepage__left">

**webpack.config.js**

```js
module.exports = {
  entry: './app.js',
  output: {
    filename: 'bundle.js'
  }
}
```

</div><div class="homepage__right">

**page.html**

```html
<html>
  <head>
    ...
  </head>
  <body>
    ...
    <script src="bundle.js"></script>
  </body>
</html>
```

执行 `webpack` 命令，将会自动生成 `bundle.js` 文件

</div>
</div>

## [入门](/guides/get-started)
