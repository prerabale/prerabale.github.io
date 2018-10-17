---
title: 使用webpack打包react项目
layout: post
categories: javascript
tags: 基础
published: false
---

## react

* main.js

```
var React = require('react');
var ReactDOM = require('react-dom');

ReactDOM.render(
	<h1>Hello, world!</h1>,
	document.getElementById('example')
);

```

> index.html

```
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>React</title>
</head>
<body>

<div id="example"></div>
<script src="./build/bundle.js" type="text/javascript"></script>
</body>
</html>
```

> webpack.config.js

```
const webpack = require( 'webpack' );
const path = require( 'path' );

module.exports = {
	entry: path.resolve( __dirname, './src/main.js' ), // 指向main.js
	output: {
		path: path.resolve( __dirname, './build' ),
		filename: 'bundle.js'
	},
	module: {
		rules: [
			{
				test: /\.js$/,
				exclude: /(node_modules|bower_components)/,
				use: {
					loader: 'babel-loader',
					options: {
						presets: ['react']
					}
				}
			}
		]
	}
};
```