## 1.webpack介绍
- 本质上，webpack 是一个用于现代 JavaScript 应用程序的静态模块打包工具。当 webpack 处理应用程序时，它会在内部构建一个 依赖图(dependency graph)，此依赖图对应映射到项目所需的每个模块，并生成一个或多个 bundle
### 1.1 安装
```sh
npm install webpack webpack-cli --save-dev
```
### 1.2 入口(entry)
- 入口起点(entry point)指示 webpack 应该使用哪个模块,来作为构建其内部依赖图(dependency graph)的开始.进入入口起点后,webpack会找出有哪些模块和库是入口起点(直接和间接)依赖的
- 默认值是 ./src/index.js, 但你可以通过在 webpack.configuration 中配置 entry 属性,来指定一个(或多个)不同的入口起点
#### 1.2.1 src\index.js
```js
let title = require('./title.txt');
document.write(title.default);
```
#### 1.2.2 webpack.config.js
```js
const path = require("path");
module.exports = {
    entry: "./src/index.js"
};
```
### 1.3 输出(output)
- output 属性告诉 webpack 在哪里输出它所创建的 bundle,以及如何命名这些文件
- 主要输出文件的默认值是 ./dist/main.js, 其他生成文件默认放置在 ./dist 文件夹中.
webpack.config.js
```js
const path = require("path");
module.exports = {
    entry: "./src/index.js",
+   output: {
+       path: path.resolve(__dirname, 'dist'),
+       filename: 'main.js'
+   }
};
```
### 1.4 loader
- webpack只能理解 JavaScript 和 JSON 文件
- loader 让 webpack 能够去处理其他类型的文件，并将它们转换为有效模块，以供应用程序使用，以及被添加到依赖图中
webpack.config.js
```js
const path = require("path");
module.exports = {
    mode: "development",
    devtool: false,
    entry: "./src/index.js",
    output: {
        path: path.resolve(__dirname, "dist"),
        filename: "main.js"
    },
+   module: {
+       rules: [
+           { test: /\.txt$/, use: "raw-loader" }
+       ]
+   }
}
```
### 1.5 插件(plugin)
- loader用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。包括：打包优化，资源管理，注入环境变量
#### 1.5.1 src\index.html
src\index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>webpack5</title>
</head>
<body>
</body>
</html>
```
#### 1.5.2 webpack.config.js
```js
const path = require("path");
+ const HtmlWebpackPlugin = require("html-webpack-plugin");
module.exports = {
    mode: "development",
    devtool: false,
    entry: "./src/index.js",
    output: {
        path: path.resolve(__dirname, "dist"),
        filename: "main.js"
    },
    module: {
        rules: [
            { test: /\.txt$/, use: "raw-loader" }
        ]
    },
+   plugins: [
+       new HtmlWebpackPlugin({ template: "./src/index.html" })
+   ]
}
```
### 1.6 模式(mode)
- 日常的前端开发工作中，一般都会有两套构建环境
- 一套开发时使用，构建结果用于本地开发调试，不进行代码压缩，打印debug信息，包含sourcemap文件
- 一套构建后的结果是直接应用于线上的，即代码都是压缩后，运行时不打印debug信息，静态文件不包括 sourcemap
- webpack 4.x版本引入了[mode](https://webpack.docschina.org/configuration/mode/)的概念
- 当你指定使用production mode时，默认会启用各种性能优化的功能，包括构建结果优化以及webpack运行性能优化
- 而如果是 development mode的话，则会开启 debug 工具，运行时打印详细的错误信息，以及更加快速的增量编译构建

| 选项 | 描述 |
| --- | --- |
| development | 会将 process.env.NODE_ENV的值设为 development。启用 NamedChunksPlugin和NamedModulesPlugin |
| production | 会将 process.env.NODE_ENV的值设为 production.启用 FlagDependencyUsagePlugin,FlagIncludedChunksPlugin,ModuleConcatentionPlugin,NoEmitOnErrorsPlugin,OccurrenceOrderPlugin,SideEffectsFlagPlugin和UglifyJsPlugin |

#### 1.6.1 环境差异
- 开发环境
    - 需要生成 sourcemap 文件
    - 需要打印 debug 信息
    - 需要 live reload 或者 hot reload 的功能
- 生产环境
    - 可能需要分离CSS成单独的文件，以便多个页面共享同一个CSS文件
    - 需要压缩HTML/CSS/JS 代码
    - 需要压缩图片
- 其默认值为production
#### 1.6.2 区分环境
- --mode 用来设置模块内的 process.env.NODE_ENV
- --env 用来设置webpack配置文件的函数参数
- corss-env 用来设置node环境的 process.env.NODE_ENV
- DefinePlugin 用来设置模块内的全局变量
##### 1.6.2.1 命令行配置1
- webpack的mode默认为 production
- webpack serve的mode默认为development
- 可以在模块内通过 process.env.NODE_ENV 获取当前的环境变量，无法在 webpack 配置文件中获取此变量
```json
"scripts": {
    "build": "webpack",
    "start": "webpack serve
}
```
index.js
```js
console.log(process.env.NODE_ENV); // development | production
```
webpack.config.js
```js
console.log("NODE_ENV", process.env.NODE_ENV); // undefined
```
##### 1.6.2.2 命令行配置2
- 同配置1
```json
"scripts": {
    "build": "webpack --mode=production",
    "start": "webpack --mode=development serve"
}
```
##### 1.6.2.3 命令行配置
- 无法在模块内通过 process.env.NODE_ENV 访问
- 可以通过 webpack配置文件中 通过函数获取当前环境变量
```json
"scripts": {
    "dev": "webpack serve --env=development",
    "build": "webpack --env=production"
}
```
index.js
```js
console.log(process.env.NODE_ENV); // undefined
```
webpack.config.js
```js
console.log("NODE_ENV", process.env.NODE_ENV); // undefined
```
```js
module.exports = (env, argv) => {
    console.log("env", env); // development | production
};
```
##### 1.6.2.4 mode配置
- 和命令行配置2一样
```js
module.exports = {
    mode: "development"
}
```
##### 1.6.2.5 DefinePlugin
- 设置全局变量(不是window),所有模块都能读取到该变量的值
- 可以在任意模块内通过 process.env.NODE_ENV 获取当前的环境变量
- 但无法在 node环境 (webpack配置文件中)下获取当前的环境变量
```js
plugins: [
    new webpack.DefinePlugin({
        "process.env.NODE_ENV": JSON.stringify("development"),
        "NODE_ENV": JSON.stringify("production")
    })
]
```
index.js
```js
console.log(NODE_ENV); // production
```
webpack.config.js
```js
console.log('process.env.NODE_ENV', process.env.NODE_ENV); // undefined
console.log("NODE_ENV", NODE_ENV); // error !!!
```
##### 1.6.2.6 cross-env
- 只能设置node环境 下的变量 NODE_ENV
package.json
```json
"scripts": {
    "build": "cross-env NODE_ENV=development webpack"
}
```
webpack.config.js
```js
console.log("process.env.NODE_ENV", process.env.NODE_ENV); // development
```
## 2.开发环境配置
### 2.1 开发服务器
#### 2.1.1 安装服务器
```sh
npm install webpack-dev-server --save-dev
```
#### 2.1.2 webpack.config.js
| 类别 | 配置名称 | 描述 |
| --- | --- | --- |
| output | path | 指定输出到硬盘上的目录 |
| output | publicPath | 表示的是打包生成的index.html文件里面引用资源的前缀 |
| devServer | publicPath | 表示的是打包生成的静态文件所在的位置(若是devServer里面的publicPath没有设置，则会认为是output里面设置的publicPath的值) |
| devServer | contentBase | 用于配置提供额外静态文件内容的目录 |
#### 2.1.3 webpack.config.js
```js
module.exports = {
    devServer: {
        contentBase: path.resolve(__dirname, "dist"),
        compress: true,
        port: 8080,
        open: true
    }
}
```
#### 2.1.4 package.json
```json
"scripts": {
    "build": "webpack",
+   "start": "webpack serve"
}
```
### 2.2 支持CSS
- [css-loader](https://www.npmjs.com/package/css-loader)用来翻译处理@import和url()
- [style-loader](https://www.npmjs.com/package/style-loader)可以把CSS插入DOM中
#### 2.2.1 安装模块
```sh
cnpm i style-loader css-loader -D
```
```js
{
    test:/\.css$/,
    //最后一个loader,就上面最左边的loader一定要返回一个JS脚本
    use:['style-loader',{
        loader:'css-loader',
        options:{importLoaders:1}
    },'postcss-loader']
}
```
```css
#less-container{
    transform:rotate(7deg);
}
```
#### 2.2.2 webpack.config.js
```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
  mode: 'development',
  devtool:false,
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].js'
  },
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' },
+     { test: /\.css$/, use: ['style-loader','css-loader'] }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};
```
#### 2.2.3 src\bg.css
src\bg.css
```css
body{
    background-color: green;
}
```
#### 2.2.4 src\index.css
src\index.css
```css
@import "./bg.css";
body {
    color: red;
}
```
#### 2.2.5 src\index.js
src\index.js
```js
+ import "./index.css";
let title = require("./title.txt");
document.write(title.default);
```
### 2.3 支持less和sass
#### 2.3.1 安装
```sh
npm i less less-loader -D
npm i node-sass sass-loader -D
```
#### 2.3.2 webpack.config.js
webpack.config.js
```js
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
module.exports = {
    mode: "development",
    devtool: false,
    entry: "./src/index.js",
    output: {
        path: path.resolve(__dirname, "dist"),
        filename: "[name].js"
    },
    module: {
        rules: [
            { test: /\.txt$/, use: "raw-loader" },
            { test: /\.css$/, use: ['style-loader', 'css-loader'] },
+           { test: /\.less$/, use: ['style-loader','css-loader', 'less-loader'] },
+           { test: /\.scss$/, use: ['style-loader','css-loader', 'sass-loader'] }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({ template: "./src/index.html" })
    ]
};
```
#### 2.3.3 src\index.html
src\index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>webpack5</title>
</head>
<body>
+  <div id="less-container">less-container</div>
+  <div id="sass-container">sass-container</div>
</body>
</html>
```
#### 2.3.4 src\index.js
src\index.js
```js
import './index.css';
+import './less.less';
+import './sass.scss';
let title = require('./title.txt');
document.write(title.default);
```
#### 2.3.5 src\less.less
src\less.less
```less
@color:blue;
#less-container{
    color:@color;
}
```
#### 2.3.6 src\sass.scss
src\sass.scss
```scss
$color:orange;
#sass-container{
    color:$color;
}
```
### 2.4 CSS兼容性
- 为了浏览器的兼容性，有时候我们必须加入-webkit,-ms,-o,-moz这些前缀
    - Trident内核：主要代表为IE浏览器, 前缀为-ms
    - Gecko内核：主要代表为Firefox, 前缀为-moz
    - Presto内核：主要代表为Opera, 前缀为-o
    - Webkit内核：产要代表为Chrome和Safari, 前缀为-webkit
- 伪元素::placeholder可以选择一个表单元素的占位文本，它允许开发者和设计师自定义占位文本的样式。
#### 2.4.1 安装
- [https://caniuse.com/](https://caniuse.com/)
- [postcss-loader](https://github.com/webpack-contrib/postcss-loader)可以使用PostCSS处理CSS
- [postcss-preset-env](https://github.com/csstools/postcss-preset-env)把现代的CSS转换成大多数浏览器能理解的
- PostCSS Preset Env已经包含了autoprefixer和browsers选项
```sh
npm i postcss-loader postcss-preset-env -D
```
#### 2.4.2 postcss.config.js
postcss.config.js
```js
let postcssPresetEnv = require('postcss-preset-env');
module.exports={
    plugins:[postcssPresetEnv({
        browsers: 'last 5 version'
    })]
}
```
#### 2.4.3 webpack.config.js
```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
  mode: 'development',
  devtool: false,
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].js'
  },
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' },
+     { test: /\.css$/, use: ['style-loader', 'css-loader','post-css'] },
+     { test: /\.less$/, use: ['style-loader','css-loader','post-css','less-loader'] },
+     { test: /\.scss$/, use: ['style-loader','css-loader','post-css','sass-loader'] }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({ template: './src/index.html' })
  ]
};
```
#### 2.4.4 src\index.css
src\index.css
```css
@import "./bg.css";
body{
    color:red;
}
#logo{
    width:540px;
    height:258px;
    background-image: url(./assets/logo.png);
    background-size: cover;
}
+::placeholder {
+    color: red;
+}
```
#### 2.4.5 src\index.html
src\index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>webpack5</title>
</head>
<body>
  <div id="less-container">less-container</div>
  <div id="sass-container">sass-container</div>
  <div id="logo"></div>
+  <input placeholder="请输入"/>
</body>
</html>
```
#### 2.4.6 package.json
- [browserslist](https://github.com/browserslist/browserslist)
- [browserslist-example](https://github.com/browserslist/browserslist-example)
- .browserslistrc
```json
{
+  "browserslist": {
+    "development": [
+      "last 1 chrome version",
+      "last 1 firefox version",
+      "last 1 safari version"
+    ],
+    "production": [
+      ">0.2%"
+    ]
+  }
+}
```
### 2.5 支持图片
#### 2.5.1 安装
- [file-loader](http://npmjs.com/package/file-loader)解决CSS等文件中的引入图片路径问题
- [url-loader](https://www.npmjs.com/package/url-loader)当图片小于limit的时候会把图片BASE64编码，大于limit参数的时候还是使用file-loader进行拷贝
```sh
cnpm i file-loader url-loader html-loader -D
```
#### 2.5.2 webpack.config.js
```js
const path = require('path');
const HtmlWebpackPlugin = require("html-webpack-plugin");
module.exports = {
    mode: "development",
    devtool: false,
    entry: "./src/index.js",
    output: {
        path: path.resolve(__dirname, "dist"),
        filename: "[name].js"
    },
    module: {
        rules: [
            { test: /\.txt$/, use: 'raw-loader' },
            { test: /\.css$/, use: ['style-loader', 'css-loader'] },
            { test: /\.less$/, use: ['style-loader','css-loader', 'less-loader'] },
            { test: /\.scss$/, use: ['style-loader','css-loader', 'sass-loader'] },
    +       { test: /\.(jpg|png|bmp|gif|svg)$/, 
    +        use: [{
    +          loader: 'url-loader', 
    +          options: {
    +            esModule: false,
    +            name: '[hash:10].[ext]',
    +            limit: 8*1024,
    +          }
    +        }]
    +      }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({ template: "./src/index.html" })
    ]
}
```
#### 2.5.3 src\index.html
src\index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>webpack5</title>
</head>
<body>
  <div id="less-container">less-container</div>
  <div id="sass-container">sass-container</div>
+ <div id="logo"></div>
</body>
</html>
```
#### 2.5.4 src\index.js
src\index.js
```js
import "./index.css";
import "./less.less";
import "./sass.scss";
let title = require("./title.txt");
document.write(title.default);
+ let logo = require("./assets/logo.png");
+ let img = new Image();
+ img.src = logo.default;
+ document.body.appendChild(img);
```
### 2.6 JS兼容性处理
- Babel其实是一个编译JavaScript的平台，可以把ES6/ES7，React的JSX转译为ES5
#### 2.6.1 @babel/preset-env
- Babel默认只转换新的最新ES语法，比如箭头函数
##### 2.6.1.1 安装依赖
- [babel-loader](https://www.npmjs.com/package/babel-loader)使用Babel和webpack转译JavaScript文件
- [@babel/@babel/core](https://www.npmjs.com/package/@babel/core)Babel编译的核心包
- [babel-preset-env](https://www.babeljs.cn/docs/babel-preset-env)
- [@babel/@babel/preset-react](https://www.npmjs.com/package/@babel/preset-react)React插件的Babel预设
- [@babel/plugin-proposal-decorators](https://babeljs.io/docs/en/babel-plugin-proposal-decorators)把类和对象装饰器编译成ES5
- [@babel/plugin-proposal-class-properties](https://babeljs.io/docs/en/babel-plugin-proposal-class-properties)转换静态类属性以及使用属性初始化值化语法声明的属性
```sh
cnpm i babel-loader @babel/core @babel/preset-env @babel/preset-react -D
cnpm i @babel/plugin-proposal-decorators @babel/plugin-proposal-class-properties -D
```
##### 2.6.1.2 webpack.config.js
```js
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
module.exports = {
    mode: "development",
    devtool: false,
    entry: "./src/index.js",
    output: {
        path: path.resolve(__dirname, "dist"),
        filename: "[name].js",
        publicPath: "/"
    },
    module: {
        rules: [
    +       {
    +          test: /\.jsx?$/,
    +          use: {
    +            loader: 'babel-loader',
    +            options: {
    +              presets: [["@babel/preset-env",{
    +                targets: "> 0.25%, not dead",
    +              }], '@babel/preset-react'],
    +              plugins: [
    +                ['@babel/plugin-proposal-decorators', { legacy: true }],
    +                ['@babel/plugin-proposal-class-properties', { loose: true }],
    +              ],
    +            },
    +          },
    +       },
            {
                test: /\.txt$/, use: "raw-loader"
            },
            {
                test: /\.css$/, use [MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader']
            },
            {
                test: /\.less$/, use: [MiniCssExtractPlugin.loader, 'css-loader', 'less-loader']
            },
            {
                test: /\.scss$/, use: [MiniCssExtractPlugin.loader, 'css-loader', 'sass-loader']
            },
            {
                test: /\.(jpg|png|bmp|gif|svg)$/,
                use: [{
                    loader: 'url-loader',
                    options: {
                        limit: 10,
                        outputPath: "images",
                        publicPath: "/images"
                    }
                }]
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: "./src/index.html"
        }),
        new MiniCssExtractPlugin({
            filename: "css/[name].css"
        }),
    ]
};
```
##### 2.6.1.3 src\index.js
src\index.js
```js
+ function readonly(target, key, descriptor) {
+   descriptor.writable = false;
+ }
+
+ class Person {
+   @readonly PI = 3.14;
+ }
+ let p1 = new Person();
+ p1.PI = 3.15;
+ console.log(p1);
```
##### 2.6.1.4 jsconfig.json
- [jsconfig](https://code.visualstudio.com/docs/languages/jsconfig)
jsconfig.json
```json
{
    "compilerOptions": {
        "experimentalDecorators": true
    }
}
```
### 2.7 ESLint代码校验
#### 2.7.1 安装
- [eslint](https://eslint.org/docs/developer-guide/nodejs-api#cliengine)
- [eslint-loader](https://www.npmjs.com/package/eslint-loader)
- [configuring](https://eslint.org/docs/user-guide/configuring)
- [babel-eslint](https://www.npmjs.com/package/babel-eslint)
- [Rules](https://cloud.tencent.com/developer/chapter/12618)
- [ESlint语法检测配置说明](https://segmentfault.com/a/1190000008742240)
```sh
cnpm install eslint eslint-loader babel-eslint --D
```
#### 2.7.2 webpack.config.js
```js
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
module.exports = {
    mode: "development",
    devtool: false,
    entry: "./src/index.js",
    output: {
        path: path.resolve(__dirname, "dist"),
        filename: "[name].js",
        publicPath: "/"
    },
    module: {
        rules: [
+           {
+               test: /\.jsx?$/,
+               loader: "eslint-loader",
+               enforce: "pre",
+               options: { fix: true },
+               exclude: /node_modules/,
+           },
            {
                test: /\.jsx?$/,
                use: {
                    loader: "babel-loader",
                    options: {
                        "presets": ["@babel/preset-env"],
                        "plugins": [
                            ["@babel/plugin-proposal-decorators", { "legacy": true }],
                            ["@babel/plugin-proposal-class-properties", { "loose": true }]
                        ]
                    }
                },
                include: path.join(__dirname, "src"),
                exclude: /node_modules/
            },
            {
                test: /\.txt$/, use: "raw-loader"
            },
            {
                test: /\.css$/, use: [MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader']
            },
            {
                test: /\.less$/, use: [MiniCssExtractPlugin.loader, 'css-loader', 'less-loader']
            },
            {
                test: /\.scss$/, use: [MiniCssExtractPlugin.loader, 'css-loader', 'sass-loader']
            },
            {
                test: /\.(jpg|png|bmp|gif|svg)$/,
                use: [
                    {
                        loader: 'url-loader',
                        options: {
                            limit: 10,
                            outputPath: "images",
                            publicPath: "/images"
                        }
                    }
                ]
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({ template: "./src/index.html" }),
        new MiniCssExtractPlugin({
            filename: 'css/[name].css'
        }),
    ]
}
```
#### 2.7.3 src\index.html
src\index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>webpack5</title>
</head>
<body>
+  <div id="root"></div>
</body>
</html>
```
#### 2.7.4 src\index.js
src\index.js
```js
+ import React from "react";
+ import ReactDOM from "react-dom";
+ ReactDOM.render("hello", document.getElementById("root"));
+
+ function readonly(target, key, descriptor) {
    descriptor.writable = false;
+ }
+
+ class Person {
+   @readonly PI = 3.14;
+ }
+ let p1 = new Person();
+ p1.PI = 3.15;
```
#### 2.7.5 .eslintrc.js
.eslintrc.js
```js
module.exports = {
    root: true,
    parser: "babel-eslint",
    // 指定解析器选项
    parserOptions: {
        sourceType: "module",
        ecmaVersion: 2015
    },
    // 指定脚本的运行环境
    env: {
        browser: true
    },
    // 启用的规则及其各自的错误级别
    rules: {
        "indent": "off", // 缩进风格
        "quotes": "off", // 引号类型
        "no-console": "error", // 禁止使用console
    }
}
```
#### 2.7.6 airbnb
- [eslint-config-airbnb](https://github.com/airbnb/javascript/tree/master/packages/eslint-config-airbnb)
```sh
cnpm i eslint-config-airbnb eslint-loader eslint eslint-plugin-import eslint-plugin-react eslint-plugin-react-hooks and eslint-plugin-jsx-a11y -D
```
eslintrc.js
```js
module.exports = {
    "parser": "babel-eslint",
    "extends": "airbnb",
    "rules": {
        "semi": "error",
        "no-console": "off",
        "linebreak-style": "off",
        "eol-last": "off"
        // "indent": ["error", 2]
    },
    "env": {
        "browser": true,
        "node": true
    }
}
```
#### 2.7.7 自动修复
- 安装vscode的[eslint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)插件
- 配置自动修复参数
.vscode\settings.json
```json
{
    "eslint.validate": [
        "javascript",
        "javascriptreact",
        "typescript",
        "typescriptreact"
    ],
    "editor.codeActionOnSave": {
        "source.fixAll.eslint": true
    }
}
```
### 2.8 sourcemap
- [sourcemap]()是为了解决开发代码与实际运行代码不一致时帮助我们 debug 到原始开发代码的技术
- webpack 通过配置可以自动给我们 source.maps文件，map文件是一种对应编译文件和源文件的方法
- [whyeval](https://github.com/webpack/docs/wiki/build-performance#sourcemaps)可以单独缓存map,重建性能更高
- [source-map](https://github.com/mozilla/source-map)
#### 2.8.1 配置项
| 类型 | 含义 |
| --- | --- |
| source-map | 原始代码 最好的sourcemap 质量有完整的结果，但是会很慢 |
| eval-source-map | 原始代码 同样道理，但是最高的质量和最低的性能 |
| cheap-module-eval-source-map | 原始代码（只有行内）同样道理，但是更高的质量和更低的性能 |
| cheap-eval-source-map | 转换代码（行内）每个模块被eval执行，并且sourcemap作为eval的一个dataurl |
| eval | 生成代码 每个模块都被 eval 执行，并且存在@sourceURL，带eval的构建模式能cache SourceMap |
| cheap-source-map | 转换代码（行内）生成的sourcemap没有列映射，从loaders生成的sourcemap没有被使用 |
| cheap-module-source-map | 原始代码（只有行内）与上面一样除了每行特点的从loader中进行映射 |
#### 2.8.2 关键字
- 看似配置项很多，其实只有五个关键字eval、source-map、cheap、module和inline的任意组合
- 关键字可以任意组合，但是又顺序要求

| 关键字 | 含义 |
| --- | --- |
| eval | 使用eval包裹模块代码 |
| source-map | 产生.map文件 |
| cheap | 不包含列信息(关于列信息的解释下面会有详细介绍)也不包含loader的sourcemap |
| module | 包含loader的sourcemap（比如jsx to js，babel的sourcemap）,否则无法定义源文件 |
| inline | 将.map作为DataURI嵌入，不单独生成.map文件 |
#### 2.8.3 webpack.config.js
```js
module.exports = {
    devtool: "source-map",
    devtool: "eval-source-map",
    devtool: "cheap-module-eval-source-map",
    devtool: "cheap-eval-source-map",
    devtool: "eval",
    devtool: "cheap-source-map",
    devtool: "cheap-module-source-map"
}
```
#### 2.8.4 组合规则
- [inline-|hidden-|eval-][nosources-][cheap-[module-]]source-map
- source-map单独在外部生成完整的sourcemap文件，并且在目标文件里建立关联，能提示错误代码的准确原始位置
- inline-source-map 以base64格式内联在打包后的文件中，内联构建速度更快，也能提示错误代码的准确原始位置
- hidden-source-map 会在外部生成sourcemap文件，但是在目标文件里没有建立关联，不能提示错误代码的准确原始位置
- eval-source-map 会为每一个模块生成一个单独的sourcemap文件进行内联，并使用eval执行
- nosources-source-map 也会在外部生成sourcemap文件，能找到原始代码位置，但源代码内容为空
- cheap-source-map 外部生成sourcemap文件，不包含列和loader的map
- cheap-module-source-map 外部生成sourcemap文件，不包含列的信息但包含loader的map
#### 2.8.5 最佳实践
##### 2.8.5.1 开发环境
- 我们在开发环境对sourceMap的要求是：速度快，调试更友好
- 要想速度快,推荐 eval-cheap-source-map
- 如果想调试更友好 cheap-module-source-map
- 折中的选择就是 eval-source-map
##### 2.8.5.2 生产环境
- 首先排除内联，因为一方面我们要隐藏源代码，另一方面要减少文件体积
- 要想调试友好 sourcemap>cheap-source-map/cheap-module-source-map>hidden-source-map/nosources-sourcemap
- 要想速度快，优先选择 cheap
- 折中的选择就是 hidden-source-map
#### 2.8.6 调试代码
##### 2.8.6.1 测试环境调试
- [source-map-dev-tool-plugin](https://www.webpackjs.com/plugins/source-map-dev-tool-plugin/)实现了对source map生成，进行更细粒度的控制
    - filename(string): 定义生成的source map的名称（如果没有值将会变成 inlined）。
    - append(string): 在原始资源后追加给定值。通常是#sourceMappingURL注释。[url]被替换成source map文件的URL
- 市面上流行两种形式的文件指定,分别是以@和#符号开头的，@开头的已经被废弃
![](/public/images/enablesourcemap.png)

webpack.config.js
```js
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const OptimizeCssAssetsWebpackPlugin = require("optimize-css-assets-webpack-plugin");
const TerserPlugin = require("terser-webpack-plugin");
+ const FileManagerPlugin = require("filemanager-webpack-plugin");
+ const webpack = require("webpack");

module.exports = {
    mode: "none",
    devtool: false,
    entry: "./src/index.js",
    optimization: {
        minimize: true,
        minimizer: [
            new TerserPlugin(),
        ],
    },
    output: {
        path: path.resolve(__dirname, "dist"),
        filename: "[name].js",
        publicPath: "/",
    },
    devServer: {
        contentBase: path.resolve(__dirname, "dist"),
        compress: true,
        port: 8080,
        open: true
    },
    module: {
        rules: [
            {
                test: /\.jsx?$/,
                loader: "eslint-loader",
                enforce: "pre",
                options: { fix: true },
                exclude: /node_moduels/,
            },
            {
                test: /\.jsx?$/,
                use: {
                    loader: "babel-loader",
                    options: {
                        presets: [
                            ["@babel/preset-env", {
                                useBuiltIns: "usage", // 按需要加载polyfill
                                corejs: {
                                    version: 3, // 指定core-js版本
                                },
                                targets: { // 指定要兼容到哪些版本的浏览器
                                    chrome: '60',
                                    firefox: '60',
                                    ie: '9',
                                    safari: '10',
                                    edge: '17',
                                },
                            }], "@babel/preset-react"
                        ],
                        plugins: [
                            ["@babel/plugin-proposal-decorators", { legacy: true }],
                            ["@babel/plugin-proposal-class-properties", { loose: true }],
                        ],
                    },
                },
                include: path.join(__dirname, "src"),
                exclude: /node_modules/,
            },
            { test: /\.txt$/, use: "raw-loader" },
            { test: /\.css$/, use: [MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader']},
            { test: /\.less$/, use: [MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader', 'less-loader']},
            { test: /\.scss$/, use: [MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader', 'sass-loader']},
            {
                test: /\.(jpg|png|bmp|gif|svg)$/,
                use: [
                    {
                        loader: 'url-loader',
                        options: {
                            esModule: false,
                            name: '[hash:10].[ext]',
                            limit: 8*1024,
                            outputPath: "images",
                            publicPath: "/images",
                        },
                    }
                ],
            },
            {
                test: /\.html$/,
                loader: "html-loader",
            },
        ],
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: "./src/index.html",
            minify: {
                collapseWhitespace: true,
                removeComments: true
            }
        }),
        new MiniCssExtractPlugin({
            filename: 'css/[name].css',
        }),
        new OptimizeCssAssetsWebpackPlugin(),
+       new webpack.SourceMapDevToolPlugin({
+           append: '\n//# sourceMappingURL=http://127.0.0.1:8081/[url]',
+           filename: '[file].map',
+       }),
+       new FileManagerPlugin({
+           events: {
+               onEnd: {
+                   copy: [{
+                   source: './dist/*.map',
+                   destination: 'C:/aprepare/zhufengwebpack2021/1.basic/sourcemap',
+                   }],
+               delete: ['./dist/*.map'],
+           },
+        },
+       }),
    ]
}
```
##### 2.8.6.2 生产环境调试
- webpack打包仍然生产sourceMap，但是将map文件挑出放到本地服务器，将不含有map文件的部署到服务器
![](/public/images/addsourcemapfile.png)
### 2.9 打包第三方类库
#### 2.9.1 直接引入
```js
import _ from "lodash";
alert(_join(["a", "b", "c"], "@"));
```
#### 2.9.2 插件引入
- webpack配置ProvidePlugin后，在使用时将不再需要import和require进行引入，直接使用即可
- _函数会自动添加到当前模块的上下文,无需显示声明
```js
+ new webpack.ProvidePlugin({
+   _: "lodash"
+ });
```
> 没有全局的$函数，所以导入依赖全局变量的插件依旧会失败
### 2.9.3 expose-loader
- expose-loader可以把模块添加到全局对象上，在调试的时候比较有用
- The expose loader adds modules to the global object. This is useful for debugging
- 不需要任何其他的插件配合，只要将下面的代码添加到所有的 loader 之前
```js
module: {
    rules: [
+      {
+          test: require.resolve('lodash'),
+          loader: 'expose-loader',
+          options: {
+              exposes: {
+                  globalName: '_',
+                  override: true,
+              },
+          },
+      }
    ]
}
```
#### 2.9.4 externals
如果我们想引用一个库，但是又不想让webpack打包，并且又不影响我们在程序中以CMD、AMD或者window/global全局等方式进行使用，那就可以通过配置externals
```js
const jQuery = require("jquery");
import jQuery from "jquery";
```
```html
<script src="https://cdn.bootcss.com/jquery/3.4.1/jquery.js"></script>
```
```js
+externals: {
+  lodash: '_',
+},
  module: {
```
### 2.10 watch
- 当代码发生修改后可以自动重新编译
```js
module.exports = {
    // 默认 false，也就是不开启
    watch: true,
    // 只有开启监听模式时，watchOptions才有意义
    watchOptions: {
        // 默认为空，不监听的文件或者文件夹，支持正则匹配
        ignored: /node_modules/,
        // 监听到变化发生后会等300ms再去执行，默认300ms
        aggregateTimeout: 300,
        // 判断文件是否发生变化是通过不停的询问文件系统指定议是有变化实现的，默认每秒问1000此
        pool: 1000
    }
}
```
- webpack定时获取文件的更新时间，并跟上次保存的时间进行比对，不一致就表示发生了变化,poll就用来配置每秒问多少次
- 当检测文件不再发生变化，会先缓存起来，等待一段时间后之后再通知监听者，这个等待时间通过aggregateTimeout配置
- webpack只会监听entry依赖的文件
- 我们需要尽可能减少需要监听的文件数量和检查频率，当然频率的降低会导致灵敏度下降
### 2.11 添加商标
```js
+ new webpack.BannerPlugin('珠峰架构)
```
### 2.12拷贝静态文件
- [copy-webpack-plugin](https://webpack.js.org/plugins/copy-webpack-plugin)可以拷贝源文件到目标目录
```sh
npm i copy-webpack-plugin -D
```
```js
+const CopyWebpackPlugin = require('copy-webpack-plugin');
+new CopyWebpackPlugin({
+  patterns: [{
+    from: path.resolve(__dirname,'src/static'),//静态资源目录源地址
+    to: path.resolve(__dirname,'dist/static'), //目标地址，相对于output的path目录
+  }],
+}),
```
### 2.13 clean-webpack-plugin
- [clean-webpack-plugin](https://www.npmjs.com/package/clean-webpack-plugin)可以打包前先清空输出目录
```sh
npm i clean-webpack-plugin -D
```
```js
+ const { CleanWebpackPlugin } = require("clean-webpack-plugin");
plugins: [
    + new CleanWebpackPlugin({cleanOnceBeforeBuildPatterns: ["**/*"]})
]
```
### 2.14 服务器代理
如果你有单独的后端开发服务器API，并且希望在同域名下发送API请求，那么代理某些URL会很有用，
#### 2.14.1 不修改路径
- 请求到 /api/users 现在会被代理到请求[http://localhost:3000/api/users](http://localhost:3000/api/users%E3%80%82)
```js
devServer: {
    proxy: {
        "/api": "http://localhost:3000"
    }
}
```
#### 2.14.2 修改路径
```js
devServer: {
    proxy: {
        "/api": {
            target: "http://localhost:3000",
            pathRewrite: {"^/api": ""}
        }
    }
}
```
#### 2.14.3 before after
- before在webpack-dev-server静态资源中间件处理之前，可以用于拦截部分请求返回特定内容，或者实现简单的数据mock。
```js
devServer: {
    before(app) {
        app.get('/api/users', function(req, res) {
            res.json([{id: 1, name: 'zhufeng'}])
        })
    }
}
```
#### 2.14.4 webpack-dev-middleware
- [webpack-dev-middleware](https://www.npmjs.com/package/)就是在Express中提供webpack-dev-server静态服务能力的一个中间件
```sh
npm install webpack-dev-middleware --save-dev
```
```js
const express = require("express");
const app = express();
const webpack = require("webpack");
const webpackDevMiddleware = require("webpack-dev-middleware");
const webpackOptions = require("./webpack.config");
webpackOptions.mode = "development";
const compiler = webpack(webpackOptions);
app.use(webpackDevMiddleware(compiler, {}));
app.listen(3000);
```
- webpack-dev-server的好处是相对简单，直接安装依赖后执行命令即可
- 而使用webpack-dev-middleware的好处是可以在既有的Express代码基础上快速添加webpack-dev-server的功能，同时利用Express来根据需要添加更多的功能，如mock服务、代理API请求等
## 3.生产环境
### 3.1 提取CSS
- 因为CSS的下载和JS可以并行，当一个HTML文件很大的时候，我们可以把CSS单独提取出来加载
#### 3.1.1 安装
- [mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin)
```sh
cnpm install --save-dev mini-css-extract-plugin
```
#### 3.1.2 webpack.config.js
webpack.config.js
```js
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
+ const MiniCssExtractPlugin = require("mini-css-extract-plugin");
module.exports = {
    mode: "development",
    devtool: false,
    entry: "./src/index.js",
    output: {
        path: path.resolve(__dirname, "dist"),
        filename: "[name].js",
+       publicPath: "/"
    },
    module: {
    rules: [
        { test: /\.txt$/, use: 'raw-loader' },
+       { test: /\.css$/, use: [MiniCssExtractPlugin.loader, 'css-loader'] },
+       { test: /\.less$/, use: [MiniCssExtractPlugin.loader, 'css-loader', 'less-loader'] },
+      { test: /\.scss$/, use: [MiniCssExtractPlugin.loader, 'css-loader', 'sass-loader'] },
            {   
                test: /\.(jpg|png|bmp|gif|svg)$/, 
                use: [{
                    loader: 'url-loader', 
                    options: {
                        esModule: false,
                        name: '[hash:10].[ext]',
                        limit: 8*1024
                    }
                }]
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({ template: './src/index.html' }),
+       new MiniCssExtractPlugin({
+           filename: '[name].css'
+       })
  ]
}
```
### 3.2 指定图片和CSS目录
#### 3.2.1 webpack.config.js
```js
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
module.exports = {
    mode: "development",
    devtool: false,
    entry: "./src/index.js",
    output: {
        path: path.resolve(__dirname, "dist"),
        filename: "[name].js",
        publicPath: "/"
    },
    module: {
        rules: [
            { test: /\.txt$/, use: "raw-loader" },
            { test: /\.css$/, use: [MiniCssExtractPlugin.loader, 'css-loader']},
            { test: /\.less$/, use: [MiniCssExtractPlugin.loader, 'css-loader', 'less-loader']},
            { test: /\.scss$/, use: [MiniCssExtractPlugin.loader, 'css-loader', 'sass-loader'] },
            { test: /\.(jpg|png|bmp|gif|svg)$/, 
                use: [{
                loader: 'url-loader', 
                options: {
                    esModule: false,
                    name: '[hash:10].[ext]',
                    limit: 8*1024,
        +           outputPath: 'images',
        +           publicPath: '/images'
                }
                }]
            }
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({ template: "./src/index.html" }),
        new MiniCssExtractPlugin({
+           filename: "css/[name].css"
        }),
    ]
};
```
### 3.3 hash、chunkhash和contenthash
- 文件指纹 是指打包后输出的文件名和后缀
- hash一般是结合CDN缓存来使用，通过webpack构建之后，生成对应文件名自动带上对应的MD5值。如果文件内容改变的话，那么对应文件哈希值也会改变，对应的HTML引用的URL地址也会改变，触发CDN服务器从源服务器上拉取对应数据，进而更新本地缓存。
指纹占位符

| 占位符名称 | 含义 |
| --- | --- |
| ext | 资源后缀名 |
| name | 文件名称 |
| path | 文件的相对路径 |
| folder | 文件所在的文件夹 |
| hash | 每次webpack构建时生成一个唯一的hash值 |
| chunkhash | 根据chunk生成hash值，来源于同一个chunk,则hash值就一样 |
| contenthash | 根据内容生成hash值，文件内容相同hash值就相同 |
#### 3.3.1 hash
![](/public/images/variableHash.png)
```js
function createHash() {
    return require('crypto').createHash("md5");
}
let entry = {
    entry1: "entry1",
    entry2: "entry2"
}
let entry1 = 'require depModule1'; // 模块entry1
let entry2 = 'require depModule2'; // 模块entry2

let depModule1 = 'depModule1'; // 模块 depModule1
let depModule2 = 'depModule2'; // 模块 depModule2
// 如果都使用hash的话，因为这是工程级别的，即每次修改任何一个文件，所有文件名的hash值都将改变。所以一旦修改了任何一个文件，整个项目的文件缓存都将失效
let hash = createHash()
.update(entry1)
.update(entry2)
.update(depModule1)
.update(depModule2)
.digest('hex');
console.log('hash', hash);
// chunkhash根据不同的入口文件（Entry）进行依赖文件解析、构建对应的chunk,生成对应的哈希值。
// 在生产环境里把一些公共库和程序入口文件区分开，单独打包构建，接着我们采用chunkhash的方式生产哈希值，那么只要我们不改动公共库的代码，就可以保证其哈希值不会受影响
let entry1ChunkHash = createHash()
.update(entry1)
.update(depModule1).digest('hex');
console.log('entry1ChunkHash', entry1ChunkHash);

let entry2ChunkHash = createHash()
.update(entry2)
.update(depModule2).digest('hex');
console.log('entry2ChunkHash', entry2ChunkHash);

let entry1File = entry1+depModule1;
let entry1ContentHash = createHash()
.update(entry1File).digest('hex');;
console.log('entry1ContentHash',entry1ContentHash);

let entry2File = entry2+depModule2;
let entry2ContentHash = createHash()
.update(entry2File).digest('hex');;
console.log('entry2ContentHash',entry2ContentHash);
```
### 3.3.2 hash
- Hash是整个项目的hash值，其根据每次编译内容计算得到，每次编译之后都会生成新的hash,即修改任何文件都会导致所有文件的hash发生改变
```js
module.exports = {
+   entry: {
+       main: "./src/index.js",
+       vender: ["lodash"]
+   },
    output: {
        path: path.resolve(__dirname, "dist"),
+       filename: "[name].[hash].js"
    },
    plugins: [
        new MiniCssExtractPlugin({
+           filename: "css/[name].[hash].css"
        })
    ]
}
```
#### 3.3.2 chunkhash
- chunkhash采用hash计算的话，每一次构建后生成的哈希值都不一样，即使文件内容压根没有改变，这样子是没办法实现缓存效果，我们需要换另一种哈希值计算方式，即chunkhash
- chunkhash和hash不一样，它根据不同的入口文件(Entry)进行依赖文件解析、创建对应的chunk、生成对应的哈希值。我们在生产环境里把一些公共库和程序入口文件区分开，单独打包构建，接着我们采用chunkhash的方式生成哈希值，那么只要我们不改动公共库的代码，就可以保证哈希值不会受影响
```js
module.exports = {
    entry: {
        main: "./src/index.js",
        vender: ["lodash"]
    },
    output: {
        path: path.resolve(__dirname, "dist"),
+       filename: "[name].[chunkhash].js"
    },
    plugins: [
        new MiniCssExtractPlugin({
+           filename: "css/[name].[chunkhash].css"
        })
    ]
};
```
#### 3.3.3 contenthash
- 使用chunkhash存在一个问题，就是当在一个JS文件中引入CSS文件，编译后他们的hash是相同的，而且只要js文件发生改变，关联的CSS文件hash也会改变，这个时候可以使用mini-css-extract-plugin里的 contenthash值，保证即使css文件所处的模块里就算其他文件内容改变，只要css文件内容不变，那么不会重复构建
```js
module.exports = {
    plugins: [
        new MiniCssExtractPlugin({
+           filename: "css/[name].[contenthash].css"
        })
    ],
};
```
### 3.4 压缩JS、CSS和HTML
- [optimize-css-assets-webpack-plugin](https://www.npmjs.com/package/optimize-css-assets-webpack-plugin)是一个优化和压缩CSS资源的插件
- [terser-webpack-plugin](https://www.npmjs.com/package/terser-webpack-plugin)是一个优化和压缩JS资源的插件

webpack.config.js
```js
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
+ const OptimizeCssAssetsWebpackPlugin = require("optimize-css-assets-webpack-plugin");
+ const TerserPlugin = require("terser-webpack-plugin");

module.exports = {
+   mode: "none",
    devtool: false,
    entry: "./src/index.js",
+   optimization: {
+     minimize: true,
+     minimizer: [
+       new TerserPlugin(),
+     ],
+   },
    output: {
        path: path.resolve(__dirname, "dist"),
        filename: "[name].js",
        publicPath: "/"
    },
    devServer: {
        contentBase: path.resolve(__dirname, "dist"),
        compress: true,
        port: 8080,
        open: true
    },
    module: {
        rules: [
            {
                test: /\.jsx?$/,
                loader: 'eslint-loader',
                enforce: 'pre',
                options: { fix: true },
                exclude: /node_modules/,
            },
            {
                test: /\.jsx?$/,
                use: {
                loader: 'babel-loader',
                options: {
                    presets: [[
                    '@babel/preset-env',
                    {
                        useBuiltIns: 'usage'
                        corejs: {
                        version: 3
                        },
                        targets: {
                        chrome: '60',
                        firefox: '60',
                        ie: '9',
                        safari: '10',
                        edge: '17',
                        },
                    },
                    ], '@babel/preset-react'],
                    plugins: [
                    ['@babel/plugin-proposal-decorators', { legacy: true }],
                    ['@babel/plugin-proposal-class-properties', { loose: true }],
                    ],
                },
                },
                include: path.join(__dirname, 'src'),
                exclude: /node_modules/,
            },
            { test: /\.txt$/, use: 'raw-loader' },
            { test: /\.css$/, use: [MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader'] },
            { test: /\.less$/, use: [MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader', 'less-loader'] },
            { test: /\.scss$/, use: [MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader', 'sass-loader'] },
            {
                test: /\.(jpg|png|bmp|gif|svg)$/,
                use: [{
                loader: 'url-loader',
                options: {
                    esModule: false,
                    name: '[hash:10].[ext]',
                    limit: 8 * 1024,
                    outputPath: 'images',
                    publicPath: '/images',
                },
                }],
            },
            {
                test: /\.html$/,
                loader: 'html-loader',
            },
        ]
    },
    plugins: [
        new HtmlWebpackPlugin({
            template: "./src/index.html",
        +   minify: {  
        +        collapseWhitespace: true,
        +        removeComments: true
            }
        }),
        new MiniCssExtractPlugin({
            filename: "css/[name].css",
        }),
+       new OptimizeCssAssetWebpackPlugin(),
    ]
}
```
### 3.5 图片压缩
- [image-webpack-loader](https://www.npmjs.com/package/image-webpack-loader)可以帮助我们对图片进行压缩和优化
```js
npm install image-webpack-loader --save-dev
```
```js
 {
          test: /\.(png|svg|jpg|gif|jpeg|ico)$/,
          use: [
            'url-loader',
+           {
+             loader: 'image-webpack-loader',
+             options: {
+               mozjpeg: {
+                 progressive: true,
+                 quality: 65
+               },
+               optipng: {
+                 enabled: false,
+               },
+               pngquant: {
+                 quality: '65-90',
+                 speed: 4
+               },
+               gifsicle: {
+                 interlaced: false,
+               },
+               webp: {
+                 quality: 75
+               }
+             }
+           }
          ]
        }
```
### 3.6 px自动转成rem
- [lib-flexible](https://github.com/amfe/lib-flexible) + rem,实现移动端自适应
- [px2rem-loader](https://www.npmjs.com/package/px2rem-loader) 自动将px转换为rem
- [px2rem](https://github.com/songsiqi/px2rem)
- 页面渲染时计算根元素的font-size值
#### 3.6.1 安装
```sh
cnpm i px2rem-loader lib-flexible -D
```
#### 3.6.2 index.html
index.html
```html
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>主页</title>
    <script>
        let docEle = document.documentElement;
        function setRemUnit() {
            // 750/10=75    375/10=37.5
            docEle.style.fontSize = docEle.clientWidth / 10 + 'px';
        }
        setRemUnit();
        window.addEventListener('resize', setRemUnit);
    </script>
</head>
<body>
    <div id="root"></div>
</body>
```
#### 3.6.3 reset.css
src/reset.css
```css
* {
    padding: 0;
    margin: 0
}
#root {
    width: 750px;
    height: 750px;
    border: 1px solid red;
    box-sizing: border-box;
}
```
#### 3.6.4 webpack.config.js
```js
 {
      test:/\.css$/,
        use:[{
                MiniCssExtractPlugin.loader,
                'css-loader',
                'postcss-loader',
                {
+                    loader:'px2rem-loader',
+                    options:{
+                        remUnit:75,
+                        remPrecesion:8
+                    }
+                }]
+            },
```
## 4.polyfill
- @babel/preset-env 会根据预设的浏览器兼容列表从stage-4选取必须的plugin,也就是说，不引入别的stage-x,@babel/preset-env将只支持到stage-4
- 三个概念
    - 最新ES 语法： 比如，箭头函数
    - 最新ES API：, 比如,Promise
    - 最新ES实例方法：比如，String.prototype.includes
### 4.1 babel-polyfill
- Babel默认只转换新的javascript语法，而不转换新的API，比如Iterator,Generator,Set,Maps,Proxy,Reflect,Symbol,Promise等全局对象。以及一些在全局对象上的方法（比如Object.assign）都不会转码。
- 比如说，ES6在Array对象上新增了Array.from方法，Babel就不会转码这个方法，如果想让这个方法运行，必须使用babel-polyfill来转换等
- babel-polyfill 它是通过向全局对象和内置对象的prototype上添加方法来实现的。比如运行环境中不支持Array.prototype.find方法，引入polyfill,我们就可以使用es6方法来编写了，但是缺点就是会造成全局空间污染
- [@babel/@babel/preset-env](https://www.npmjs.com/package/@babel/preset-env)为每一个环境的预设
- @babel/preset-env默认支持语法转化，需要开启 useBuiltlns 配置才能转化API和实例方法
- useBuiltlns 可选值包括: "usage"|"entry"|false,默认为false,表示不对polyfills处理，这个配置是引入polyfills的关键
#### 4.1.1 安装
```sh
npm i @babel/polyfill
```
#### 4.1.2 "useBuiltlns": fasle
- "useBuiltlns": false 此时不对polyfill做操作，如果引入 @babel/polyfill，则无视配置的浏览器兼容，引入所有的 polyfill
- 86.4 KiB
```js
import "@babel/polyfill";
```
```js
 {
    test: /\.jsx?$/,
    exclude: /node_modules/,
    use: {
        loader: 'babel-loader',
        options: {
            presets: [["@babel/preset-env", {
+                           useBuiltIns: false,
            }], "@babel/preset-react"],
            plugins: [
                ["@babel/plugin-proposal-decorators", { legacy: true }],
                ["@babel/plugin-proposal-class-properties", { loose: true }]
            ]
        }

    }
},
```
#### 4.1.3 "useBuiltlns": "entry"
- 在项目入口引入一次(多次引入会报错)
- "useBuiltlns": "entry"根据配置的浏览器兼容，引入浏览器不兼容的polyfill。需要在入口文件手动添加 import '@babel/polyfill', 会自动根据browserslist替换成浏览器不兼容的所有polyfill
- 这里需要指定core-js的版本，如果"corejs":3,则import "@babel/polyfill"需要改成 import "core-js/stable"; import "regenerator-runtime/runtime";
- corejs 默认是2，配置2的话需要单独安装core-js@3
- 80.6KiB
- 10.7KiB
```js
import "@babel/polyfill";
```
```sh
npm i core-js@3
```
```js
import "core-js/stable";
import "regenerator-runtime/runtime";
```
```js
{
                test: /\.jsx?$/,
                exclude: /node_modules/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: [["@babel/preset-env", {
+                           useBuiltIns: 'entry',
+                           corejs: { version: 2 }
                        }], "@babel/preset-react"],
                        plugins: [
                            ["@babel/plugin-proposal-decorators", { legacy: true }],
                            ["@babel/plugin-proposal-class-properties", { loose: true }]
                        ]
                    }

                }
},
```
```json
{
    "browserslist": {
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ],
    "production": [
      ">1%"
    ]
  },
}
```
#### 4.1.4 "useBuiltlns": "usage"
- "useBuiltlns": "usage" usage会根据配置的浏览器兼容，以及你代码中用到的API来进行polyfill，实现了按需添加
- 当设置为usage时，polyfills会自动按需添加，不再需要手工引入@babel/polyfill
- usage的行为类似 babel-transform-runtime，不会造成全局污染，因此也会不会对类似Array.prototype.includes()进行polyfill
- 0 bytes
- 8.98 KiB
```js
import "@babel/polyfill";
console.log(Array.from([]);
```
```js
 {
    test: /\.jsx?$/,
    exclude: /node_modules/,
    use: {
        loader: 'babel-loader',
        options: {
            presets: [["@babel/preset-env", {
+               useBuiltIns: 'usage',
+               corejs: { version: 3 }
            }], "@babel/preset-react"],
            plugins: [
                ["@babel/plugin-proposal-decorators", { legacy: true }],
                ["@babel/plugin-proposal-class-properties", { loose: true }]
            ]
        }
    }
},
```
### 4.2 babel-runtime
- Babel为了解决全局空间污染的问题，提供了单独的包[babel-runtime](https://babeljs.io/docs/en/babel-runtime)用以提供编译模块的工具函数
- 简单说 babel-runtime 更像是一种按需加载的实现，比如你哪里需要使用Promise,只要在这个文件头部import Promise from "babel-runtime/core-js/promise"就行了
```sh
npm i babel-runtime -D
```
```js
import Promise from "babel-runtime/core-js/promise";
const p = new Promise(() => {

});
console.log(p);
```
### 4.3 babel-plugin-transform-runtime
- @babel/plugin-transform-runtime 插件是为了解决
    - 多个文件重复引用相同helpers(帮助函数) -> 提取运行时
    - 新API方法全局污染 -> 局部引入
- 启用插件 babel-plugin-transform-runtime后，Babel就会使用 bebel-runtime下的工具函数
- babel-plugin-transform-runtime 插件能够将这些工具函数的代码转换成require语句，指向为对 babel-runtime的引用
- babel-plugin-transform-runtime 就是可以在我们使用新 API 时自动 import babel-runtime 里面的 polyfill
    - 当我们使用 async/await 时，自动引入 babel-runtime/regenerator
    - 当我们使用 ES6 的静态事件或内置对象时，自动引入 babel-runtime/core-js
    - 移除内联babel helpers并替换使用babel-runtime/helpers 来替换
- corejs默认是3,配置2的话需要单独安装@babel/runtime-corejs2
```sh
npm i @babel/runtime-corejs2 -D
```
```js
      {
        test: /\.jsx?$/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ["@babel/preset-env",'@babel/preset-react'],
            plugins: [
+              [
+                "@babel/plugin-transform-runtime",
+                {
+                  corejs: 2,//当我们使用 ES6 的静态事件或内置对象时自动引入 babel-runtime/core-js
+                  helpers: true,//移除内联babel helpers并替换使用babel-runtime/helpers 来替换
+                  regenerator: true,//是否开启generator函数转换成使用regenerator runtime来避免污染全局域
+                },
+              ],
              ['@babel/plugin-proposal-decorators', { legacy: true }],
              ['@babel/plugin-proposal-class-properties', { loose: true }],
            ],
          },
        },
      },
```
corejs: 2 corejs 2=>false 131 KiB => 224 bytes
```js
const p = new Promise(()=> {});
console.log(p);
```
helpers true=>false 160 KiB=>150 KiB
```js
class A {

}
class B extends A {

}
console.log(new B());
```
regenerator false=>true B490 bytes->28.6 Ki
```js
function* gen() {

}
console.log(gen());
```
### 4.4 最佳实践
- babel-runtime 适合在组件和类库项目中使用，而babel-polyfill 适合在业务项目中使用。
### 4.5 polyfill-service
- [polyfill.io](https://polyfill.io/v3/)自动化的JavaScript Polyfill服务
- [polyfill.io](https://polyfill.io/v3/)通过分析请求头信息中的UserAgent实现自动加载浏览器所需的polyfills
```html
<script src="https://polyfill.io/v3/polyfill.min.js"></script>
```