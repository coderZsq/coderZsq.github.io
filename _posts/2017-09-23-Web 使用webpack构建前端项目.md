---
layout: post
title: Web 使用webpack构建前端项目
date: 2017.09.23 15:46
tag: 前端开发
---

> 好久没写技术博客了, 原因在于最近在学习前端方面的技术, 熟悉我的同学都知道, 之前我有使用Vue搭建了一个个人简历, 体验了一把最新的前端技术, 但之前我们使用的是vue-cli脚手架工具, 对于如何自己实现前端构建工具, 当下最为流行的就是webpack和gulp了, 之前一篇我们讲了gulp, 这一篇我们来好好讨论webpack.

![](http://upload-images.jianshu.io/upload_images/1229762-e8ed33b306d59976.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


说起webpack, 想必做前端的同学肯定不会陌生, 其实之前我们使用gulp构建的时候, 也使用了webpack的打包技术, 其实gulp和webpack并不是相互替代的关系, 而是相辅相成, 今天我们就来好好看看webpack的神奇之处吧.

我们学习一样新技术, 首先肯定是从他的[官方文档](https://webpack.js.org/guides/getting-started/)入手, 当然我们要学习也是学最新版的.webpack的官方教程写的非常好, 一步一步讲的很到位, 各位同学可以直接阅读官方文档, 比起博客中的二手, 三手以及四手的资料, 官方文档肯定是你更好的选择.

这篇文章, 不是教你什么看这一篇就够了之类的对于官方文档拷贝的水文, 而是能让你快速上手并且觉得所谓的webpack其实也就这么一回事, webpack你只要记住一个中心思想, 就和上面的图示一样, 将所有错综复杂的文件逻辑打包压缩成几个静态资源, 不多说了, 我们还是看代码来的实际.

#### webpack.config.js

对于一些抛弃jquery迎接react和vue的前端开发者来说, webpack虽然可能自己没有写过, 但看总是看过的吧, 一般来说, 都会有一个`webpack.config.js`的webpack配置文件.下面的代码就是一个简单的webpack的配置, 麻雀虽小五脏俱全.

```
var debug = process.env.NODE_ENV !== "production"; //是否是测试环境
var webpack = require('webpack'); //导入webpack包
var path = require('path');

module.exports = { //导出 webpack固定写法
  context: path.join(__dirname),
  devtool: debug ? "inline-sourcemap" : null, //是否使用map工具, 用于浏览器debug
  entry: "./src/js/root.js", //打包的实体
  module: {
    loaders: [ //加载的配置
      {
        test: /\.js?$/,
        exclude: /(node_modules)/,
        loader: 'babel-loader',
        query: {
          presets: ['react', 'es2015'], //添加预处理器
          plugins: ['react-html-attrs'], //添加组件的插件配置
        }
      },
      { test: /\.css$/, loader: 'style-loader!css-loader' },
      {
        test: /\.less$/,
        loader: "style!css!less"
      }
    ]
  },
  output: { //输出的路径及文件名
    path: __dirname,
    filename: "./src/bundle.js"
  },
  plugins: debug ? [] : [ //一些插件
    new webpack.optimize.DedupePlugin(),
    new webpack.optimize.OccurenceOrderPlugin(),
    new webpack.optimize.UglifyJsPlugin({ mangle: false, sourcemap: false }),
  ],
};

```

webpack主要包括`entry `, `module `, `output `, `plugins `四大类, 官方文档说的已经很清楚了, 想要进一步的学习,请翻阅官方文档, 如果不想折腾直接拷贝上述代码即可.

相较gulp, webpack在打包方面更为精简, 这也是流行的原因吧, 但光看上面的文件, 的确也是简单, 但是还有进一步改善的空间.

#### package.json

对于npm的介绍我就不多说了, 我们直接来看文件.

```
{
  "name": "webpack",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": { //命令行工具
    "test": "echo \"Error: no test specified\" && exit 1",
    "watch": "webpack --progress --watch",
    "start": "webpack-dev-server --open --config webpack.dev.js",
    "build": "webpack --config webpack.prod.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": { //开发环境依赖
    "babel-loader": "^7.1.2",
    "babel-plugin-import": "^1.5.0",
    "babel-plugin-react-html-attrs": "^2.0.0",
    "babel-preset-es2015": "^6.24.1",
    "babel-preset-react": "^6.24.1",
    "babelify": "^7.3.0",
    "clean-webpack-plugin": "^0.1.16",
    "css-loader": "^0.28.7",
    "csv-loader": "^2.1.1",
    "file-loader": "^0.11.2",
    "html-webpack-plugin": "^2.30.1",
    "json-loader": "^0.5.7",
    "lodash": "^4.17.4",
    "style-loader": "^0.18.2",
    "uglifyjs-webpack-plugin": "^0.4.6",
    "webpack": "^3.6.0",
    "webpack-dev-middleware": "^1.12.0",
    "webpack-dev-server": "^2.8.2",
    "webpack-merge": "^4.1.0",
    "xml-loader": "^1.2.1"
  },
  "dependencies": { //生产环境依赖
    "react": "^15.6.1",
    "react-dom": "^15.6.1",
    "react-mixin": "^4.0.0",
    "react-router": "^4.2.0"
  }
}

```

命令行工具就是`npm run build`等于执行了`webpack --config webpack.prod.js`, 而`npm start` 等于执行了`webpack-dev-server --open --config webpack.dev.js`.简单易懂吧.

在项目依赖中, 哦们加了很多的插件和loader, 都是用来搭建webpack的, 官方文档的教程中都会讲到, 值得注意的就是`webpack-merge`这个包, 这个包可以让我们生产环境和开发环境很好的隔离配置, 我们看看怎么做呢?

首先我们需要将之前的`webpack.config.js`分成三个文件 --- `webpack.common.js`, `webpack.dev.js`, `webpack.prod.js`.

#### webpack.common.js

这个是webpack的共同配置, 总体和之前看到的大同小异, 我们主要是导入了两个插件, 一个是清除插件, 一个是创建html的插件.

```
const path = require('path');
const webpack = require('webpack');
const CleanWebpackPlugin = require('clean-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = {
  entry: './src/index.js',
  plugins: [
    new CleanWebpackPlugin(['dist']),
    new HtmlWebpackPlugin({title: 'webpack'}),
    new webpack.HashedModuleIdsPlugin()
  ],
  output: {
    filename: '[name].[chunkhash].js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.js?$/,
        exclude: /(node_modules)/,
        loader: 'babel-loader',
        query: {
          presets: [
            'react', 'es2015'
          ],
          plugins: ['react-html-attrs']
        }
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      }, {
        test: /\.(png|svg|jpg|gif)$/,
        use: ['file-loader']
      }, {
        test: /\.(woff|woff2|eot|ttf|otf)$/,
        use: ['file-loader']
      }, {
        test: /\.(csv|tsv)$/,
        use: ['csv-loader']
      }, {
        test: /\.xml$/,
        use: ['xml-loader']
      }
    ]
  }
};

```

`rules`配置中我们也就是将一些可能用到的文件也配置到webpack中来, `babel-loader`这种如果要讲还可以再开一篇, 其实就是个js的兼容性工具, 这样理解就可以了.

#### webpack.dev.js

webpack开发环境的配置, 非常简单, 就是用了之前讲的`webpack-merge`工具, 就和git一样, 合并了`webpack.common.js`的配置外新加了可以进行调试的`inline-source-map`工具, 以及热更新的内容索引.

```
const merge = require('webpack-merge');
const common = require('./webpack.common.js');

module.exports = merge(common, {
  devtool: 'inline-source-map',
  devServer: {
    contentBase: './dist'
  }
});
```

#### webpack.prod.js

webpack生产环境的配置, 新加了一个压缩插件以及环境配置的插件, 这里的开发工具和开发还款下的有所不同, 具体可直接看官方文档.

```
const webpack = require('webpack');
const merge = require('webpack-merge');
const UglifyJSPlugin = require('uglifyjs-webpack-plugin');
const common = require('./webpack.common.js');

module.exports = merge(common, {
  devtool: 'source-map',
  plugins: [
    new UglifyJSPlugin({sourceMap: true}),
    new webpack.DefinePlugin({
      'process.env': {
        'NODE_ENV': JSON.stringify('production')
      }
    })
  ]
});

```

#### terminal

这样我们就配置完成啦, 我们在终端上输入看下效果:

```
cd ../ && npm i 
```
首先我们进入到目录下并进行node包的安装.

##### npm run build

```
MacBook-Pro-15:webpack zhushuangquan$ npm run build

> webpack@1.0.0 build /Users/zhushuangquan/Documents/code/webpack
> webpack --config webpack.prod.js

clean-webpack-plugin: /Users/zhushuangquan/Documents/code/webpack/dist has been removed.
Hash: 85b65f54ef1436b295a5
Version: webpack 3.6.0
Time: 1148ms
                           Asset       Size  Chunks             Chunk Names
    main.014ac9aa420264da48eb.js  671 bytes       0  [emitted]  main
main.014ac9aa420264da48eb.js.map    6.47 kB       0  [emitted]  main
                      index.html  197 bytes          [emitted]  
[lVK7] ./src/index.js 184 bytes {0} [built]
Child html-webpack-plugin for "index.html":
     1 asset
    [3IRH] (webpack)/buildin/module.js 517 bytes {0} [built]
    [DuR2] (webpack)/buildin/global.js 509 bytes {0} [built]
        + 2 hidden modules
```

我们可以看到已经打包好的文件:

###### main.014ac9aa420264da48eb.js

```
!function(e){function n(r){if(t[r])return t[r].exports;var o=t[r]={i:r,l:!1,exports:{}};return e[r].call(o.exports,o,o.exports,n),o.l=!0,o.exports}var t={};n.m=e,n.c=t,n.d=function(e,t,r){n.o(e,t)||Object.defineProperty(e,t,{configurable:!1,enumerable:!0,get:r})},n.n=function(e){var t=e&&e.__esModule?function(){return e.default}:function(){return e};return n.d(t,"a",t),t},n.o=function(e,n){return Object.prototype.hasOwnProperty.call(e,n)},n.p="",n(n.s="lVK7")}({lVK7:function(e,n,t){"use strict";document.body.appendChild(function(){var e=document.createElement("div");return e.innerHTML="Hello webpack",e}())}});
//# sourceMappingURL=main.014ac9aa420264da48eb.js.map
```

我们可以看到在webpack的打包和压缩下, 代码已经基本不可读了. 所以我们需要加上之前的调试插件, 以便生产环境出现bug后的补救.

##### npm start

```
MacBook-Pro-15:webpack zhushuangquan$ npm start

> webpack@1.0.0 start /Users/zhushuangquan/Documents/code/webpack
> webpack-dev-server --open --config webpack.dev.js

clean-webpack-plugin: /Users/zhushuangquan/Documents/code/webpack/dist has been removed.
Project is running at http://localhost:8080/
webpack output is served from /
Content not from webpack is served from ./dist
webpack: wait until bundle finished: /
Hash: 06f20ec519d58fbd5c28
Version: webpack 3.6.0
Time: 1460ms
                       Asset       Size  Chunks                    Chunk Names
main.5eb4d4e3f458c49658a2.js     852 kB       0  [emitted]  [big]  main
                  index.html  197 bytes          [emitted]         
[6Um2] (webpack)/node_modules/url/util.js 314 bytes {0} [built]
[8o/D] (webpack)-dev-server/client/overlay.js 3.71 kB {0} [built]
[HPf+] (webpack)/node_modules/url/url.js 23.3 kB {0} [built]
[Lx3u] (webpack)/hot/log.js 1.04 kB {0} [optional] [built]
[Sj28] (webpack)-dev-server/node_modules/strip-ansi/index.js 161 bytes {0} [built]
[TfA6] (webpack)/hot nonrecursive ^\.\/log$ 170 bytes {0} [built]
[U2me] (webpack)/hot/emitter.js 77 bytes {0} [built]
[V3KU] (webpack)-dev-server/client/socket.js 1.04 kB {0} [built]
[cMmS] (webpack)-dev-server/client?http://localhost:8080 7.27 kB {0} [built]
[gqsi] (webpack)-dev-server/node_modules/loglevel/lib/loglevel.js 7.74 kB {0} [built]
   [0] multi (webpack)-dev-server/client?http://localhost:8080 ./src/index.js 40 bytes {0} [built]
[gt+Q] (webpack)-dev-server/node_modules/ansi-regex/index.js 135 bytes {0} [built]
[lVK7] ./src/index.js 184 bytes {0} [built]
[p7Vd] (webpack)/node_modules/punycode/punycode.js 14.7 kB {0} [built]
[pEPF] (webpack)/node_modules/querystring-es3/index.js 127 bytes {0} [built]
    + 73 hidden modules
Child html-webpack-plugin for "index.html":
     1 asset
    [3IRH] (webpack)/buildin/module.js 517 bytes {0} [built]
    [DuR2] (webpack)/buildin/global.js 509 bytes {0} [built]
    [M4fF] ./node_modules/lodash/lodash.js 540 kB {0} [built]
    [a/t9] ./node_modules/html-webpack-plugin/lib/loader.js!./node_modules/html-webpack-plugin/default_index.ejs 538 bytes {0} [built]
webpack: Compiled successfully.

```
我们可以看到打开了一个内容为`Hello webpack`的网页在8080端口, 当我们修改了文件时候网页会自动刷新.

#### 知识点:

回到我们刚才的package.json的命令行配置来看.

```
  "scripts": { //命令行工具
    "test": "echo \"Error: no test specified\" && exit 1",
    "watch": "webpack --progress --watch",
    "start": "webpack-dev-server --open --config webpack.dev.js",
    "build": "webpack --config webpack.prod.js"
  },
```

- 上面的`npm run build` => `webpack` => `webpack.prod.js`, 就是执行了生产环境的配置的打包命令. 

- 上面的`npm start` => `webpack-dev-server --open` => `webpack.dev.js`, 就是执行了开发环境配置的服务端命令.


- `--config`是用于执行webpack配置文件的命令, 而默认为`webpack.config.js`.
- `webpack`命令就是和之前的gulp的逻辑相似, 将`entry`实例复制到`output`路径的逻辑. 当然还伴随着一系列的操作.
- `webpack-dev-server --open`命令是打开服务器并进行热加载的用途.

以上就是webpack的使用及逻辑, 并没有想象中的复杂吧, 甚至可以说是简单, 实测一天即可入门webpack.

想要了解webpack进阶, 提高性能的同学可以关注鹅厂的沙龙视频 --> [点击跳转](http://www.imooc.com/video/14125)

由于webpack的通用配置是固定代码, 我已经打包上传[github](https://github.com/coderZsq/coderZsq.webpack.js), 需要的同学可以进行[下载](https://github.com/coderZsq/coderZsq.webpack.js).
