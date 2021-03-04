title: Set up project with ReactJS, webpack, and ES2015
archive: Study
date: 2016-05-23 16:41:33
category: React
---

这篇博客以一个简单的小程序来演示用 ReactJS + ES6 + webpack 搭建一个hello world 应用的过程。

从[官方文档](https://facebook.github.io/react/docs/reusable-components.html#es6-classes)中可知，ReactJS支持使用ES6。

本次就用ES6来编写ReactJS前端代码，并用webpack进行模块管理。

目标：创建两个组件，分别为`hello.js` 和 `world.js`，然后调用这两个组件，在页面上进行渲染。

### 创建项目文件夹

```bash
mkdir simple-react-webpack
cd simple-react-webpack
```
<!-- more -->

### 安装npm包

react react-dom：react相关包
babel-core babel-loader babel-preset-es2015 babel-preset-react：babel解析es6和react的相关包
webpack webpack-dev-server：webpack的相关包


```bash
npm install react react-dom babel-core babel-loader babel-preset-es2015 babel-preset-react  webpack webpack-dev-server --save
```

webpack安装完之后，会在项目根目录下生成一个`webpack.config.js`配置文件，会在后面用到

### 创建组件`Hello`和`World`

```bash
touch hello.js
touch world.js
```

`hello.js`

```js
import React from 'react'
import ReactDOM from 'react-dom'

class Hello extends React.Component {
  render() {
    return (
        <div>Hello</div>
      )
  }
}
ReactDOM.render(<Hello />, document.getElementById('hello'))
```

`world.js`
```js
import React from 'react'
import ReactDOM from 'react-dom'

class World extends React.Component {
  render() {
    return (
        <div>world</div>
      )
  }
}
ReactDOM.render(<World />, document.getElementById('world'))
```

### 创建webpack编译的入口文件

webpack中需要指定编译入口文件，我们在这个入口文件中引入组件`Hello`和`World`。

```bash
touch main.js
```
`main.js`

```js
import Hello from './hello.js'
import World from './world.js'
```

### 配置webpack.config.js

```js
var path = require('path');
var webpack = require('webpack');

module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js',
    path: __dirname
  },
  module: {
    loaders: [
      {
        test: /.js?$/,
        loader: 'babel-loader',
        exclude: /node_modules/,
        query: {
          presets: ['es2015', 'react']
        }
      }
    ]
  }
}
```
这个配置定义了webpack的编译规则。

entry: webpack进行编译时的入口，详见[文档](http://webpack.github.io/docs/configuration.html#entry)。
output: webpack将组件编译打包后,文件的命名及位置，本例中指定为当前目录下`bundle.js`文件。
loaders: 此处只加载了一个loader，意为：除了`/node_modules/`文件夹之外的，后缀名为`.js`的类型文件，全部用babel-loader编译，并在编译时，加载es2015和react插件（详见babel [options](http://babeljs.io/docs/usage/options/))。loader的详细参数见[文档](http://webpack.github.io/docs/loaders.html#loaders-by-config)

### 新建页面并载入已经编译好的bundle.js

```bash
touch index.html
```

`index.html`

```js
<!DOCTYPE html>
<html>
<head>
  <title>Simple webpack react demo</title>
</head>
<body>
  <div id="hello"></div>
  <div id="world"></div>
  <script src="./bundle.js"></script>
</body>
</html>
```

### 运行

```
webpack-dev-server --progress --colors
```
我们用`webpack-dev-server`来启动服务，默认8080端口。
在浏览器中访问：[http://localhost:8080/webpack-dev-server/]( http://localhost:8080/webpack-dev-server/)
关于`webpack-dev-server`, 详见[webpack-dev-server](http://webpack.github.io/docs/webpack-dev-server.html)

博客中的代码我放在了[github](https://github.com/hanpanpan200/simple-react-webpack)上供参考。
