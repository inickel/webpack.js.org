---
title: 代码分离
sort: 30
contributors:
  - pksjce
  - pastelsky
  - simon04
---

`webpack` 中的 代码分离 是最引人注目的特性之一。 你可以把你的代码分离到不同的 `bundle` 中， 然后你就可以去按需加载这些文件。例如， 当用户跳转到匹配的路由，或用户触发了事件时，加载对应文件。 如果使用了正确的使用方式，这可以使我们有更小的 `bundle` ，同时可以控制资源加载优先级，从而优化应用程序的加载事件等。

总的来说，使用 `webpack` 的代码分离 主要有两种：

## 分离资源

### 分离第三方库

典型的应用程序中，通常会依赖许多第三方的框架或者函数库。不同于应用程序代码，而这些第三方库代码不会频繁修改。因此，无论业务代码如何变更，但第三方 `vendor` 的 `[hash]` 值通常是保持不变的.

如果我们将这些第三方库中的代码，打包在独立于应用程序代码的 `bundle` 中，就可以利用浏览器缓存机制，把这些文件长时间地缓存在用户端，以减少消耗网络请求。

查阅如何使用 `CommonsChunkPlugin` 来[分离第三方库](/guides/code-splitting-libraries) .

### 分离CSS

你也可能需要将你的css代码分离到单独的 `bundle` 中，与应用程序的逻辑分离。 使得css文件可独立，浏览器才能能够并行加载css文件，以避免[闪屏问题](https://en.wikipedia.org/wiki/Flash_of_unstyled_content)。

查阅如何使用 `ExtractTextWebpackPlugin` [分离css](/guides/code-splitting-css)

## 按需分离

前面提到的几种分离策略，开发者都需要预先在配置中指定分离的模块，在 `webpack` 中，也可以在应用程序代码中创建动态分离模块。

这可以用于更细粒度的代码块；例如，根据我们的应用程序路由，或根据用户行为预测。这可以使用户按照实际需要加载非必要资源。

查阅如何使用 `import()` 或 `require.ensure()` 执行[按需分离](/guides/code-splitting-async)

>T 有点啰嗦，其实无非是前端性能优化的几个点，具体参见雅虎军规
