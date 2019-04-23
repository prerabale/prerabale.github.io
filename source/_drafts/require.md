title: require
author: prerabale
tags:
  - Node.js
categories:
  - JS
  - ''
date: 2019-03-29 23:51:00
---
## require

> 大纲
* require 的简介
* require 为什么这么设计，AMD，CMD，UMD，commonJS，ES6
* require 的缓存设计
* require 的循环引用问题
* require 全局函数，require 怎么实现
* require.extension(废弃但不会移除), require.cache, require.resolve, require.resolve.paths,实现一个可以注释JSON的require
* require 实现热更新，弊端或限制
* require('lodash/get') 引用 lodash 的一个方法？
* 在文件中不用申明语句申明一个变量 name = 'luxi'

### require 简介

`require` 是 `Node.js` 中内置的一个全局函数，用于引用指定的模块，得到模块的输出结果。

![摘录自](/images/require_module.png)

`require` 引用的模块可以是：
* [npm](https://www.npmjs.com/) 上的一个模块
* 原生模块(Node.js 自带的模块)
* 目录/文件


```js Module#require https://nodejs.org/api/modules.html#modules_require_id 
// Importing a local module:
const myLocalModule = require('./path/myLocalModule');

// Importing a JSON file:
const jsonData = require('./path/filename.json');

// Importing a module from node_modules or Node.js built-in module:
const crypto = require('crypto');
```

`require` 此时会根据当前`环境`遵循特定的`寻径规则`对`支持的文件类型`进行`特定的解析`并输出内容。

上面这一段话看着挺绕口的，但其实最关键的就是几个关键词

#### 环境

这里环境指的是使用 `require` 函数的模块所在的位置，
例如：
```js
// a.js 位于 /Users/prerabale/a.js
require('xxx')
```
此时 `a.js` 的环境就是 `/Users/prerabale` 目录，根据目录的变化，`require` 查找的路径`可能`会有所不同，引用的就`可能`是不同的模块(请注意上面的两个`可能`)。

例如:

我们有如下一个目录结构
```sh
├── a.js
├── b.js
└── lib
    ├── a.js
    └── b.js

```

他们的内容分别如下：

```js
// b.js
exports.sayHi = function() {
  console.log("I'm b");
}
// lib/b.js
exports.sayHi = function() {
  console.log("I'm lib/b")
}
// a.js
const b = require('./b'); // 此处可以省略 .js 后缀
b.sayHi(); // > I'm b

// lib/a.js
const b = require('./b'); // 此处可以省略 .js 后缀
b.sayHi(); // > I'm lib/b
```

在上面的例子中 `a.js` 和 `lib/a.js` 的内容完全一样，不同的只是文件所在的目录，但他们却引用到了不同的 `b.js`(这很好理解，对吧)。

至于我为什么说有两个`可能`，最简单的例子就是 `require("path")`，但不仅限于此。

#### 寻径规则



#### 支持的文件类型

#### 特定的解析

## 参考

> 所以对于未来来说第三方库要同时发布 commonjs 格式和 es6 格式的模块。es6 模块的入口由 package.json 的字段 module 指定。而 commonjs 则还是在 main 字段指定。

* [require() 源码解读](http://www.ruanyifeng.com/blog/2015/05/require.html)
* [import、require、export、module.exports 混合使用详解 - 掘金](https://juejin.im/post/5a2e5f0851882575d42f5609)