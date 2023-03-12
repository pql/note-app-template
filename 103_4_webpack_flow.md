## 1.调试 webpack
### 1.1 通过chrome调试
```sh
node --inspect-brk ./node_modules/webpack-cli/bin/cli.js
```
### 1.2 通过执行命令调试
- 打开工程目录，点击调试按钮，再点击小齿轮的配置按钮系统就会生成 launch.json 配置文件
- 修改好了以后直接点击F5就可以启动调试

.vscode\launch.json
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "request": "launch",
            "name": "debug webpack",
            "cwd": "${workspaceFolder}",
            "program": "${workspaceFolder}/node_modules/webpack-cli/bin/cli.js"
        }
    ]
}
```
### 1.3 debugger.js
```js
const webpack = require("webpack");
const webpackOptions = require("./webpack.config");
const compiler = webpack(webpackOptions);
// 4.执行对象的run方法开始执行编译
compiler.run((err, stats) => {
    console.log(err);
    console.log(stats.toJson({
        assets: true,
        chunks: true,
        modules: true,
        entries: true
    }));
});
```
### 1.4 调试webpack-cli源码
```sh
git clone https://github.com/webpack/webpack.git
git reset --hard vx.x.x
yarn
yarn link

git clone https://github.com/webpack/webpack-cli.git
git reset --hard webpack-cli@x.x.x
cd packages\webpack-cli
yarn link webpack
yarn link
```
```sh
yarn link webpack
yarn link webpack-cli

yarn unlink webpack
yarn unlink webpack-cli
```
## 2.tapable.js
- tapable是一个类似于Node.js中的EventEmitter的库，但更专注于自定义事件的触发和处理
- webpack 通过 tapable 将实现与流程解耦，所有具体实现通过插件的形式存在
```js
class SyncHook {
    constructor() {
        this.taps = []
    }
    tap(name, fn) {
        this.taps.push(fn);
    }
    call() {
        this.taps.forEach(tap => tap());
    }
}

let hook = new SyncHook();
hook.tap('some name', () => {
    console.log('some name');
});

class Plugin {
    apply() {
        hook.tap('Plugin', () => {
            console.log('Plugin');
        });
    }
}
new Plugin().apply();
hook.call();
```
## 3.webpack 编译流程
1. 初始化参数：从配置文件和Shell语句中读取并合并参数,得出最终的配置对象
2. 用上一步得到的参数初始化Compiler对象
3. 加载所有配置的插件
4. 执行对象的run方法开始执行编译
5. 根据配置中的entry找出入口文件
6. 从入口文件出发,调用所有配置的Loader对模块进行编译
7. 再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理
8. 根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk
9. 再把每个Chunk转换成一个单独的文件加入到输出列表
10. 在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统

> 在以上过程中，Webpack 会在特定的时间点广播出特定的事件，插件在监听到感兴趣的事件后会执行特定的逻辑，并且插件可以调用 Webpack 提供的 API 改变 Webpack 的运行结果
![](/public/imges/webpackflow2020.png)
### 3.1 debugger.js
debugger.js
```js
const webpack = require('./webpack');
const webpackOptions = require('./webpack.config');
//compiler代表整个编译过程.
const compiler = webpack(webpackOptions);
//调用它的run方法可以启动编译
compiler.run((err,stats)=>{
    console.log(err);
    let result = stats.toJson({
        files:true,//产出了哪些文件
        assets:true,//生成了那些资源
        chunk:true,//生成哪些代码块
        module:true,//模块信息
        entries:true //入口信息
    });
    console.log(JSON.stringify(result,null,2));
});
```
### 3.2 webpack.config.js
webpack.config.js
```js
const path = require('path');
const RunPlugin = require('./plugins/run-plugin');
const DonePlugin = require('./plugins/done-plugin');
const AssetPlugin = require('./plugins/assets-plugin');
module.exports = {
    mode:'development',
    devtool:false,
    context:process.cwd(),//上下文目录, ./src .默认代表根目录 默认值其实就是当前命令执行的时候所在的目录
    entry:{
        entry1:'./src/entry1.js',
        entry2:'./src/entry2.js'
    },
    output:{
        path:path.join(__dirname,'dist'),
        filename:'[name].js'
    },
    resolve:{
        extensions:['.js','.jsx','.json']
    },
    module:{
        rules:[
            {
                test:/\.js$/,
                use:[
                    path.resolve(__dirname,'loaders','logger1-loader.js'),
                    path.resolve(__dirname,'loaders','logger2-loader.js')
                ]
            }
        ]
    },
    plugins:[
        new RunPlugin(),
        new DonePlugin(),
        new AssetPlugin()
    ]
}
```
### 3.3 webpack.js
webpack.js
```js
const Compiler = require('./Compiler');
function webpack(options){
    //1. 初始化参数：从配置文件和Shell语句中读取并合并参数,得出最终的配置对象
    let shellConfig= process.argv.slice(2).reduce((shellConfig,item)=>{
        //item=   --mode=development
        let [key,value] = item.split('=');
        shellConfig[key.slice(2)]=value;
        return shellConfig;
    },{});
    let finalConfig = {...options,...shellConfig};
    //2. 用上一步得到的参数初始化Compiler对象
    let compiler = new Compiler(finalConfig);
    //3. 加载所有配置的插件
    let {plugins} = finalConfig;
    for(let plugin of plugins){
        plugin.apply(compiler);
    }
    return compiler;
}
module.exports = webpack;
```
### 3.4 Compiler.js
Compiler.js
```js
let {SyncHook} = require('tapable');
let Complication = require('./Complication');
let path = require('path');
let fs = require('fs');
class Compiler{
  constructor(options){
    this.options = options;
    this.hooks = {
        run:new SyncHook(),//开始启动编译 刚刚开始
        emit:new SyncHook(['assets']),//会在将要写入文件的时候触发
        done:new SyncHook()//将会在完成编译的时候触发 全部完成
    }
  }
  //4. 执行Compiler对象的run方法开始执行编译
  run(callback){
    this.hooks.run.call();//触发run钩子
    //5. 根据配置中的entry找出入口文件
    this.compile((err,stats)=>{
      this.hooks.emit.call(stats.assets);
      //10. 在确定好输出内容后，根据配置确定输出的路径和文件名，把文件内容写入到文件系统
      for(let filename in stats.assets){
          let filePath = path.join(this.options.output.path,filename);
          fs.writeFileSync(filePath,stats.assets[filename],'utf8');
      }
      callback(null,{
        toJson:()=>stats
      });
    });
    //监听入口的文件变化,如果文件变化了,重新再开始编译
   /*  Object.values(this.options.entry).forEach(entry=>{
      fs.watchFile(entry,()=>this.compile(callback));
    }); */
    //中间是我们编译流程
    this.hooks.done.call();//编译之后触发done钩子
  }
  compile(callback){
    let complication = new Complication(this.options);
    complication.build(callback);
  }
}
module.exports = Compiler;
```
### 3.5 Complication.js
Complication.js
```js
const path = require('path');
const fs = require('fs');
const types = require('babel-types');
const parser = require('@babel/parser');
const traverse = require('@babel/traverse').default;
const generator = require('@babel/generator').default;
const baseDir = toUnitPath(process.cwd());//\
function toUnitPath(filePath) {
    return filePath.replace(/\\/g, '/');
}
class Complication {
    constructor(options) {
        this.options = options;
        //webpack4 数组  webpack5 set
        this.entries = [];//存放所有的入口
        this.modules = [];// 存放所有的模块
        this.chunks = [];//存放所的代码块
        this.assets = {};//所有产出的资源
        this.files = [];//所有产出的文件
    }
    build(callback) {
        //5. 根据配置中的entry找出入口文件
        let entry = {};
        if (typeof this.options.entry === 'string') {
            entry.main = this.options.entry;
        } else {
            entry = this.options.entry;
        }
        //entry={entry1:'./src/entry1.js',entry2:'./src/entry2.js'}
        for (let entryName in entry) {
            //5.获取 entry1的绝对路径
            let entryFilePath = toUnitPath(path.join(this.options.context, entry[entryName]));
            //6.从入口文件出发,调用所有配置的Loader对模块进行编译
            let entryModule = this.buildModule(entryName, entryFilePath);
            //this.modules.push(entryModule);
            //8. 根据入口和模块之间的依赖关系，组装成一个个包含多个模块的 Chunk
            let chunk = {
                name: entryName, entryModule, modules: this.modules.filter(item => {
                    return item.name === entryName || item.extraNames.includes(entryName);
                })
            };
            this.entries.push(chunk);
            this.chunks.push(chunk);
        }
        //9. 再把每个Chunk转换成一个单独的文件加入到输出列表
        this.chunks.forEach(chunk => {
            let filename = this.options.output.filename.replace('[name]', chunk.name);
            // this.assets就是输出列表 key输出的文件名 值就是输出的内容
            this.assets[filename] = getSource(chunk);
        });

        callback(null, {
            entries: this.entries,
            chunks: this.chunks,
            modules: this.modules,
            files: this.files,
            assets: this.assets
        });
    }
    //name=名称,modulePath=模块的绝对路径
    buildModule(name, modulePath) {
        //6. 从入口文件出发,调用所有配置的Loader对模块进行编译
        //1.读取模块文件的内容 
        let sourceCode = fs.readFileSync(modulePath, 'utf8');//console.log('entry1');
        let rules = this.options.module.rules;
        let loaders = [];///寻找匹配的loader
        for (let i = 0; i < rules.length; i++) {
            let { test } = rules[i];
            //如果此rule的正则和模块的路径匹配的话
            if (modulePath.match(test)) {
                loaders = [...loaders, ...rules[i].use];
            }
        }
        sourceCode = loaders.reduceRight((sourceCode, loader) => {
            return require(loader)(sourceCode);
        }, sourceCode);

        /*  for(let i=loaders.length-1;i>=0;i--){
             let loader = loaders[i];
             sourceCode = require(loader)(sourceCode);
         } */
        //console.log('entry1');//2//1
        //console.log(sourceCode);
        //7. 再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理
        //获得当前模块模块ID ./src/index.js
        let moduleId = './' + path.posix.relative(baseDir, modulePath);
        let module = { id: moduleId, dependencies: [], name, extraNames: [] };
        let ast = parser.parse(sourceCode, { sourceType: 'module' });
        traverse(ast, {
            CallExpression: ({ node }) => {
                if (node.callee.name === 'require') {
                    //依赖的模块的相对路径
                    let moduleName = node.arguments[0].value;//./title1
                    //获取当前模块的所有的目录
                    let dirname = path.posix.dirname(modulePath);// /
                    //C:/aproject/zhufengwebpack202106/4.flow/src/title1
                    let depModulePath = path.posix.join(dirname, moduleName);
                    let extensions = this.options.resolve.extensions;
                    depModulePath = tryExtensions(depModulePath, extensions);//已经包含了拓展名了
                    //得到依赖的模块ID C:/aproject/zhufengwebpack202106/4.flow/src/title1.js
                    //相对于项目根目录 的相对路径 ./src/title1.js
                    let depModuleId = './' + path.posix.relative(baseDir, depModulePath);
                    //require('./title1');=>require('./src/title1.js');
                    node.arguments = [types.stringLiteral(depModuleId)];
                    //依赖的模块绝对路径放到当前的模块的依赖数组里
                    module.dependencies.push({ depModuleId, depModulePath });
                }
            }
        });
        let { code } = generator(ast);
        module._source = code;//模块源代码指向语法树转换后的新生成的源代码
        //7. 再找出该模块依赖的模块，再递归本步骤直到所有入口依赖的文件都经过了本步骤的处理
        module.dependencies.forEach(({ depModuleId, depModulePath }) => {
            let depModule = this.modules.find(item => item.id === depModuleId);
            if (depModule) {
                depModule.extraNames.push(name);
            } else {
                let dependencyModule = this.buildModule(name, depModulePath);
                this.modules.push(dependencyModule);
            }
        });
        return module;
    }
}
function getSource(chunk) {
    return `
    (() => {
        var modules = ({
            ${chunk.modules.map(module => `
                    "${module.id}":(module,exports,require)=>{
                        ${module._source}
                    }
                `).join(',')
        }
        });
        var cache = {};
        function require(moduleId) {
          var cachedModule = cache[moduleId];
          if (cachedModule !== undefined) {
            return cachedModule.exports;
          }
          var module = cache[moduleId] = {
            exports: {}
          };
          modules[moduleId](module, module.exports, require);
          return module.exports;
        }
        var exports = {};
        (() => {
         ${chunk.entryModule._source}
        })();
      })()
        ;
    `
}
function tryExtensions(modulePath, extensions) {
    extensions.unshift('');
    for (let i = 0; i < extensions.length; i++) {
        let filePath = modulePath + extensions[i];// ./title.js
        if (fs.existsSync(filePath)) {
            return filePath;
        }
    }
    throw new Error(`Module not found`);
}
module.exports = Complication;
```
### 3.6 run-plugin.js
plugins\run-plugin.js
```js
class RunPlugin{
    apply(compiler){
        //对于代码执行来说没有用,但是对于阅读代码的人来说可以起到提示的作用
        compiler.hooks.run.tap('RunPlugin',()=>{
            console.log('开始编译了');
        });
    }
}
module.exports = RunPlugin;
```
### 3.7 assets-plugin.js
plugins\assets-plugin.js
```js
class AssetPlugin{
    apply(compiler){
        //对于代码执行来说没有用,但是对于阅读代码的人来说可以起到提示的作用
        compiler.hooks.emit.tap('AssetPlugin',(assets)=>{
            debugger
            assets['assets.md']= Object.keys(assets).join('\n');
        });
    }
}
module.exports = AssetPlugin;
```
### 3.8 done-plugin.js
plugins\done-plugin.js
```js
class DonePlugin{
    apply(compiler){
        //对于代码执行来说没有用,但是对于阅读代码的人来说可以起到提示的作用
        compiler.hooks.done.tap('DonePlugin',()=>{
            console.log('编译结束了');
        });
    }
}
module.exports = DonePlugin;
```
### 3.9 logger1-loader.js
loaders\logger1-loader.js
```js
function loader(source){
   console.log('logger1-loader.js');
   return source+'//1';
}
module.exports = loader;
```
### 3.10 logger2-loader.js
loaders\logger2-loader.js
```js
function loader(source){
   console.log('logger2-loader.js');
   return source+'//2';
}
module.exports = loader;
```
### 3.11 src\entry1.js
src\entry1.js
```js
let title1 = require('./title1');
console.log(title1);
```
### 3.12 src\entry2.js
src\entry2.js
```js
let title1 = require('./title1');
console.log(title1);
```
### 3.13 src\title1.js
src\title1.js
```js
module.exports = "title1";
```
### 3.14 src\title2.js
src\title2.js
```js
module.exports = "title2";
```
## 4.Stats 对象
- 在 Webpack 的回调函数中会得到 stats 对象
- 这个对象实际来自于Compilation.getStats()，返回的是主要含有modules、chunks和assets三个属性值的对象。
- Stats 对象本质上来自于[lib/Stats.js](https://github.com/webpack/webpack/blob/v4.39.3/lib/Stats.js)的类实例

| 字段 | 含义 |
| --- | --- |
| modules | 记录了所有解析后的模块 |
| chunks | 记录了所有chunk |
| assets | 记录了所有要生成的文件 |

```sh
npx webpack --profile --json > stats.json
```

```json
{
  "hash": "780231fa9b9ce4460c8a",     //编译使用的 hash
  "version": "5.8.0",                 // 用来编译的 webpack 的版本
  "time": 83,                         // 编译耗时 (ms)
  "builtAt": 1606538839612,           //编译的时间
  "publicPath": "auto",               //资源访问路径
  "outputPath": "C:\\webpack5\\dist", //输出目录
  "assetsByChunkName": {              //代码块和文件名的映射
    "main": ["main.js"]
  },
  "assets": [                         //资源数组
    {
      "type": "asset",                //资源类型
      "name": "main.js",              //文件名称
      "size": 2418,                   //文件大小
      "chunkNames": [                 //对应的代码块名称
        "main"
      ],
      "chunkIdHints": [],
      "auxiliaryChunkNames": [],
      "auxiliaryChunkIdHints": [],
      "emitted": false,
      "comparedForEmit": true,
      "cached": false,
      "info": {
        "javascriptModule": false,
        "size": 2418
      },
      "related": {},
      "chunks": [
        "main"
      ],
      "auxiliaryChunks": [],
      "isOverSizeLimit": false
    }
  ],
  "chunks": [                      //代码块数组
    {
      "rendered": true,
      "initial": true,
      "entry": true,
      "recorded": false,
      "size": 80,
      "sizes": {
        "javascript": 80
      },
      "names": [
        "main"
      ],
      "idHints": [],
      "runtime": [
        "main"
      ],
      "files": [
        "main.js"
      ],
      "auxiliaryFiles": [],
      "hash": "d25ad7a8144077f69783",
      "childrenByOrder": {},
      "id": "main",
      "siblings": [],
      "parents": [],
      "children": [],
      "modules": [
        {
          "type": "module",
          "moduleType": "javascript/auto",
          "identifier": "C:\\webpack5\\src\\index.js",
          "name": "./src/index.js",
          "nameForCondition": "C:\\webpack5\\src\\index.js",
          "index": 0,
          "preOrderIndex": 0,
          "index2": 1,
          "postOrderIndex": 1,
          "size": 55,
          "sizes": {
            "javascript": 55
          },
          "cacheable": true,
          "built": true,
          "codeGenerated": true,
          "cached": false,
          "optional": false,
          "orphan": false,
          "dependent": false,
          "issuer": null,
          "issuerName": null,
          "issuerPath": null,
          "failed": false,
          "errors": 0,
          "warnings": 0,
          "profile": {
            "total": 38,
            "resolving": 26,
            "restoring": 0,
            "building": 12,
            "integration": 0,
            "storing": 0,
            "additionalResolving": 0,
            "additionalIntegration": 0,
            "factory": 26,
            "dependencies": 0
          },
          "id": "./src/index.js",
          "issuerId": null,
          "chunks": [
            "main"
          ],
          "assets": [],
          "reasons": [
            {
              "moduleIdentifier": null,
              "module": null,
              "moduleName": null,
              "resolvedModuleIdentifier": null,
              "resolvedModule": null,
              "type": "entry",
              "active": true,
              "explanation": "",
              "userRequest": "./src/index.js",
              "loc": "main",
              "moduleId": null,
              "resolvedModuleId": null
            }
          ],
          "usedExports": null,
          "providedExports": null,
          "optimizationBailout": [],
          "depth": 0
        },
        {
          "type": "module",
          "moduleType": "javascript/auto",
          "identifier": "C:\\webpack5\\src\\title.js",
          "name": "./src/title.js",
          "nameForCondition": "C:\\webpack5\\src\\title.js",
          "index": 1,
          "preOrderIndex": 1,
          "index2": 0,
          "postOrderIndex": 0,
          "size": 25,
          "sizes": {
            "javascript": 25
          },
          "cacheable": true,
          "built": true,
          "codeGenerated": true,
          "cached": false,
          "optional": false,
          "orphan": false,
          "dependent": true,
          "issuer": "C:\\webpack5\\src\\index.js",
          "issuerName": "./src/index.js",
          "issuerPath": [
            {
              "identifier": "C:\\webpack5\\src\\index.js",
              "name": "./src/index.js",
              "profile": {
                "total": 38,
                "resolving": 26,
                "restoring": 0,
                "building": 12,
                "integration": 0,
                "storing": 0,
                "additionalResolving": 0,
                "additionalIntegration": 0,
                "factory": 26,
                "dependencies": 0
              },
              "id": "./src/index.js"
            }
          ],
          "failed": false,
          "errors": 0,
          "warnings": 0,
          "profile": {
            "total": 0,
            "resolving": 0,
            "restoring": 0,
            "building": 0,
            "integration": 0,
            "storing": 0,
            "additionalResolving": 0,
            "additionalIntegration": 0,
            "factory": 0,
            "dependencies": 0
          },
          "id": "./src/title.js",
          "issuerId": "./src/index.js",
          "chunks": [
            "main"
          ],
          "assets": [],
          "reasons": [
            {
              "moduleIdentifier": "C:\\webpack5\\src\\index.js",
              "module": "./src/index.js",
              "moduleName": "./src/index.js",
              "resolvedModuleIdentifier": "C:\\webpack5\\src\\index.js",
              "resolvedModule": "./src/index.js",
              "type": "cjs require",
              "active": true,
              "explanation": "",
              "userRequest": "./title.js",
              "loc": "1:12-33",
              "moduleId": "./src/index.js",
              "resolvedModuleId": "./src/index.js"
            },
            {
              "moduleIdentifier": "C:\\webpack5\\src\\title.js",
              "module": "./src/title.js",
              "moduleName": "./src/title.js",
              "resolvedModuleIdentifier": "C:\\webpack5\\src\\title.js",
              "resolvedModule": "./src/title.js",
              "type": "cjs self exports reference",
              "active": true,
              "explanation": "",
              "userRequest": null,
              "loc": "1:0-14",
              "moduleId": "./src/title.js",
              "resolvedModuleId": "./src/title.js"
            }
          ],
          "usedExports": null,
          "providedExports": null,
          "optimizationBailout": [
            "CommonJS bailout: module.exports is used directly at 1:0-14"
          ],
          "depth": 1
        }
      ],
      "origins": [
        {
          "module": "",
          "moduleIdentifier": "",
          "moduleName": "",
          "loc": "main",
          "request": "./src/index.js"
        }
      ]
    }
  ],
  "modules": [                              //模块数组
    {
      "type": "module",
      "moduleType": "javascript/auto",
      "identifier": "C:\\webpack5\\src\\index.js",
      "name": "./src/index.js",
      "nameForCondition": "C:\\webpack5\\src\\index.js",
      "index": 0,
      "preOrderIndex": 0,
      "index2": 1,
      "postOrderIndex": 1,
      "size": 55,
      "sizes": {
        "javascript": 55
      },
      "cacheable": true,
      "built": true,
      "codeGenerated": true,
      "cached": false,
      "optional": false,
      "orphan": false,
      "issuer": null,
      "issuerName": null,
      "issuerPath": null,
      "failed": false,
      "errors": 0,
      "warnings": 0,
      "profile": {
        "total": 38,
        "resolving": 26,
        "restoring": 0,
        "building": 12,
        "integration": 0,
        "storing": 0,
        "additionalResolving": 0,
        "additionalIntegration": 0,
        "factory": 26,
        "dependencies": 0
      },
      "id": "./src/index.js",
      "issuerId": null,
      "chunks": [
        "main"
      ],
      "assets": [],
      "reasons": [
        {
          "moduleIdentifier": null,
          "module": null,
          "moduleName": null,
          "resolvedModuleIdentifier": null,
          "resolvedModule": null,
          "type": "entry",
          "active": true,
          "explanation": "",
          "userRequest": "./src/index.js",
          "loc": "main",
          "moduleId": null,
          "resolvedModuleId": null
        }
      ],
      "usedExports": null,
      "providedExports": null,
      "optimizationBailout": [],
      "depth": 0
    },
    {
      "type": "module",
      "moduleType": "javascript/auto",
      "identifier": "C:\\webpack5\\src\\title.js",
      "name": "./src/title.js",
      "nameForCondition": "C:\\webpack5\\src\\title.js",
      "index": 1,
      "preOrderIndex": 1,
      "index2": 0,
      "postOrderIndex": 0,
      "size": 25,
      "sizes": {
        "javascript": 25
      },
      "cacheable": true,
      "built": true,
      "codeGenerated": true,
      "cached": false,
      "optional": false,
      "orphan": false,
      "issuer": "C:\\webpack5\\src\\index.js",
      "issuerName": "./src/index.js",
      "issuerPath": [
        {
          "identifier": "C:\\webpack5\\src\\index.js",
          "name": "./src/index.js",
          "profile": {
            "total": 38,
            "resolving": 26,
            "restoring": 0,
            "building": 12,
            "integration": 0,
            "storing": 0,
            "additionalResolving": 0,
            "additionalIntegration": 0,
            "factory": 26,
            "dependencies": 0
          },
          "id": "./src/index.js"
        }
      ],
      "failed": false,
      "errors": 0,
      "warnings": 0,
      "profile": {
        "total": 0,
        "resolving": 0,
        "restoring": 0,
        "building": 0,
        "integration": 0,
        "storing": 0,
        "additionalResolving": 0,
        "additionalIntegration": 0,
        "factory": 0,
        "dependencies": 0
      },
      "id": "./src/title.js",
      "issuerId": "./src/index.js",
      "chunks": [
        "main"
      ],
      "assets": [],
      "reasons": [
        {
          "moduleIdentifier": "C:\\webpack5\\src\\index.js",
          "module": "./src/index.js",
          "moduleName": "./src/index.js",
          "resolvedModuleIdentifier": "C:\\webpack5\\src\\index.js",
          "resolvedModule": "./src/index.js",
          "type": "cjs require",
          "active": true,
          "explanation": "",
          "userRequest": "./title.js",
          "loc": "1:12-33",
          "moduleId": "./src/index.js",
          "resolvedModuleId": "./src/index.js"
        },
        {
          "moduleIdentifier": "C:\\webpack5\\src\\title.js",
          "module": "./src/title.js",
          "moduleName": "./src/title.js",
          "resolvedModuleIdentifier": "C:\\webpack5\\src\\title.js",
          "resolvedModule": "./src/title.js",
          "type": "cjs self exports reference",
          "active": true,
          "explanation": "",
          "userRequest": null,
          "loc": "1:0-14",
          "moduleId": "./src/title.js",
          "resolvedModuleId": "./src/title.js"
        }
      ],
      "usedExports": null,
      "providedExports": null,
      "optimizationBailout": [
        "CommonJS bailout: module.exports is used directly at 1:0-14"
      ],
      "depth": 1
    }
  ],
  "entrypoints": {                        //入口点
    "main": {
      "name": "main",
      "chunks": [
        "main"
      ],
      "assets": [
        {
          "name": "main.js",
          "size": 2418
        }
      ],
      "filteredAssets": 0,
      "assetsSize": 2418,
      "auxiliaryAssets": [],
      "filteredAuxiliaryAssets": 0,
      "auxiliaryAssetsSize": 0,
      "children": {},
      "childAssets": {},
      "isOverSizeLimit": false
    }
  },
  "namedChunkGroups": {                   //命名代码块组
    "main": {
      "name": "main",
      "chunks": [
        "main"
      ],
      "assets": [
        {
          "name": "main.js",
          "size": 2418
        }
      ],
      "filteredAssets": 0,
      "assetsSize": 2418,
      "auxiliaryAssets": [],
      "filteredAuxiliaryAssets": 0,
      "auxiliaryAssetsSize": 0,
      "children": {},
      "childAssets": {},
      "isOverSizeLimit": false
    }
  },
  "errors": [],
  "errorsCount": 0,
  "warnings": [],
  "warningsCount": 0,
  "children": []
}
```