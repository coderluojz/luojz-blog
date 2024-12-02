---
# 控制台使用 new Date().toISOString() 生成时间
pubDatetime: 2021-05-14T14:41:00.000Z
modDatetime: 2021-05-14T14:41:00.000Z
title: webpack 基础知识
# 页面唯一地址
slug: webpack-basic-usage
# 是否在首页精选版块显示此帖子
featured: false
# 将这篇文章标记为草稿。
draft: false
tags:
  - webpack
description: 什么是webpack？webpack是静态模块打包器，当webpack处理应用程序时，会将所有这些模块打包成一个或者多个文件。webpack可以处理js/css/图片、图标字体等单位。用于处理开发过程中存在于本地的静态资源。
---

> 什么是webpack？webpack是静态模块打包器，当webpack处理应用程序时，会将所有这些模块打包成一个或者多个文件。webpack可以处理js/css/图片、图标字体等单位。用于处理开发过程中存在于本地的静态资源。

_初始化`npm init -y`，新建文件`webpack.config.js`_

## 一、entry & output

> entry：入口文件；output：出口文件

_单入口与单出口的写法_

```javascript
const path = require("path");
module.exports = {
  // 单入口
  entry: "./src/index.js",
  // 出口
  output: {
    // 单出口文件名称
    filename: "main.js",
    path: path.resolve(__dirname, "dist"),
  },
};
```

_多入口与多出口写法_

```javascript
const path = require("path");
module.exports = {
  // 多入口
  entry: {
    index: "./src/index.js",
    search: "./src/search.js",
  },
  // 出口
  output: {
    // 多出口名称
    filename: "[name].js",
    path: path.resolve(__dirname, "dist"),
  },
};
```

## 二、loader

> 什么是loader？loader就是能够让webpack去处理那些非js文件，例如：图片、css文件等。当然loader也可以处理js文件。被用于转换某些类型的模块。

_安装loader依赖`npm i --save-dev babel-loader @babel/core @babel/preset-env`，在当前文件夹中新建`babel.config.json`,配置：_

```javascript
{
  "presets": ["@babel/preset-env"]
}
```

_在`webpack.config.js`中配置：_

```javascript
const path = require("path");
module.exports = {
  // 入口
  entry: {
    index: "./src/index.js",
  },
  // 出口
  output: {
    filename: "[name].js",
    path: path.resolve(__dirname, "dist"),
  },
  module: {
    // 规则
    rules: [
      {
        // 正则所有 JS 文件
        test: /\.js$/,
        // 排除 node_modules 文件
        exclude: /node_modules/,
        loader: "babel-loader",
      },
    ],
  },
};
```

**当打包`npm run webpack`后，会通过`webpack.config.js`中设置的规则，入口文件夹 src 下面的 index.js 文件就会被 babel-loader 处理，处理完成后会得到出口 dist 文件夹下编译后的 es6 以下的 index.js 文件**

_安装依赖 core-js ，在入口文件 index.js 中引入`import 'core-js/stable'`用于处理兼容IE部分语法_

## 三、plugins

> plugins(插件)，可以用于执行范围更广的任务，例如：打包优化和压缩，定义环境中的变量等。

**安装`html-webpack-plugin`插件处理html文件：**`npm install --save-dev html-webpack-plugin`

**单页面插件配置：**

```javascript
const path = require("path");
// 引入html-webpack-plugin插件
const htmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  mode: "development",
  // 入口
  entry: {
    index: "./src/index.js",
  },
  // 出口
  output: {
    filename: "[name].js",
    path: path.resolve(__dirname, "dist"),
  },
  // 插件
  plugins: [
    // 调用
    new htmlWebpackPlugin({
      template: "./index.html",
    }),
  ],
};
```

**多页面插件配置：**

```javascript
const path = require("path");
// 引入html-webpack-plugin插件
const htmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
  mode: "development",
  // 多入口
  entry: {
    index: "./src/index.js",
    search: "./src/search.js",
  },
  // 出口
  output: {
    filename: "[name].js",
    path: path.resolve(__dirname, "dist"),
  },
  // 多页面调用插件
  plugins: [
    new htmlWebpackPlugin({
      template: "./index.html",
      filename: "index.html",
      // 指定引入文件
      chunks: ["index"],
      // html-webpack-plugin其他
      minify: {
        // 删除注释
        removeComments: true,
        // 删除空格
        collapseWhitespace: true,
        // 删除标签属性值的双引号
        removeAttributeQuotes: true,
      },
    }),
    new htmlWebpackPlugin({
      template: "./search.html",
      filename: "search.html",
      chunks: ["search"],
    }),
  ],
};
```

## 四、简单使用

**1、安装`css-loader`和`mini-css-extract-plugin`插件处理css文件：**`npm install --save-dev css-loader mini-css-extract-plugin`

```javascript
const path = require("path");
// 引入html-webpack-plugin插件
const htmlWebpackPlugin = require("html-webpack-plugin");
// 引入mini-css-extract-plugin插件
const miniCssExtractPlugin = require("mini-css-extract-plugin");
module.exports = {
  mode: "development",
  // 入口
  entry: {
    index: "./src/index.js",
  },
  // 出口
  output: {
    filename: "[name].js",
    path: path.resolve(__dirname, "dist"),
  },
  module: {
    rules: [
      // 处理css文件
      {
        test: /\.css$/,
        // 从右向左执行
        use: [miniCssExtractPlugin.loader, "css-loader"],
      },
    ],
  },
  plugins: [
    // 处理html文件
    new htmlWebpackPlugin({
      template: "./index.html",
      filename: "index.html",
    }),
    // 处理css文件
    new miniCssExtractPlugin({
      filename: "css/[name].css",
    }),
  ],
};
```

**2、使用`file-loader`依赖处理css中本地图片，安装 loader：`npm install --save-dev file-loader`**

```javascript
const path = require("path");
// 引入html-webpack-plugin插件
const htmlWebpackPlugin = require("html-webpack-plugin");
// 引入mini-css-extract-plugin插件
const miniCssExtractPlugin = require("mini-css-extract-plugin");
module.exports = {
  mode: "development",
  // 入口
  entry: {
    index: "./src/index.js",
  },
  // 出口
  output: {
    filename: "[name].js",
    path: path.resolve(__dirname, "dist"),
  },
  module: {
    rules: [
      // 处理css文件
      {
        test: /\.css$/,
        // 从右向左执行
        use: [
          {
            loader: miniCssExtractPlugin.loader,
            // 路径
            options: {
              publicPath: "../",
            },
          },
          "css-loader",
        ],
      },
      // file-loader 处理css中本地图片
      {
        test: /\.(jpg|png|gif)$/,
        use: {
          loader: "file-loader",
          options: {
            // [ext] 代表当前图片的后缀
            name: "images/[name].[ext]",
          },
        },
      },
    ],
  },
  plugins: [
    // 处理html文件
    new htmlWebpackPlugin({
      template: "./index.html",
      filename: "index.html",
    }),
    // 处理css文件
    new miniCssExtractPlugin({
      filename: "css/[name].css",
    }),
  ],
};
```

**3、使用`html-withimg-loader`依赖处理HTML中本地图片，安装 loader：`npm install --save-dev html-withimg-loader`**

```javascript
const path = require("path");
// 引入html-webpack-plugin插件
const htmlWebpackPlugin = require("html-webpack-plugin");
// 引入mini-css-extract-plugin插件
const miniCssExtractPlugin = require("mini-css-extract-plugin");
module.exports = {
  mode: "development",
  // 入口
  entry: {
    index: "./src/index.js",
  },
  // 出口
  output: {
    filename: "[name].js",
    path: path.resolve(__dirname, "dist"),
  },
  module: {
    rules: [
      // 处理css文件
      {
        test: /\.css$/,
        // 从右向左执行
        use: [
          {
            loader: miniCssExtractPlugin.loader,
            // 路径
            options: {
              publicPath: "../",
            },
          },
          "css-loader",
        ],
      },
      // file-loader 处理css中本地图片
      {
        test: /\.(jpg|png|gif)$/,
        use: {
          loader: "file-loader",
          options: {
            // [ext] 代表当前图片的后缀
            name: "images/[name].[ext]",
            // 取消es6模块导出
            esModule: false,
          },
        },
      },
      // html-withimg-loader 处理HTML中的本地图片
      {
        test: /\.(htm|html)$/,
        use: "html-withimg-loader",
      },
    ],
  },
  plugins: [
    // 处理html文件
    new htmlWebpackPlugin({
      template: "./index.html",
      filename: "index.html",
    }),
    // 处理css文件
    new miniCssExtractPlugin({
      filename: "css/[name].css",
    }),
  ],
};
```

**4、使用`url-loader`依赖处理图片文件，安装 loader：`npm install --save-dev url-loader`，注意：使用`url-loader`时也要安装`file-loader`**

```javascript
// url-loader 处理本地图片
module: {
  rules: [
    {
      test: /\.(jpg|png|gif)$/,
      use: {
        loader: "url-loader",
        options: {
          // [ext] 代表当前图片的后缀
          name: "images/[name].[ext]",
          // 取消es6模块导出
          esModule: false,
          // 小于10KB的图片以Base64方式展示
          limit: 10000,
        },
      },
    },
  ];
}
```

**5、使用`webpack-dev-server`搭建开发环境，安装：`npm install --save-dev webpack-dev-server`**

_package.json中添加：`"dev":"webpack-dev-server --open"`_

```json
"scripts": {
    "webpack": "webpack --config webpack.config.js",
    "dev":"webpack-dev-server --open"
  },
```

_使用`npm run dev`运行开发环境_
