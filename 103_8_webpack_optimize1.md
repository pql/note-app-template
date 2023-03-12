## 1.缩小范围
### 1.1 extensions
指定extension之后可以不用在 require 或是 input 的时候加文件扩展名，会依次尝试添加扩展名进行匹配
```js
resolve: {
  extensions: [".js",".jsx",".json",".css"]
},
```
### 1.2 alias
配置别名可以加快webpack查找模块的速度
- 每当引入bootstrap模块的时候，它会直接引入bootstrap,而不需要从node_modules文件夹中按模块的查找规则查找
```js
const bootstrap = path.resolve(__dirname,'node_modules/_bootstrap@3.3.7@bootstrap/dist/css/bootstrap.css');
resolve: {
+    alias:{
+        "bootstrap":bootstrap
+    }
},
```
### 1.3 modules
- 对于直接声明依赖名的模块（如 react ），webpack 会类似 Node.js 一样进行路径搜索，搜索node_modules目录
- 这个目录就是使用resolve.modules字段进行配置的 默认配置
```js
resolve: {
    modules: ['node_modules'],
}
```
如果可以确定项目内所有的第三方依赖模块都是在项目根目录下的 node_modules 中的话
```js
resolve: {
    modules: [path.resolve(__dirname, 'node_modules')],
}
```
### 1.4 mainFields 
默认情况下package.json 文件则按照文件中 main 字段的文件名来查找文件
```js
resolve: {
  // 配置 target === "web" 或者 target === "webworker" 时 mainFields 默认值是：
  mainFields: ['browser', 'module', 'main'],
  // target 的值为其他时，mainFields 默认值为：
  mainFields: ["module", "main"],
}
```
### 1.5 mainFiles
当目录下没有 package.json 文件时，我们说会默认使用目录下的 index.js 这个文件，其实这个也是可以配置的
```js
resolve: {
  mainFiles: ['index'], // 你可以添加其他默认使用的文件名
},
```
### 1.6 resolveLoader
resolve.resolveLoader用于配置解析 loader 时的 resolve 配置,默认的配置：
```js
module.exports = {
  resolveLoader: {
    modules: [ 'node_modules' ],
    extensions: [ '.js', '.json' ],
    mainFields: [ 'loader', 'main' ]
  }
};
```
## 2. noParse
- module.noParse 字段，可以用于配置哪些模块文件的内容不需要进行解析
- 不需要解析依赖（即无依赖） 的第三方大型类库等，可以通过这个字段来配置，以提高整体的构建速度
```js
module.exports = {
    // ...
    module: {
        noParse: /jquery|lodash/, // 正则表达式
        // 或者使用函数
        noParse(content) {
            return /jquery|lodash/.test(content)
        },
    }
}...
```
> 使用 noParse 进行忽略的模块文件中不能使用 import、require、define 等导入机制

## 3.DefinePlugin
DefinePlugin创建一些在编译时可以配置的全局常量
```js
let webpack = require('webpack');
new webpack.DefinePlugin({
    PRODUCTION: JSON.stringify(true),
    VERSION: "1",
    EXPRESSION: "1+2",
    COPYRIGHT: {
        AUTHOR: JSON.stringify("珠峰培训")
    }
})
```
```js
console.log(PRODUCTION);
console.log(VERSION);
console.log(EXPRESSION);
console.log(COPYRIGHT);
```
- 如果配置的值是字符串，那么整个字符串会被当成代码片段来执行，其结果作为最终变量的值
- 如果配置的值不是字符串，也不是一个对象字面量，那么该值会被转为一个字符串，如 true，最后的结果是 'true'
- 如果配置的是一个对象字面量，那么该对象的所有 key 会以同样的方式去定义
- JSON.stringify(true) 的结果是 'true'
## 4.IgnorePlugin
IgnorePlugin用于忽略某些特定的模块，让 webpack 不把这些指定的模块打包进去
### 4.1 src/index.js
```js
import moment from  'moment';
import 'moment/locale/zh-cn'
console.log(moment().format('MMMM Do YYYY, h:mm:ss a'));
```
### 4.2 webpack.config.js 
```js
import moment from  'moment';
console.log(moment);
```
```js
new webpack.IgnorePlugin({
    //A RegExp to test the context (directory) against.
    contextRegExp: /moment$/,
    //A RegExp to test the request against.
    resourceRegExp: /^\.\/locale/
new MiniCSSExtractPlugin({
    filename:'[name].css'
})
/*  new webpack.IgnorePlugin( {
    //A filter function for resource and context.
    checkResource: (resource, context) => {
        if(/moment$/.test(context)){
            if(/^\.\/locale/.test(resource)){
                if(!(/zh-cn/.test(resource))){
                    return true;
                }
            }
        }
        return false;
    } 
})*/
```
- 第一个是匹配引入模块路径的正则表达式
- 第二个是匹配模块的对应上下文，即所在目录名
## 7.日志优化
- 日志太多太少都不美观
- 可以修改stats

| 预设 | 替代 | 描述 |
| --- | --- | --- |
| errors-only | none | 只在错误时输出 |
| minimal | none | 发生错误和新的编译时输出 |
| none | false | 没有输出 |
| normal | true | 标准输出 |
| verbose | none | 全部输出 |

### 7.1 friendly-errors-webpack-plugin
- [friendly-errors-webpack-plugin](https://www.npmjs.com/package/friendly-errors-webpack-plugin)
- success 构建成功的日志提示
- warning 构建警告的日志提示
- error 构建报错的日志提示
```sh
npm i friendly-errors-webpack-plugin
```
```js
+ stats:'verbose',
  plugins:[
+   new FriendlyErrorsWebpackPlugin()
  ]
```
> 编译完成后可以通过echo $?获取错误码，0为成功，非0为失败
## 8. 日志输出
```json
  "scripts": {
    "build": "webpack",
+    "build:stats":"webpack --json > stats.json",
    "dev": "webpack-dev-server --open"
  },
```
```js
const webpack = require('webpack');
const config = require('./webpack.config.js');
webpack(config,(err,stats)=>{
  if(err){
    console.log(err);
  }
  if(stats.hasErrors()){
    return console.error(stats.toString("errors-only"));
  }
  console.log(stats);
});
```
## 9. 费时分析
```js
const SpeedMeasureWebpackPlugin = require('speed-measure-webpack-plugin');
const smw = new SpeedMeasureWebpackPlugin();
module.exports =smw.wrap({});
```
## 10.webpack-bundle-analyzer
- 是一个webpack的插件，需要配合webpack和webpack-cli一起使用。这个插件的功能是生成代码分析报告，帮助提升代码质量和网站性能
```sh
cnpm i webpack-bundle-analyzer -D
```
```js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
module.exports={
  plugins: [
    new BundleAnalyzerPlugin()  // 使用默认配置
    // 默认配置的具体配置项
    // new BundleAnalyzerPlugin({
    //   analyzerMode: 'server',
    //   analyzerHost: '127.0.0.1',
    //   analyzerPort: '8888',
    //   reportFilename: 'report.html',
    //   defaultSizes: 'parsed',
    //   openAnalyzer: true,
    //   generateStatsFile: false,
    //   statsFilename: 'stats.json',
    //   statsOptions: null,
    //   excludeAssets: null,
    //   logLevel: info
    // })
  ]
}
```
```json
{
 "scripts": {
    "dev": "webpack --config webpack.dev.js --progress"
  }
}
```
webpack.config.js
```js
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
module.exports={
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'disabled', // 不启动展示打包报告的http服务器
      generateStatsFile: true, // 是否生成stats.json文件
    }),
  ]
}
```
```json
{
 "scripts": {
    "generateAnalyzFile": "webpack --profile --json > stats.json", // 生成分析文件
    "analyz": "webpack-bundle-analyzer --port 8888 ./dist/stats.json" // 启动展示打包报告的http服务器
  }
}
```
```sh
npm run generateAnalyzFile
npm run analyz
```
## 11. libraryTarget 和 library
- [outputlibrarytarget](https://webpack.js.org/configuration/output/#outputlibrarytarget)
- 当用 Webpack 去构建一个可以被其他模块导入使用的库时需要用到它们
- output.library 配置导出库的名称
- output.libraryExport 配置要导出的模块中哪些子模块需要被导出。 它只有在 output.libraryTarget 被设置成 commonjs 或者 commonjs2 时使用才有意义
- output.libraryTarget 配置以何种方式导出库,是字符串的枚举类型，支持以下配置

| libraryTarget | 使用者的引入方式 | 使用者提供给被使用者的模块的方式 |
| --- | --- | --- |
| var | 只能以script标签的形式引入我们的库 | 只能以全局变量的形式提供这些被依赖的模块 |
| commonjs | 只能按照commonjs的规范引入我们的库 | 被依赖模块需要按照commonjs规范引入 |
| commonjs2 | 只能按照commonjs2的规范引入我们的库 | 被依赖模块需要按照commonjs2规范引入 |
| amd | 只能按amd规范引入 | 被依赖的模块需要按照amd规范引入 |
| this | | |
| window | | |
| global | | |
| umd | 可以用script、commonjs、amd引入 | 按对应的方式引入 |

### 11.1 var(默认)
编写的库将通过var被赋值给通过library指定名称的变量。
#### 11.1.1 webpack.config.js
```js
{
  output: {
        path: path.resolve("build"),
        filename: "[name].js",
+       library:'calculator',
+       libraryTarget:'var'
  }
}
```
#### 11.1.2 index.js
```js
module.exports =  {
    add(a,b) {
        return a+b;
    }
}
```
#### 11.1.3 bundle.js
```js
var calculator=(function (modules) {}({})
```
#### 11.1.4 index.html
```html
<script src="bundle.js"></script>
<script>
    let ret = calculator.add(1,2);
    console.log(ret);
</script>
```
### 11.2 commonjs
- 编写的库将通过 CommonJS 规范导出
#### 11.2.1 导出方式
```js
exports["calculator"] = (function (modules) {}({})
```
#### 11.2.2 使用方式
```js
let main = require('./main');
console.log(main.calculator.add(1,2));
```
```js
require('npm-name')['calculator'].add(1,2);
```
> npm-name是指模块发布到 Npm 代码仓库时的名称
### 11.3 commonjs2
- 编写的库将通过 CommonJS 规范导出。
#### 11.3.1 导出方式
```js
module.exports = (function (modules) {}({})
```
#### 11.3.2 使用方式
```js
require('npm-name').add();
```
> 在 output.libraryTarget 为 commonjs2 时，配置 output.library 将没有意义。
### 11.4 this
- 编写的库将通过 this 被赋值给通过library 指定的名称，输出和使用的代码如下：
#### 11.4.1 导出方式
```js
this["calculator"]= (function (modules) {}({})
```
#### 11.4.2 使用方式
```js
this.calculator.add();
```
### 11.5 window
- 编写的库将通过 window 被赋值给通过 library 指定的名称，即把库挂载到 window 上，输出和使用的代码如下
#### 11.5.1 导出方式
```js
window["calculator"]= (function (modules) {}({})
```
#### 11.5.2 使用方式
```js
window.calculator.add();
```
### 11.6 global
- 编写的库将通过 global 被赋值给通过 library 指定的名称，即把库挂载到 global 上，输出和使用的代码如下：
#### 11.6.1 导出方式
```js
global["calculator"]= (function (modules) {}({})
```
#### 11.6.2 使用方式
```js
global.calculator.add();
```
### 11.7 umd
```js
(function webpackUniversalModuleDefinition(root, factory) {
  if(typeof exports === 'object' && typeof module === 'object')
    module.exports = factory();
  else if(typeof define === 'function' && define.amd)
    define([], factory);
  else if(typeof exports === 'object')
    exports['MyLibrary'] = factory();
  else
    root['MyLibrary'] = factory();
})(typeof self !== 'undefined' ? self : this, function() {
  return _entry_return_;
});
```