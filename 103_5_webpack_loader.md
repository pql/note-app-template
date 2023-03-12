## 1.loader
- 所谓 loader 只是一个导出为函数的 JavaScript 模块。它接收上一个 loader 产生的结果或者资源文件(resource file)作为入参。也可以用多个loader函数组成 loader chain
- compiler 需要得到最后一个 loader 产生的处理结果。这个处理结果应该是String或者Buffer（被转换为一个String）
### 1.1 loader 运行的总体流程
![](/public/images/webpackflowloader.png)
![](/public/images/loader-runner2.png)
### 1.2 loader-runner
- [loader-runner](https://github.com/webpack/loader-runner#readme)是一个执行loader链条的模块
#### 1.2.1 loader 类型
- [loader的叠加顺序](https://github.com/webpack/webpack/blob/v4.39.3/lib/NormalModuleFactory.js#L159-L339) = post(后置)+inline(内联)+normal(正常)+pre(前置)
#### 1.2.2 特殊配置
- [loaders/#configuration](https://webpack.js.org/concepts/loaders/#configuration)

| 符号 | 变量 | 含义 | |
| --- | --- | --- | --- |
| -! | noPreAutoLoaders | 不要前置和普通loader | Prefixing with -! will disable all configured preLoaders and loaders but not postLoaders |
| ! | noAutoLoaders | 不要普通 loader | Prefixing with ! will disable all configured normal loaders |
| !! | noPrePostAutoLoaders | 不要前后置和普通 loader,只要内联 loader | Prefixing with !! will disable all configured loaders (preLoaders, loaders, postLoaders) |

#### 1.2.3 pitch
- 比如 a!b!c!module,正常调用顺序应该是c、b、a，但是真正调用顺序是a(pitch)、b(pitch)、c(pitch)、c、b、a，如果其中任何一个 pitching loader 返回了值就相当于在它以及它右边的loader已经执行完毕
- 比如如果b返回了字符串"result b",接下来只有a会被系统执行，且a的loader收到的参数是 result b
- loader 根据返回值可以分为两种，一种是返回 js 代码（一个 module 的代码，含有类似 module.export 语句）的 loader，还有不能作为最左边 loader 的其他 loader
- 有时候我们想把两个第一种 loader chain 起来，比如 style-loader!css-loader! 问题是 css-loader 的返回值是一串 js 代码，如果按正常方式写 style-loader 的参数就是一串代码字符串
- 为了解决这种问题，我们需要在 style-loader 里执行 require(css-loader!resources)

pitch 与 loader 本身方法的执行顺序图
```bash
|- a-loader `pitch`
  |- b-loader `pitch`
    |- c-loader `pitch`
      |- requested module is picked up as a dependency
    |- c-loader normal execution
  |- b-loader normal execution
|- a-loader normal execution
```
![](/public/images/loader_pitch.jpg)
#### 1.2.4 执行流程
##### 1.2.4.1 runner.js
```js
let {runLoaders} = require('loader-runner')
//let runLoaders = require('./loader-runner')
let path = require('path');
let fs = require('fs');
let filePath = path.resolve(__dirname, 'src', 'index.js');//入口模块
//定在require方法里的 inline Loader
let request = `!!inline1-loader!inline2-loader!${filePath}`;
//不同的loader并不loader的类型属性，而是你在使用 的使用了什么样的enforce
let rules = [
    {
        test: /\.js$/,
        use: ['normal1-loader', 'normal2-loader']//普通的loader
    },
    {
        test: /\.js$/,
        enforce: 'post',
        use: ['post1-loader', 'post2-loader']//post的loader 后置
    },
    {
        test: /\.js$/,
        enforce: 'pre',
        use: ['pre1-loader', 'pre2-loader']//pre的loader 前置 
    },
]
//loader执行的真正顺序是
// post pitch inline pitch normal pitch pre pitch=>pre loader normal loader inline loader post loader
// pitch很少使用
//顺序是 post(后置)+inline(内联)+normal(普通)+pre(前置)
//parts=['inline1-loader','inline2-loader','src/index.js']
let parts = request.replace(/^-?!+/,'').split('!');
let resource = parts.pop();//'src/index.js'
//解析loader的绝对路径 C:\5.loader\loaders\inline1-loader.js
let resolveLoader = loader => path.resolve(__dirname, 'loaders', loader);
//inlineLoaders=[inline1-loader绝对路径，inline2-loader绝对路径]
let inlineLoaders = parts;
let preLoaders = [], normalLoaders = [], postLoaders = [];
for(let i=0;i<rules.length;i++){
    let rule = rules[i];
    if(rule.test.test(resource)){
        if(rule.enforce==='pre'){
            preLoaders.push(...rule.use);
        }else if(rule.enforce==='post'){
            postLoaders.push(...rule.use);
        }else{
            normalLoaders.push(...rule.use);
        }
    }
}

/**
 * 正常 post(后置)+inline(内联)+normal(普通)+pre(前置)
 * Prefixing with ! will disable all configured normal loaders
 * post(后置)+inline(内联)+pre(前置)
 * Prefixing with !! will disable all configured loaders (preLoaders, loaders, postLoaders)
 * inline(内联)
 * Prefixing with -! will disable all configured preLoaders and loaders but not postLoaders
 * post(后置)+inline(内联)
 */
let loaders = [];//表示最终生效的loader
if(request.startsWith('!!')){
    loaders = [...inlineLoaders];
}else if(request.startsWith('-!')){
    loaders = [...postLoaders,...inlineLoaders];
}else if(request.startsWith('!')){
    loaders = [...postLoaders,...inlineLoaders,...preLoaders];
}else{
    loaders = [...postLoaders,...inlineLoaders,...normalLoaders,...preLoaders];
}
loaders  = loaders.map(resolveLoader);
//我们已经 收到了所有的loader绝对路径组成的数组
runLoaders({
    resource,//要加载和转换的模块
    loaders,//loader的数组
    context:{name:'zhufeng'}, //基础上下文件对象
    readResource:fs.readFile.bind(fs) //读取硬盘文件的方法

},(err,result)=>{
  console.log(err);
  console.log(result);
  console.log(result.resourceBuffer.toString('utf8'));
});
```
#### 1.2.5 loaders
##### 1.2.5.1 pre-loader1.js
loaders\pre-loader1.js
```js
function loader(source) {
  console.log("pre1");
  return source + "//pre1";
}
module.exports = loader;
```
##### 1.2.5.2 pre-loader2.js
loaders\pre-loader2.js
```js
function loader(source) {
  console.log("pre2");
  return source + "//pre2";
}
module.exports = loader;
```
##### 1.2.5.3 normal-loader1.js
loaders\normal-loader1.js
```js
function loader(source) {
    console.log("normal1");
    return source + "//normal1";
}
loader.pitch = function() {
    return "normal1pitch";
}
module.exports = loader;
```
##### 1.2.5.4 normal-loader2.js
loaders\normal-loader2.js
```js
function loader(source) {
  console.log("normal2");
  return source + "//normal2";
}
/* loader.pitch = function(){
  return 'normal-loader2-pitch';
} */
module.exports = loader;
```
##### 1.2.5.5 inline-loader1.js
loaders\inline-loader1.js
```js
function loader(source) {
  console.log("inline1");
  return source + "//inline1";
}

module.exports = loader;
```
##### 1.2.5.6 inline-loader2.js
loaders\inline-loader2.js
```js
function loader(source) {
  console.log("inline2");
  return source + "//inline2";
}
module.exports = loader;
```
##### 1.2.5.7 post-loader1.js
loaders\post-loader1.js
```js
function loader(source) {
  console.log("post1");
  return source + "//post1";
}
module.exports = loader;
```
##### 1.2.5.8 post-loader2.js
loaders\post-loader2.js
```js
function loader(source) {
    console.log("post2");
    return source + "//post2";
}
module.exports = loader;
```
![](/public/images/pitchloaderexec.png)
## 2.babel-loader
- [babel-loader](https://github.com/babel/babel-loader/blob/master/src/index.js)
- [@babel/core](https://babeljs.io/docs/en/next/babel-core.html)
- [babel-plugin-transform-react-jsx](https://babeljs.io/docs/en/babel-plugin-transform-react-jsx/)
- previousRequest 前面的loader
- currentRequest 自己和后面的loader
- remainingRequest 后面的loader+资源路径
- data: 和普通的loader函数的第三个参数一样,而且loader执行的全程用的是同一个对象

| 属性 | 值 |
| --- | --- |
| this.request | /loaders/babel-loader.js!/src/index.js |
| this.resourcePath | /src/index.js |

```sh
$ npm i @babel/preset-env @babel/core -D
```
```js
const babel = require("@babel/core");
function loader(source, imputSourceMap, data) {
    //C:\webpack-analysis2\loaders\babel-loader.js!C:\webpack-analysis2\src\index.js
    const options = {
        presets: ["@babel/preset-env"],
        inputuSourceMap: inputSourceMap,
        sourceMaps: true, // sourceMaps: true 是告诉 babel 要生成 sourcemap
        filename: this.request.split("!")[1].split("/").pop(),
    };
    // 在 webpack.config.js 中增加 devtool: "eval-source-map"
    let { code, map, ast } = babel.transform(source, options);
    return this.callback(null, code, map, ast);
}
module.exports = loader;
```
```js
resolveLoader: {
    alias: { // 可以配置别名
        "babel-loader": resolve("./build/babel-loader.js")
    }, // 也可以配置 loaders 加载目录
    modules: [path.resolve('./loaders'), 'node_modules']
},
{
    test: /\.js$/,
    use: ["babel-loader"]
}
```
## 3.file
- file-loader 并不会对文件内容进行任何转换，只是复制一份文件内容，并根据配置为他生成一个唯一的文件名。
### 3.1 file-loader
- [loader-utils](https://github.com/webpack/loader-utils)
- [file-loader](https://github.com/webpack-contrib/file-loader/blob/master/src/index.js)
- [public-path](https://webpack.js.org/guides/public-path/#on-the-fly)

```js
const { getOptions, interpolateName } = require("loader-utils");
function loader(content) {
    let options = getOptions(this) || {};
    let url = interpolateName(this, options.filename || "[hash].[ext]", {
        content,
    });
    this.emitFile(url, content);
    return `module.exports = ${JSON.stringify(url)}`;
}
loader.raw = true;
module.exports = loader;
```
- 通过 loaderUtils.interpolateName 方法可以根据 options.name 以及文件内容生成一个唯一的文件名 url（一般配置都会带上hash,否则很可能由于文件重名而冲突）
- 通过 this.emitFile(url, content) 告诉 webpack 我需要创建一个文件，webpack 会根据参数创建对应的文件，放在 public path 目录下
- 返回 module.exports = ${JSON.stringify(url)} ,这样就会把原来的文件路径替换为编译后的路径

### 3.2 url-loader
```js
let { getOptions } = require("loader-utils");
var mime = require("mime");
function loader(source) {
  let options = getOptions(this) || {};
  let { limit, fallback = "file-loader" } = options;
  if (limit) {
    limit = parseInt(limit, 10);
  }
  const mimetype = mime.getType(this.resourcePath);
  if (!limit || source.length < limit) {
    let base64 = `data:${mimetype};base64,${source.toString("base64")}`;
    return `module.exports = ${JSON.stringify(base64)}`;
  } else {
    let fileLoader = require(fallback || "file-loader");
    return fileLoader.call(this, source);
  }
}
loader.raw = true;
module.exports = loader;
```
### 3.3 样式处理
- [css-loader](https://github.com/webpack-contrib/css-loader/blob/master/lib/loader.js)的作用是处理 css 中的 @import 和 url 这样的外部资源
- [style-loader](https://github.com/webpack-contrib/style-loader/blob/master/index.js)的作用是把样式插入到 DOM 中，方法是在 head 中插入一个 style 标签，并把样式写入到这个标签的 innerHTML 里
- [less-loader](https://github.com/webpack-contrib/less-loader)把 less 编译成 css
- [pitching-loader](https://webpack.js.org/api/loaders/#pitching-loader)
- [loader-utils](https://github.com/webpack/loader-utils)
- [!!](https://webpack.js.org/concepts/loaders/#configuration)
```bash
$ npm i less postcss css-selector-tokenizer -D
```
#### 3.3.2 使用 less-loader
##### 3.3.2.1 index.js
src\index.js
```js
import "./index.less";
```
##### 3.3.2.2 src\index.less
src\index.less
```less
@color: red;
#root {
    color: @color;
}
```
##### 3.3.2.3 src\index.html
src\index.html
```html
<div id="root">hello</div>
<div class="avatar"></div>
```
##### 3.3.2.4 webpack.config.js
webpack.config.js
```js
{
  test: /\.less$/,
  use: [
    'style-loader',
    'less-loader'
  ]
}
```
##### 3.3.2.5 less-loader.js
```js
let less = require("less");
function loader(source) {
  let callback = this.async();
  less.render(source, { filename: this.resource }, (err, output) => {
    callback(err, output.css);
  });
}
module.exports = loader;
```
##### 3.3.2.6 style-loader
```js
function loader(source) {
  let script = `
      let style = document.createElement("style");
      style.innerHTML = ${JSON.stringify(source)};
    document.head.appendChild(style);
    module.exports = "";
    `;
  return script;
}
module.exports = loader;
```
#### 3.3.5 两个左侧模块连用
##### 3.3.5.1 less-loader.js
```js
const less = require("less");
function loader(source) {
    const callback = this.async();
    less.render(source, { filename: this.resource }, (err, output) => {
        callback(err, `module.exports = ${JSON.stringify(output.css)}`);
    });
}
module.exports = loader;
```
##### 3.3.5.2 style-loader.js
```js
let loaderUtils = require("loader-utils");
function loader(source) {}
//https://github.com/webpack/webpack/blob/v4.39.3/lib/NormalModuleFactory.js#L339
loader.pitch = function (remainingRequest, previousRequest, data) {
  //C:\webpack-analysis2\loaders\less-loader.js!C:\webpack-analysis2\src\index.less
  console.log("previousRequest", previousRequest); //之前的路径
  //console.log('currentRequest', currentRequest);//当前和后面的路径
  console.log("remainingRequest", remainingRequest); //剩下的路径
  console.log("data", data);
  // !! noPrePostAutoLoaders 不要前后置和普通loader
  //__webpack_require__(/*! !../loaders/less-loader.js!./index.less */ "./loaders/less-loader.js!./src/index.less");
  let style = `
    var style = document.createElement("style");
    style.innerHTML = require(${loaderUtils.stringifyRequest(
      this,
      "!!" + remainingRequest
    )});
    document.head.appendChild(style);
 `;
  return style;
};
module.exports = loader;
```
## 4.loader-runner实现
- [LoaderRunner.js](https://github.com/webpack/loader-runner/blob/v2.4.0/lib/LoaderRunner.js)
- [NormalModuleFactory.js](https://github.com/webpack/webpack/blob/v4.39.3/lib/NormalModuleFactory.js#L180)
- [NormalModule.js](https://github.com/webpack/webpack/blob/v4.39.3/lib/NormalModule.js#L292)
![](/public/images/3f34c35da4cc3049ed2c414abbae9b99.png)

```js
/**
 * 把loader转成loader对象
 * @param {*} loader loader的绝对路径
 */
function createLoaderObject(loader) {
    let module = require(loader);
    let normal = module;
    let pitch = module.pitch;
    let raw = module.raw;
    let loaderObject = {
        path: loader, //loader的绝对路径
        normal,//loader的normal函数
        pitch, //loader的pitch函数
        raw,//是否要转成Buffer, raw=true参数就要转成Buffer,raw=false参数就要转成字符串
        data: {},//每个load都有一个自己的data自定义对象,用来可以存放一些自定义的数据
        pitchExecuted: false,//此loader的pitch方法是否已经执行过了
        normalExecuted: false,//此loader的normal方法是否已经执行过了
    }
    return loaderObject;
}
function iterateNormalLoaders(options, loaderContext, args, runLoadersCallback) {
    if (loaderContext.loaderIndex < 0) {
        return runLoadersCallback(null, ...args);
    }
    let currentLoaderObject = loaderContext.loaders[loaderContext.loaderIndex];//获取当前的索引对应的loader
    if (currentLoaderObject.normalExecuted) {//如果说当前的loader已经执行过了,则执行下一个loader对应的pitch
        loaderContext.loaderIndex--;
        return iterateNormalLoaders(options, loaderContext,args, runLoadersCallback);
    }
    let fn = currentLoaderObject.normal;//获取当前的loader的normal函数
    currentLoaderObject.normalExecuted = true;//表示当前的loader的normal函数已经执行过了
    // 如果这个loader有normal方法
    //fn可以同步也可以异步
    convertArgs(args, currentLoaderObject.raw);
    runSyncOrAsync(fn, loaderContext, args, (err, ...args) => {//args可能有值,也可有没有值,可能有一个值,也可能有多个值 
        if (err) return runLoadersCallback(err);
        return iterateNormalLoaders(options, loaderContext, args, runLoadersCallback);
    });
}
function convertArgs(args, raw) {
    if (raw && !Buffer.isBuffer(args[0])) {
        args[0] = Buffer.from(args[0]);//如果这个normal函数想要buffer,但是参数不是Buffer
    } else if (!raw && Buffer.isBuffer(args[0])) {//想要字符串，但是是个buffer
        args[0] = args[0].toString('utf8');
    }
}
function processResource(options, loaderContext, runLoadersCallback) {
    options.readResource(loaderContext.resource, (err, buffer) => {
        loaderContext.loaderIndex = loaderContext.loaders.length - 1;
        options.resourceBuffer = buffer;//要加载的文件的原始文件内容
        iterateNormalLoaders(options, loaderContext, [buffer], runLoadersCallback);
    });
}
function iteratePitchLoaders(options, loaderContext, runLoadersCallback) {
    //如果当前索引已经 大于等loader的数量了,则表示所有的loader pitch执行完了
    if (loaderContext.loaderIndex >= loaderContext.loaders.length) {
        return processResource(options, loaderContext, runLoadersCallback);
    }
    let currentLoaderObject = loaderContext.loaders[loaderContext.loaderIndex];//获取当前的索引对应的loader
    if (currentLoaderObject.pitchExecuted) {//如果说当前的loader已经执行过了,则执行下一个loader对应的pitch
        loaderContext.loaderIndex++;
        return iteratePitchLoaders(options, loaderContext, runLoadersCallback);
    }
    let fn = currentLoaderObject.pitch;//获取当前的loader的pitch函数
    currentLoaderObject.pitchExecuted = true;//表示当前的loader的pitch函数已经执行过了
    if (!fn) {//如果当前的loader没有pitch,直接执行下一个loader的pitch方法 
        return iteratePitchLoaders(options, loaderContext, runLoadersCallback);
    }
    // 如果这个loader有pitch方法
    //fn可以同步也可以异步
    runSyncOrAsync(fn, loaderContext, [
        loaderContext.remainingRequest, loaderContext.previousRequest, loaderContext.data
    ], (err, ...args) => {//args可能有值,也可有没有值,可能有一个值,也可能有多个值 
        if (args.length > 0 && args.some(item => !!item)) {//有任何一个有值就可以
            //跳过后续的pitch和读文件,直接掉头执行前一个loader的normal
        } else {
            return iteratePitchLoaders(options, loaderContext, runLoadersCallback);
        }
    });
}
/**
 * 以同步或者异步的方式执行fn
 * context.fn(args);callback()
 * @param {*} fn 要执行的函数
 * @param {*} context fn执行的时候的this指针
 * @param {*} args 传递给fn的参数
 * @param {*} callback fn执行完成后的回调 
 */
function runSyncOrAsync(fn, context, args, callback) {
    let isSync = true;//默认是同步执行
    context.callback = (...args) => {
        callback(null, ...args);
    }
    context.async = () => {
        isSync = false;//把同步变成异步
        return context.callback;
    }
    var result = fn.apply(context, args);//用指定的参数和this对象执行函数,返回一个结果
    // isSync为true表示是同步,同步意味着执行完当前函数后,会直接自动执行callback回调
    // isSync为false表示是同步,那意味着扫行当前函数后什么都不做了
    if (isSync) {
        if (result === undefined) {
            return callback();
        } else {
            return callback(null, result);
        }
    }
}

function runLoaders(options, finalCallback) {
    let resource = options.resource;//加载的文件 src\index.js
    let loaders = options.loaders || [];//
    let loaderContext = options.context || {};//loader函数执行时的上下文对象
    let readResource = options.readResource || fs.readFile;//用来读加载的文件的方法
    let loaderObjects = loaders.map(createLoaderObject);
    loaderContext.resource = resource;//加载的文件
    loaderContext.readResource = readResource;
    loaderContext.loaders = loaderObjects;
    loaderContext.loaderIndex = 0;//当前正在执行的loader的索引
    loaderContext.callback = null;//后面在执行 loader的时候会赋值 返回多个值
    loaderContext.async = null;//后面在执行 loader的时候会赋值 把loader的执行从同步变成异步
    Object.defineProperty(loaderContext, 'request', {
        get() {//request = 所有的loader!要加载的模块
            return loaderContext.loaders.map(l = l.path).concat(loaderContext.resource).join('!')
        }
    });
    Object.defineProperty(loaderContext, 'remainingRequest', {
        get() {//request = 所有的loader!要加载的模块
            return loaderContext.loaders.slice(loaderContext.loaderIndex + 1).map(loader => loader.path).concat(loaderContext.resource).join('!')
        }
    });
    Object.defineProperty(loaderContext, 'currentRequest', {
        get() {//request = 所有的loader!要加载的模块 currentRequest 不是仅仅自己,而包括自己和后续的
            return loaderContext.loaders.slice(loaderContext.loaderIndex).map(loader => loader.path).concat(loaderContext.resource).join('!')
        }
    });
    Object.defineProperty(loaderContext, 'previousRequest', {
        get() {//从第一个loader到当前的loader,不包含当前的loader
            return loaderContext.loaders.slice(0, loaderContext.loaderIndex).map(loader => loader.path).concat(loaderContext.resource).join('!')
        }
    });

    Object.defineProperty(loaderContext, 'data', {
        get() {//从第一个loader到当前的loader,不包含当前的loader
            return loaderContext.loaders[loaderContext.loaderIndex].data;
        }
    });
    let processOptions = {
        resourceBuffer: null,//存放原始内容对应的Buffer 用loader转换前的内容Buffer
        readResource  //fs.readFile
    }
    //开始从左往后执行每个loader的pitch方法
    iteratePitchLoaders(processOptions, loaderContext, (err, result) => {
        finalCallback(
            err, {
            result,
            resourceBuffer: processOptions.resourceBuffer
        }
        );
    });
}

exports.runLoaders = runLoaders;
```