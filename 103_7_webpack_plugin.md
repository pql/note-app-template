## 1.plugin
插件向第三方开发者提供了 webpack 引擎中完整的能力。使用阶段式的构建回调，开发者可以引入它们自己的行为到 webpack 构建流程中。创建插件比创建 loader 更加高级，因为你将需要理解一些 webpack 底层的内部特性来做相应的钩子
### 1.1 为什么需要一个插件
- webpack 基础配置无法满足需求
- 插件几乎能够任意更改 webpack 编译结果
- webpack 内部也是通过大量内部插件实现的
### 1.2 可以加载插件的常用对象

| 对象 | 钩子 |
| [Compiler](https://github.com/webpack/webpack/blob/v4.39.3/lib/Compiler.js) | run,compile,compilation,make,emit,done |
| [Compilation](https://github.com/webpack/webpack/blob/v4.39.3/lib/Compilation.js) | buildModule,normalModuleLoader,succeedModule,finishModules,seal,optimize,after-seal |
| [ModuleFactory](https://github.com/webpack/webpack/blob/master/lib/ModuleFactory.js) | beforeResolver,afterResolver,module,parser |
| Module | |
| [Parser](https://github.com/webpack/webpack/blob/master/lib/Parser.js) | program,statement,call,expression |
| [Template](https://github.com/webpack/webpack/blob/master/lib/Template.js) | hash,bootstrap,localVars,render |

## 2.创建插件
- 插件是一个类
- 类上有一个apply的实例方法
- apply的参数是compiler
```js
class DonePlugin {
    constructor(options) {
        this.options = options;
    }
    apply(compiler) {

    }
}
module.exports = DonePlugin;
```

## 3.Compiler和Compilation
在插件开发中最重要的两个资源就是 compiler 和 compilation 对象。理解它们的角色是扩展 webpack 引擎重要的第一步。
- compiler 对象代表了完整的 webpack 环境配置。这个对象在启动webpack时被一次性建立，并配置好所有可操作的设置，包括options,loader和plugin。当在webpack环境中应用一个插件时，插件均收到此compiler对象的引用。可以使用它来访问webpack的主环境。
- compilation 对象代表了一次资源版本构建。当运行 webpack 开发环境中间件时，每当检测到一个文件变化，就会创建一个新的 compilation，从而生成一组新的编译资源。一个 compilation 对象表现了当前的模块资源、编译生成资源、变化的文件、以及被跟踪依赖的状态信息。compilation 对象也提供了很多关键时机的回调，以供插件做自定义处理时选择使用。
## 4. 基本插件架构
- 插件是由「具有 apply 方法的 prototype 对象」所实例化出来的
- 这个 apply 方法在安装插件时，会被 webpack compiler 调用一次
- apply 方法可以接收一个 webpack compiler 对象的引用，从而可以在回调函数中访问到 compiler 对象
### 4.1 使用插件代码
- [使用插件](https://github.com/webpack/webpack/blob/master/lib/webpack.js#L60-L69)
```js
if (options.plugins && Array.isArray(options.plugins)) {
  for (const plugin of options.plugins) {
    plugin.apply(compiler);
  }
}
```
### 4.2 Compiler 插件
- [done: new AsyncSeriesHook(["stats"])](https://github.com/webpack/webpack/blob/master/lib/Compiler.js#L105)
#### 4.2.1 同步
```js
class DonePlugin {
  constructor(options) {
    this.options = options;
  }
  apply(compiler) {
    compiler.hooks.done.tap("DonePlugin", (stats) => {
      console.log("Hello ", this.options.name);
    });
  }
}
module.exports = DonePlugin;
```
#### 4.2.2 异步
```js
class DonePlugin {
  constructor(options) {
    this.options = options;
  }
  apply(compiler) {
    compiler.hooks.done.tapAsync("DonePlugin", (stats, callback) => {
      console.log("Hello ", this.options.name);
      callback();
    });
  }
}
module.exports = DonePlugin;
```
### 4.3 使用插件
- 要安装这个插件，只需要在你的 webpack 配置的 plugin 数组中添加一个实例
```js
const DonePlugin = require("./plugins/DonePlugin");
module.exports = {
  entry: "./src/index.js",
  output: {
    path: path.resolve("build"),
    filename: "bundle.js",
  },
  plugins: [new DonePlugin({ name: "zhufeng" })],
};
```
### 4.4 触发钩子执行
- [done](https://github.com/webpack/webpack/blob/master/lib/Compiler.js#L360-L363)
```js
if (this.hooks.shouldEmit.call(compilation) === false) {
                const stats = new Stats(compilation);
                stats.startTime = startTime;
                stats.endTime = Date.now();
+                this.hooks.done.callAsync(stats, err => {
+                    if (err) return finalCallback(err);
+                    return finalCallback(null, stats);
+                });
                return;
            }
```
## 5. compilation 插件
- 使用 compiler 对象时，你可以绑定提供了编译 compilation 引用的回调函数，然后拿到每次新的 compilation 对象。这些 compilation 对象提供了一些钩子函数，来钩入到构建流程的很多步骤中
### 5.1 asset-plugin.js
plugins\asset-plugin.js
```js
class AssetPlugin{
    constructor(options){
        this.options = options;
    }
    apply(compiler){
        //compiler只有一个,每当监听到文件的变化,就会创建一个新的compilation
        //每当compiler开启一次新的编译,就会创建一个新的compilation,触发一次compilation事件 
        compiler.hooks.compilation.tap('AssetPlugin',(compilation)=>{
            // main=>main.js  vendor=>vendor.js
            compilation.hooks.chunkAsset.tap('AssetPlugin',(chunk,filename)=>{
                console.log(chunk,filename);
            });
        });
    }
}

module.exports = AssetPlugin;
```
### 5.2 compilation.call
- [Compiler.js](https://github.com/webpack/webpack/blob/master/lib/Compiler.js#L853-L860)
```js
newCompilation(params) {
    const compilation = this.createCompilation();
    this.hooks.compilation.call(compilation, params);
    return compilation;
}
```
### 5.3 chunkAsset.call
- [chunkAsset.call](https://github.com/webpack/webpack/blob/v4.39.3/lib/Compilation.js#L2019)
```js
chunk.files.push(file);
+this.hooks.chunkAsset.call(chunk, file);
```
> 关于 compiler, compilation 的可用回调，和其它重要的对象的更多信息，请查看 [插件](https://webpack.docschina.org/api/compiler-hooks/) 文档。

## 6.打包zip
- [webpack-sources](https://www.npmjs.com/package/webpack-sources)
### 6.1 zip-plugin.js
plugins\zip-plugin.js
```js
const { RawSource } = require("webpack-sources");
const JSZip = require("jszip");
const path = require("path");
class ZipPlugin {
  constructor(options) {
    this.options = options;
  }
  apply(compiler) {
    let that = this;
    compiler.hooks.emit.tapAsync("ZipPlugin", (compilation, callback) => {
      var zip = new JSZip();
      for (let filename in compilation.assets) {
        const source = compilation.assets[filename].source();
        zip.file(filename, source);
      }
      zip.generateAsync({ type: "nodebuffer" }).then((content) => {
        compilation.assets[that.options.filename] = new RawSource(content);
        callback();
      });
    });
  }
}
module.exports = ZipPlugin;
```
### 6.2 webpack.config.js
webpack.config.js
```js
  plugins: [
+    new zipPlugin({
+      filename: 'assets.zip'
+    })
]
```
## 7.自动外链
### 7.1 使用外部类库
- 手动指定 external
- 手动引入 script
> 能否检测代码中的 import 自动处理这个步骤?
```js
{
  externals:{
    //key jquery是要require或import 的模块名,值 jQuery是一个全局变量名
  'jquery':'$'
}, 
  module:{}
}
```
### 7.2 思路
- 解决 import 自动处理external和script的问题，需要怎么实现，该从哪方面开始考虑
- 依赖 当检测到有import该library时，将其设置为不打包类似exteral,并在指定模版中加入 script,那么如何检测 import？这里就用Parser
- external依赖 需要了解 external 是如何实现的，webpack 的 external 是通过插件ExternalsPlugin实现的，ExternalsPlugin 通过tap NormalModuleFactory 在每次创建 Module 的时候判断是否是ExternalModule
- webpack4 加入了模块类型之后，Parser获取需要指定类型 moduleType,一般使用javascript/auto即可
### 7.3 使用plugins
```js
plugins: [
  new HtmlWebpackPlugin({
    template: "./src/index.html",
    filename: "index.html",
  }),
  new AutoExternalPlugin({
    jquery: {
      expose: "$",
      url: "https://cdn.bootcss.com/jquery/3.1.0/jquery.js",
    },
  }),
];
```
### 7.4 AutoExternalPlugin
- [ExternalsPlugin.js](https://github.com/webpack/webpack/blob/0d4607c68e04a659fa58499e1332c97d5376368a/lib/ExternalsPlugin.js)
- [ExternalModuleFactoryPlugin](https://github.com/webpack/webpack/blob/eeafeee32ad5a1469e39ce66df671e3710332608/lib/ExternalModuleFactoryPlugin.js)
- [ExternalModule.js](https://github.com/webpack/webpack/blob/eeafeee32ad5a1469e39ce66df671e3710332608/lib/ExternalModule.js)
- [parser](https://github.com/zhufengnodejs/webpack-analysis/blob/master/node_modules/_webpack%404.20.2%40webpack/lib/NormalModuleFactory.js#L87)
- [factory](https://github.com/zhufengnodejs/webpack-analysis/blob/master/node_modules/_webpack%404.20.2%40webpack/lib/NormalModuleFactory.js#L66)
- [htmlWebpackPluginAlterAssetTags](https://github.com/jantimon/html-webpack-plugin/blob/v3.2.0/index.js#L62)

AsyncSeriesBailHook factorize
```js
let { AsyncSeriesBailHook } = require("tapable");
let factorize = new AsyncSeriesBailHook(["resolveData"]);
factorize.tapAsync("tap1", (resolveData, callback) => {
    if (resolveData === "jquery") {
        callback(null, { externalModule: "jquery" });
    } else {
        callback();
    }
});
factorize.tapAsync("tap2", (resolveData, callback) => {
    callback(null, { normalModule: resolveData });
});
//由tap1返回
factorize.callAsync("jquery", (err, module) => {
    console.log(module);
});
//由tap2返回
factorize.callAsync("jquery2", (err, module) => {
    console.log(module);
});
```
plugins\auto-external-plugin.js
```js
/**
 * https://webpack.js.org/api/compiler-hooks/#normalmodulefactory
 * 1.通过AST语法树检测当前的项目脚本中引入了哪些模块,是不是引入了jquery
 * 2.如果发现引入了,则要自动插入CDN脚本
 */
const {ExternalModule} = require('webpack');
const HtmlWebpackPlugin = require('html-webpack-plugin');
class AutoExternalPlugin {
  constructor(options) {
    this.options = options;
    this.externalModules = Object.keys(this.options);// ['jquery','lodash']
    this.importedModules = new Set();//存放着所有导入的外部依赖模块
  }
  apply(compiler) {
    //每种模块都会有一个对应的模块工厂来创建这个模块,普通模块对应的工作就是普通模块工厂
    compiler.hooks.normalModuleFactory.tap('AutoExternalPlugin', (normalModuleFactory) => {
      normalModuleFactory.hooks.parser
        .for('javascript/auto')//parser babel esprima  acorn 可以把源代码转成抽象语法树,然后进行遍历,
        //遍历到不同类型的节点会触发不同的钩子,执行钩子对应的事件函数
        .tap('AutoExternalPlugin', parser => {
          parser.hooks.import.tap('AutoExternalPlugin',(statement,source)=>{
            //console.log(statement,source);
            if(this.externalModules.includes(source)){//jquery
              this.importedModules.add(source);
            }
          }); 
          //拦截对require的方法调用
          parser.hooks.call.for('require').tap('AutoExternalPlugin',(expression)=>{
            console.log(expression);
            let value = expression.arguments[0].value;
            if(this.externalModules.includes(value)){//jquery
              this.importedModules.add(value);
            }
          })
        })
       //改造创建模块的过程
       normalModuleFactory.hooks.factorize.tapAsync('AutoExternalPlugin',(resolveData, callback)=>{
          let request = resolveData.request;//jquery
          if(this.externalModules.includes(request)){//如果这个模块是一个外部模块的话,进行拦截
            let expose = this.options[request].expose;
            //创建一个外部模块并返回 jquery = window.jQuery
            //模块不一样,打包不一样
            callback(null,new ExternalModule(expose,'window',request));
          }else{
            callback();//如果是正常模块,会直接调用callback向后执行
          }
       });
    });
    compiler.hooks.compilation.tap('AutoExternalPlugin',(compilation)=>{
      //改变资源标签
      HtmlWebpackPlugin.getHooks(compilation).alterAssetTags.tapAsync('AutoExternalPlugin',(htmlData,callback)=>{
        console.log('htmlData',htmlData);
        let {assetTags} = htmlData;
        //找了我实际引入了哪些模块 jquery lodash
        let importedExternalModules = Object.keys(this.options).filter(item=>this.importedModules.has(item));
        importedExternalModules.forEach(key=>{
          assetTags.scripts.unshift({
            tagName:'script',
            voidTag:false,
            attributes:{
              src:this.options[key].url,
              defer:false
            }
          });
        });
        callback(null,htmlData);
      });
    });
  }
}

module.exports = AutoExternalPlugin;
/**
 * Node {
  type: 'ImportDeclaration',
  start: 0,
  end: 23,
  loc: SourceLocation {
    start: Position { line: 1, column: 0 },
    end: Position { line: 1, column: 23 }
  },
  range: [ 0, 23 ],
  specifiers: [
    Node {
      type: 'ImportDefaultSpecifier',
      start: 7,
      end: 8,
      loc: [SourceLocation],
      range: [Array],
      local: [Node]
    }
  ],
  source: Node {
    type: 'Literal',
    start: 14,
    end: 22,
    loc: SourceLocation { start: [Position], end: [Position] },
    range: [ 14, 22 ],
    value: 'jquery',
    raw: "'jquery'"
  }
}
jquery
 */
```
## 8.HashPlugin
- 可以自己修改各种hash值
```js
class HashPlugin{
    constructor(options){
        this.options = options;
    }
    apply(compiler){
        compiler.hooks.compilation.tap('HashPlugin',(compilation,params)=>{
            //如果你想改变hash值，可以在hash生成这后修改
            compilation.hooks.afterHash.tap('HashPlugin',()=>{
                let fullhash = 'fullhash';//时间戳
                console.log('本次编译的compilation.hash',compilation.hash);
                compilation.hash= fullhash;//output.filename [fullhash]
                for(let chunk of compilation.chunks){
                    console.log('chunk.hash',chunk.hash);
                    chunk.renderedHash = 'chunkHash';//可以改变chunkhash
                    console.log('chunk.contentHash',chunk.contentHash);
                    chunk.contentHash= { javascript: 'javascriptContentHash','css/mini-extract':'cssContentHash' }
                }
            });
        });
    }
}
module.exports = HashPlugin;
/**
 * 三种hash
 * 1. hash compilation.hash 
 * 2. chunkHash 每个chunk都会有一个hash
 * 3. contentHash 内容hash 每个文件会可能有一个hash值
 */
```
webpack.config.js
```js
const path = require('path');
const DonePlugin = require('./plugins/DonePlugin');
const AssetPlugin = require('./plugins/AssetPlugin');
const ZipPlugin = require('./plugins/ZipPlugin');
const HashPlugin = require('./plugins/HashPlugin');
const AutoExternalPlugin = require('./plugins/AutoExternalPlugin');
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
module.exports = {
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [
+                   MiniCssExtractPlugin.loader,
                    'css-loader'
                ]
            }
        ]
    },
    plugins: [
+       new HashPlugin(),
    ]
}
```
## 9. AsyncQueue
### 9.1 AsyncQueue
```js
let AsyncQueue = require('webpack/lib/util/AsyncQueue');
let AsyncQueue = require('./AsyncQueue');
function processor(item, callback) {
    setTimeout(() => {
        console.log('process',item);
        callback(null, item);
    }, 3000);
}
const getKey = (item) => {
    return item.key;
}
let queue  = new AsyncQueue({
    name:'createModule',parallelism:3,processor,getKey
});
const start = Date.now();
let item1 = {key:'module1'};
queue.add(item1,(err,result)=>{
    console.log(err,result);
    console.log(Date.now() - start);
});
queue.add(item1,(err,result)=>{
    console.log(err,result);
    console.log(Date.now() - start);
});
queue.add({key:'module2'},(err,result)=>{
    console.log(err,result);
    console.log(Date.now() - start);
});
queue.add({key:'module3'},(err,result)=>{
    console.log(err,result);
    console.log(Date.now() - start);
});
queue.add({key:'module4'},(err,result)=>{
    console.log(err,result);
    console.log(Date.now() - start);
});
```
### 9.2 use.js
use.js
```js
const QUEUED_STATE = 0;//已经 入队，待执行
const PROCESSING_STATE = 1;//处理中
const DONE_STATE = 2;//处理完成
class ArrayQueue {
    constructor() {
        this._list = [];
    }
    enqueue(item) {
        this._list.push(item);//[1,2,3]
    }
    dequeue() {
        return this._list.shift();//移除并返回数组中的第一个元素
    }
}
class AsyncQueueEntry {
    constructor(item, callback) {
        this.item = item;//任务的描述
        this.state = QUEUED_STATE;//这个条目当前的状态
        this.callback = callback;//任务完成的回调
    }
}
class AsyncQueue {
    constructor({ name, parallelism, processor, getKey }) {
        this._name = name;//队列的名字
        this._parallelism = parallelism;//并发执行的任务数
        this._processor = processor;//针对队列中的每个条目执行什么操作
        this._getKey = getKey;//函数，返回一个key用来唯一标识每个元素
        this._entries = new Map();
        this._queued = new ArrayQueue();//将要执行的任务数组队列 
        this._activeTasks = 0;//当前正在执行的数，默认值1
        this._willEnsureProcessing = false;//是否将要开始处理
    }
    add = (item, callback) => {
        const key = this._getKey(item);//获取这个条目对应的key
        const entry = this._entries.get(key);//获取 这个key对应的老的条目
        if (entry !== undefined) {
            if (entry.state === DONE_STATE) {
                process.nextTick(() => callback(entry.error, entry.result));
            } else if (entry.callbacks === undefined) {
                entry.callbacks = [callback];
            } else {
                entry.callbacks.push(callback);
            }
            return;
        }
        const newEntry = new AsyncQueueEntry(item, callback);//创建一个新的条目
        this._entries.set(key, newEntry);//放到_entries
        this._queued.enqueue(newEntry);//把这个新条目放放队列
        if (this._willEnsureProcessing === false) {
            this._willEnsureProcessing = true;
            setImmediate(this._ensureProcessing);
        }
    }
    _ensureProcessing = () => {
        //如果当前的激活的或者 说正在执行任务数行小于并发数
        while (this._activeTasks < this._parallelism) {
            const entry = this._queued.dequeue();//出队 先入先出
            if (entry === undefined) break;
            this._activeTasks++;//先让正在执行的任务数++
            entry.state = PROCESSING_STATE;//条目的状态设置为执行中
            this._startProcessing(entry);
        }
        this._willEnsureProcessing = false;
    }
    _startProcessing = (entry) => {
        this._processor(entry.item, (e, r) => {
            this._handleResult(entry, e, r);
        });
    }
    _handleResult = (entry, error, result) => {
        const callback = entry.callback;
        const callbacks = entry.callbacks;
        entry.state = DONE_STATE;//把条目的状态设置为已经完成
        entry.callback = undefined;//把callback
        entry.callbacks = undefined;
        entry.result = result;//把结果赋给entry
        entry.error = error;//把错误对象赋给entry
        callback(error, result);
        if (callbacks !== undefined) {
            for (const callback of callbacks) {
                callback(error, result);
            }
        }
        this._activeTasks--;
        if (this._willEnsureProcessing === false) {
            this._willEnsureProcessing = true;
            setImmediate(this._ensureProcessing);
        }
    }
}
module.exports = AsyncQueue;
```
## 11.参考
- [Node.js SDK](https://developer.qiniu.com/kodo/sdk/1289/nodejs)
- [writing-a-plugin](https://webpack.js.org/contribute/writing-a-plugin/)
- [api/plugins](https://webpack.js.org/api/plugins/)