---
title: 模块规范
layout: post
categories: javascript
tags: 基础
published: false
---

### [CommonJS](http://wiki.commonjs.org/wiki/CommonJS)

服务器端Node.js一直使用着的规范，核心思想是模块通过`exports`、`module.exports`导出，使用`require`方法同步进行加载。

```js
const http = require( 'http' );
const pkgName = require( './package.json' ).name;
```

为什么不用异步？异步加载会导致流程控制变得困难，代码书写和阅读也会因此变得复杂。有些时候你并不希望你的文件被多次`require`引用时所有的代码都被重复执行的（例如：数据库连接）。   

同步加载完成之后，在全局的`require`缓存中将此模块的导出结果缓存，之后引用这部分模块的地方会直接使用缓存。即可以减少开销，又可以避免不希望被多次执行的代码在每次`require`时都被执行。   
服务端的资源终究都是要读取至内存中的，如果耗费大的代价使用异步只是为了加快服务器的启动速度是得不偿失的。
在浏览器中情况又大有不同了。为了更好的用户体验，更快速的响应，不被立即使用的资源是不希望被立即加载的，并且同步就意味着阻塞，显然同步的模块加载方式并不适用于浏览器环境中。

### [AMD](https://github.com/amdjs/amdjs-api)

[Asynchronous Module Definition](https://github.com/amdjs/amdjs-api) 规范其实只有一个主要接口 define(id?, dependencies?, factory)，它要在声明模块的时候指定所有的依赖 dependencies，并且还要当做形参传到 factory 中，对于依赖的模块提前执行，依赖前置，例如：`RequireJS`。

```
define("module", ["dep1", "dep2"], function(d1, d2) {
  return someExportedValue;
});
require(["module", "../file"], function(module, file) { /* ... */ });
```

优点：  
适合在浏览器环境中异步加载模块
可以并行加载多个模块  

缺点：  
提高了开发成本，代码的阅读和书写比较困难，模块定义方式的语义不顺畅
不符合通用的模块化思维方式，是一种妥协的实现。

### [CMD](https://github.com/cmdjs/specification/blob/master/draft/module.md)

[Common Module Definition](https://github.com/cmdjs/specification/blob/master/draft/module.md) 规范和 AMD 很相似，尽量保持简单，并与 CommonJS 和 Node.js 的 Modules 规范保持了很大的兼容性。

```
define(function(require, exports, module) {
  var $ = require('jquery');
  var Spinning = require('./spinning');
  exports.doSomething = ...
  module.exports = ...
})
```