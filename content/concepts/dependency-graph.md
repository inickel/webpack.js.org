---
title: 依赖关系
sort: 9
contributors:
  - TheLarkInn
---

每当一个模块依赖于另外一个模块时， webpack 认为两者就产生 _dependency_。

Any time one file depends on another, webpack treats this as a _dependency_. This allows webpack to take non-code assets, such as images or web fonts, and also provide them as _dependencies_ for your application.

webpack在打包项目代码时，先根据参数（或配置）标记出被定义的模块列表。
从 _entry points_, 开始解析并递归出每个模块所涉及到的依赖关系图谱， 然后在打包到 bundle 文件中。

When webpack processes your application, it starts from a list of modules defined on the command line or in its config file.
Starting from these _entry points_, webpack recursively builds a _dependency graph_ that includes every module your application needs, then packages all of those modules into a small number of _bundles_ - often, just one - to be loaded by the browser.

T> Bundling your application is especially powerful for *HTTP/1.1* clients, as it minimizes the number of times your app has to wait while the browser starts a new request. For *HTTP/2*, you can also use Code Splitting and bundling through webpack for the [best optimization](https://medium.com/webpack/webpack-http-2-7083ec3f3ce6#.7y5d3hz59).
