title: Node.js 源码调试
author: prerabale
tags:
  - Node.js
categories:
  - JS
date: 2019-04-19 16:46:00
---
> 任何信息的价值都有时效性和适用性，本文写时 [Node.js](https://nodejs.org) 的最新发行版是 `v11.14.0`，稳定版本是 `v10.15.3`，文中出现的源码均来自 `tag`: `v11.14.0`。使用的电脑环境是：macOs 10.14.2。

## 前言

> Node.js is a JavaScript runtime built on Chrome's V8 JavaScript engine. 

[Node.js](https://nodejs.org) 是基于 [V8](https://github.com/v8/v8) 和 [libuv](https://github.com/libuv/libuv) 进行构建的，底层是以 `C/C++` 实现，而标准库部分则是采用 `JS` 编写。所以 Node.js 的源码调试分为两部分，`C/C++` 代码调试和 `JS` 代码调试。[更多...](https://yjhjstz.gitbooks.io/deep-into-node/chapter1/chapter1-0.html)

## 准备工作

1. Node.js [源码](https://github.com/nodejs/node)一份
2. [Visual Studio Code](https://code.visualstudio.com/) 或其它调试 `C/C++`、`JS` 的 `调试器/IDE`。

## 开始调试

### 编译

发行版本的 `Node` 是不支持调试的，所以我们需要自己通过源码构建一份可调试的 `Node`，`Node` 项目构建通过 `make` 进行管理，开发者们贴心的准备好了 `configure` 文件，所以构建一个自己定制版的 `Node` 非常方便。

> [官方构建指南](https://github.com/nodejs/node/blob/master/BUILDING.md)

#### 第一步：入口文件添加 debugger

首先，进入到下载下来的 Node.js 源码仓库目录（之后的操作都在这个目录进行）。  

修改 `JS` 源码入口文件(`./lib/internal/bootstrap/node.js`)文件中的内容，在头部加入 `debugger`。

```js
'use strict';
debugger; // <<--- 在这里加入 debugger;
// This file is compiled as if it's wrapped in a function with arguments
// passed by node::RunBootstrapping()
/* global process, require, internalBinding, isMainThread, ownsProcessState */
/* global primordials */

const { Object, Symbol } = primordials;
const config = internalBinding('config');
const { deprecate } = require('internal/util');

setupProcessObject();

setupGlobalProxy();
setupBuffer();
```

#### 第二步：执行编译

Node.js 使用 make 管理项目，开发者们准备了 configure 文件，我们只需要执行 `./configure` 就可生成当前环境可用的编译默认配置，然后执行 make 进行编译。但是默认的编译配置是没有启用调试模式的，因此，我们需要在执行 `./configure` 时加上 `--debug` 就可以生成可调试的编译配置项，然后再进行编译。

完整的命令如下：

```bash
#!/bin/bash

./configure --debug
make -C out  BUILDTYPE=Debug -j4

echo "showtime 🎉"
```
因为此后我们每次修改文件都需要重新编译，所以我这里把这些命令写到了 `build.sh` 里，之后的修改需要重新编译时，执行一次这个文件就可以了。

执行这个文件之前，需要先给这个文件一个可执行的权限：

```bash
chmod +x build.sh
```
完事具备，只需要在命令行输入 `./build.sh` 就可以开始编译了，然后你就可以去冲杯☕️了（coffee or tea? tea, pls）。  
编译之后的文件你可以在 `./out/Debug/` 目录下找到，里面的 `Node` 文件就是我们所需要的了。

### 调试 JS 源码

#### 第一步：准备一份测试文件

创建一个用于调试的项目/文件，里面随便写上一些什么，当然，你也可以用现成已经有的项目/文件。
为了方便，我就在 `Node.js` 的源码仓库目录下面创建了一个文件:

```js
console.log('hello world');
```

#### 第二步：启动服务

现在让我们用编译出来的 `Node` 执行这个文件：

```bash
./out/Debug/Node --inspect-brk=9229 test.js
```
看到如下提示就说明你的服务已经启用并处于可调试状态：

```bash
Debugger listening on ws://127.0.0.1:9229/0aeaa4ec-113b-4e08-94d9-0477c61d59ac
For help, see: https://nodejs.org/en/docs/inspector
```

#### 第三步：调试

接下来我们完成另外一半，用安装好的 `Visual Studio Code` 打开你的项目

点击左侧的蜘蛛标志，进入调试配置界面，然后点击右上角的配置按钮(打开目录下的 `.vscode/launch.json`)

> 注意：Visual Studio Code 需要打开一个项目时才能够创建配置文件

在配置文件中填入如下内容：

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "request": "attach",
            "name": "Attact Program",
            "port": 9229
        }
    ]
}
```
切记，记得进行保存。回到设置界面选择刚刚配置的 "Attact Program"，然后点击绿色的三角标开始，然后你就可以进入到 `Node.js` 源码中 `JS` 部分的调试了。

![](/images/vscode-debug-setting.png)

之后每次改动你的代码的时候都记得执行以下 `build.sh` 重新执行编译。

> 关于 vscode 中 JS 的更多调试姿势请看[这里](https://code.visualstudio.com/docs/editor/debugging)

### 调试 C/C++ 源码

#### 安装插件

[Visual Studio Code](https://code.visualstudio.com/) 默认是不支持 `C/C++` 调试的，需要安装对应的插件，打开 [Visual Studio Code](https://code.visualstudio.com/) 的插件商店，
搜索 `c++`，安装名为 `C/C++` 的官方插件

![](/images/vscode-c-plugin.jpg)

#### 调试

同 JS 部分一样，打开安装好的 `Visual Studio Code` 打开你的项目

点击左侧的蜘蛛标志，进入调试配置界面，然后点击右上角的配置按钮(打开目录下的 `.vscode/launch.json`)

> 注意：Visual Studio Code 需要打开一个目录才能够创建配置文件

在配置文件中填入如下内容：

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(lldb) Launch",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/out/Debug/Node",
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": true,
            "MIMode": "lldb"
        }
    ]
}
```
切记，记得进行保存。

![](/images/vscode-debug-setting.png)

然后用 IDE 在 `C/C++` 入口文件处 `./src/node_mian.cc` 打上 debugger 标识。
回到设置界面选择刚刚配置的 "Attact Program"，然后点击绿色的三角标开始，然后你就可以进入到 `Node.js` 源码中 `C/C++` 部分的调试了。

![](/images/vscode-c-debug.jpg)


## 参考

* [Building Node.js](https://github.com/nodejs/node/blob/master/BUILDING.md)
* [Node.js源码学习(1) 使用cLion调试node.js源码
](https://juejin.im/post/5a5cc0f4518825734216e166)
* [node源码粗读（2）：node编译过程详解及如何在本地进行源码修改和调试](https://github.com/xtx1130/blog/issues/9)
* [node源码粗读（5）：通过调试./lib库的js代码来看javascript在node中运行环境的变化](https://github.com/xtx1130/blog/issues/14)
* [深入理解Node.js：核心思想与源码分析](https://yjhjstz.gitbooks.io/deep-into-node/chapter1/chapter1-0.html)
* [使用 Visual Studio Code 搭建 C/C++ 开发和调试环境](https://zhuanlan.zhihu.com/p/36654741)
* [C++ programming with Visual Studio Code](https://code.visualstudio.com/docs/languages/cpp)