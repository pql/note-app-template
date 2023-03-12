## 1. rollup实战
- [rollupjs](https://rollupjs.org/guide/en/)是下一代ES模块捆绑器

### 1.1 背景
- webpack打包非常繁琐，打包体积比较大
- rollup主要是用来打包JS库的
- vue/react/angular都在用rollup作为打包工具

### 1.2 安装依赖
```sh
cnpm i @babel/core @babel/preset-env  @rollup/plugin-commonjs @rollup/plugin-node-resolve @rollup/plugin-typescript lodash rollup rollup-plugin-babel postcss rollup-plugin-postcss rollup-plugin-terser tslib typescript rollup-plugin-serve rollup-plugin-livereload -D
```
### 1.3 初次使用
#### 1.3.1 rollup.config.js
- Asynchronous Module Definition异步模块定义
- ES6 module是es6提出了新的模块化方案
- IIFE(Immediately Invoked Function Expression)即立即执行函数表达式，所谓立即执行，就是声明一个函数，声明完了立即执行
- UMD全称为Universal Module Definition,也就是通用模块定义
- cjs是nodejs采用的模块化标准，commonjs使用方法require来引入模块,这里require()接收的参数是模块名或者是模块文件的路径

rullup.config.js
```js
export default {
    input:'src/main.js',
    output:{
        file:'dist/bundle.cjs.js',//输出文件的路径和名称
        format:'cjs',//五种输出格式：amd/es6/iife/umd/cjs
        name:'bundleName'//当format为iife和umd时必须提供，将作为全局变量挂在window下
    }
}
```
#### 1.3.2 src\main.js
src\main.js
```js
console.log('hello');
```
#### 1.3.3 package.json
package.json
```json
{
 "scripts": {
    "build": "rollup --config"
  },
}
```
#### 1.3.4 dist\index.html 
dist\index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>rollup</title>
</head>
<body>
    <script src="bundle.cjs.js"></script>
</body>
</html>
```
### 1.4 支持babel 
- 为了使用新的语法，可以使用babel来进行编译输出
#### 1.4.1 安装依赖
- @babel/core是babel的核心包
- @babel/preset-env是预设
- rollup-plugin-babel是babel插件
```sh
cnpm install rollup-plugin-babel @babel/core @babel/preset-env --save-dev
```
#### 1.4.2 src\main.js
```js
let sum = (a,b)=>{
    return a+b;
}
let result = sum(1,2);
console.log(result);
```
#### 1.4.3 .babelrc
.babelrc
```json
{
    "presets": [
       [
        "@babel/env",
        {
            "modules":false
        }
       ]
    ]
}
```
#### 1.4.4 rollup.config.js
rollup.config.js
```js
+import babel from 'rollup-plugin-babel';
export default {
    input:'src/main.js',
    output:{
        file:'dist/bundle.cjs.js',//输出文件的路径和名称
        format:'cjs',//五种输出格式：amd/es6/iife/umd/cjs
        name:'bundleName'//当format为iife和umd时必须提供，将作为全局变量挂在window下
    },
+   plugins:[
+       babel({
+           exclude:"node_modules/**"
+       })
+   ]
}
```
### 1.5 tree-shaking
- Tree-shaking的本质是消除无用的js代码
- rollup只处理函数和顶层的import/export变量
#### 1.5.1 src\main.js
src\main.js
```js
import {name,age} from './msg';
console.log(name);
```
#### 1.5.2 src\msg.js
src\msg.js
```js
export var name = 'zhufeng';
export var age = 12;
```
### 1.6 使用第三方模块
- rollup.js编译源码中的模块引用默认只支持 ES6+的模块方式import/export
#### 1.6.1 安装依赖
```sh
cnpm install @rollup/plugin-node-resolve @rollup/plugin-commonjs lodash  --save-dev
```
#### 1.6.2 src\main.js
src\main.js
```js
import _ from 'lodash';
console.log(_);
```
#### 1.6.3 rollup.config.js
rollup.config.js
```js
import babel from 'rollup-plugin-babel';
+import resolve from '@rollup/plugin-node-resolve';
+import commonjs from '@rollup/plugin-commonjs';
export default {
    input:'src/main.js',
    output:{
        file:'dist/bundle.cjs.js',//输出文件的路径和名称
        format:'cjs',//五种输出格式：amd/es6/iife/umd/cjs
        name:'bundleName'//当format为iife和umd时必须提供，将作为全局变量挂在window下
    },
    plugins:[
        babel({
            exclude:"node_modules/**"
        }),
+       resolve(),
+       commonjs()
    ]
}
```
### 1.7 使用CDN
#### 1.7.1 src\main.js
src\main.js
```js
import _ from 'lodash';
import $ from 'jquery';
console.log(_.concat([1,2,3],4,5));
console.log($);
export default 'main';
```
#### 1.7.2 dist\index.html
dist\index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>rollup</title>
</head>
<body>
    <script src="https://cdn.jsdelivr.net/npm/lodash/lodash.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/jquery/jquery.min.js"></script>
    <script src="bundle.cjs.js"></script>
</body>
</html>
```
#### 1.7.3 rollup.config.js
rollup.config.js
```js
import babel from 'rollup-plugin-babel';
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
export default {
    input:'src/main.js',
    output:{
        file:'dist/bundle.cjs.js',//输出文件的路径和名称
+       format:'iife',//五种输出格式：amd/es6/iife/umd/cjs
+       name:'bundleName',//当format为iife和umd时必须提供，将作为全局变量挂在window下
+       globals:{
+           lodash:'_', //告诉rollup全局变量_即是lodash
+           jquery:'$' //告诉rollup全局变量$即是jquery
+       }
    },
    plugins:[
        babel({
            exclude:"node_modules/**"
        }),
        resolve(),
        commonjs()
    ],
+   external:['lodash','jquery']
}
```
### 1.8 使用typescript 
#### 1.8.1 安装
```sh
cnpm install tslib typescript @rollup/plugin-typescript --save-dev
```
#### 1.8.2 src\main.ts
src\main.ts
```ts
let myName:string = 'zhufeng';
let age:number=12;
console.log(myName,age);
```
#### 1.8.3 rollup.config.js
rollup.config.js
```js
import babel from 'rollup-plugin-babel';
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
+import typescript from '@rollup/plugin-typescript';
export default {
+   input:'src/main.ts',
    output:{
        file:'dist/bundle.cjs.js',//输出文件的路径和名称
        format:'iife',//五种输出格式：amd/es6/iife/umd/cjs
        name:'bundleName',//当format为iife和umd时必须提供，将作为全局变量挂在window下
        globals:{
            lodash:'_', //告诉rollup全局变量_即是lodash
            jquery:'$' //告诉rollup全局变量$即是jquery
        }
    },
    plugins:[
        babel({
            exclude:"node_modules/**"
        }),
        resolve(),
        commonjs(),
+       typescript()
    ],
    external:['lodash','jquery']
}
```
#### 1.8.4 tsconfig.json
tsconfig.json
```json
{
  "compilerOptions": {  
    "target": "es5",                          
    "module": "ESNext",                     
    "strict": true,                         
    "skipLibCheck": true,                    
    "forceConsistentCasingInFileNames": true 
  }
}
```
### 1.9 压缩JS
- terser是支持ES6 +的JavaScript压缩器工具包
#### 1.9.1 安装
```sh
cnpm install rollup-plugin-terser --save-dev
```
#### 1.9.2 rollup.config.js
rollup.config.js
```js
import babel from 'rollup-plugin-babel';
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import typescript from '@rollup/plugin-typescript';
+import {terser} from 'rollup-plugin-terser';
export default {
    input:'src/main.ts',
    output:{
        file:'dist/bundle.cjs.js',//输出文件的路径和名称
        format:'iife',//五种输出格式：amd/es6/iife/umd/cjs
        name:'bundleName',//当format为iife和umd时必须提供，将作为全局变量挂在window下
        globals:{
            lodash:'_', //告诉rollup全局变量_即是lodash
            jquery:'$' //告诉rollup全局变量$即是jquery
        }
    },
    plugins:[
        babel({
            exclude:"node_modules/**"
        }),
        resolve(),
        commonjs(),
        typescript(),
+       terser(),
    ],
    external:['lodash','jquery']
}
```
### 1.10 编译css
- terser是支持ES6 +的JavaScript压缩器工具包
#### 1.10.1 安装
```sh
cnpm install  postcss rollup-plugin-postcss --save-dev
```
#### 1.10.2 rollup.config.js
rollup.config.js
```js
import babel from 'rollup-plugin-babel';
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import typescript from '@rollup/plugin-typescript';
import {terser} from 'rollup-plugin-terser';
+import postcss from 'rollup-plugin-postcss';
export default {
    input:'src/main.js',
    output:{
        file:'dist/bundle.cjs.js',//输出文件的路径和名称
        format:'iife',//五种输出格式：amd/es6/iife/umd/cjs
        name:'bundleName',//当format为iife和umd时必须提供，将作为全局变量挂在window下
        globals:{
            lodash:'_', //告诉rollup全局变量_即是lodash
            jquery:'$' //告诉rollup全局变量$即是jquery
        }
    },
    plugins:[
        babel({
            exclude:"node_modules/**"
        }),
        resolve(),
        commonjs(),
        typescript(),
        //terser(),
+       postcss()
    ],
    external:['lodash','jquery']
}
```
#### 1.10.3 src\main.js
src\main.js
```js
import './main.css';
```
#### 1.10.4 src\main.css
src\main.css
```css
body{
    background-color: green;
}
```
### 1.11 本地服务器
#### 1.11.1 安装
```sh
cnpm install rollup-plugin-serve --save-dev
```
#### 1.11.2 rollup.config.dev.js
rollup.config.dev.js
```js
import babel from 'rollup-plugin-babel';
import resolve from '@rollup/plugin-node-resolve';
import commonjs from '@rollup/plugin-commonjs';
import typescript from '@rollup/plugin-typescript';
import postcss from 'rollup-plugin-postcss';
+import serve from 'rollup-plugin-serve';
export default {
    input:'src/main.js',
    output:{
        file:'dist/bundle.cjs.js',//输出文件的路径和名称
        format:'iife',//五种输出格式：amd/es6/iife/umd/cjs
        name:'bundleName',//当format为iife和umd时必须提供，将作为全局变量挂在window下
        sourcemap:true,
        globals:{
            lodash:'_', //告诉rollup全局变量_即是lodash
            jquery:'$' //告诉rollup全局变量$即是jquery
        }
    },
    plugins:[
        babel({
            exclude:"node_modules/**"
        }),
        resolve(),
        commonjs(),
        typescript(),
        postcss(),
+       serve({
+           open:true,
+           port:8080,
+           contentBase:'./dist'
+       })
    ],
    external:['lodash','jquery']
}
```
#### 1.11.3 package.json
package.json
```json
{
  "scripts": {
    "build": "rollup --config rollup.config.build.js",
    "dev": "rollup --config rollup.config.dev.js -w"
  },
}
```
## 2.前置知识
### 2.1. 初始化项目
```sh
cnpm install magic-string acorn --save
```
### 2.2. magic-string 
- [magic-string](https://www.npmjs.com/package/magic-string)是一个操作字符串和生成source-map的工具
```js
var MagicString = require('magic-string');
var magicString = new MagicString('export var name = "zhufeng"');
//返回magicString的克隆，删除原始字符串开头和结尾字符之前的所有内容
console.log(magicString.snip(0, 6).toString());
//从开始到结束删除字符(原始字符串,而不是生成的字符串)
console.log(magicString.remove(0,7).toString());
//使用MagicString.Bundle可以联合多个源代码
let bundleString = new MagicString.Bundle();
bundleString.addSource({
  content: 'var a = 1;',
  separator: '\n'
})
bundleString.addSource({
  content: 'var b = 2;',
  separator: '\n'
})
console.log(bundleString.toString());
```
### 2.3. AST
- 通过JavaScript Parser可以把代码转化为一颗抽象语法树AST,这颗树定义了代码的结构，通过操纵这颗树，我们可以精准的定位到声明语句、赋值语句、运算语句等等，实现对代码的分析、优化、变更等操作

![](/public/images/868f60df0f5227621151031e261678a9.jpg)

#### 2.3.1 AST工作流
- Parse(解析) 将源代码转换成抽象语法树，树上有很多的estree节点
- Transform(转换) 对抽象语法树进行转换
- Generate(代码生成) 将上一步经过转换过的抽象语法树生成新的代码

![](/public/images/1d821a22ff221e924731a6d8c8a654c4.png)

#### 2.3.2 acorn
- [astexplorer](https://astexplorer.net/)可以把代码转成语法树
- acorn 解析结果符合The Estree Spec规范

![](/public/images/d434186f338a42917a8ef3835df3a28f.png)
##### 2.3.2.1 walk.js
```js
function walk(ast, { enter, leave }) {
    visit(ast, null, enter, leave)
}
function visit(node, parent, enter, leave) {
    if (enter) {
        enter.call(null, node, parent)
    }
    let keys = Object.keys(node).filter(key => typeof node[key] === 'object');
    keys.forEach(key => {
        let value = node[key];
        if (Array.isArray(value)) {
            value.forEach(val => {
                visit(val, node, enter, leave);
            });
        } else if (value && value.type) {
            visit(value, node, enter, leave);
        }
    });
    if (leave) {
        leave(node, parent)
    }
}

module.exports = walk
```
##### 2.3.2.2 use.js
```js
const acorn = require("acorn")
let walk = require('./walk');
const ast = acorn.parse(`
import $ from 'jquery';
`, { locations: true, ranges: true, sourceType: 'module', ecmaVersion: 8 });
let ident = 0;
const padding = () => " ".repeat(ident);
ast.body.forEach(statement => {
    walk(statement, {
        enter(node) {
            if (node.type){
                console.log(padding() +node.type);
                ident+=2;
            }  
        },
        leave(node) {
            if (node.type){
                ident-=2;
                console.log(padding() +node.type);
            }
        }
    });
});
```

![](/public/images/d39f73349c0580b4bfe6aa106ef0b1ae.png)

```
ImportDeclaration
  ImportDefaultSpecifier
    Identifier
    Identifier
  ImportDefaultSpecifier
  Literal
  Literal
ImportDeclaration
```

### 2.4 作用域
#### 2.4.1 作用域
- 在JS中，作用域是用来规定变量访问范围的规则
```js
function one() {
  var a = 1;
}
console.log(a);
```
#### 2.4.2 作用域链
- 作用域链是由当前执行环境与上层执行环境的一系列变量对象组成的，它保证了当前执行环境对符合访问权限的变量和函数的有序访问
##### 2.4.2.1 scope.js 
scope.js
```js
class Scope {
    constructor(options = {}) {
        this.name = options.name;
        this.parent = options.parent;
        this.names = options.params || [];//存放着这个作用域内的所有变量
    }
    add(name) {
        this.names.push(name);
    }
    findDefiningScope(name) {
        if (this.names.includes(name)) {
            return this
        }
        if (this.parent) {
            return this.parent.findDefiningScope(name)
        }
        return null;
    }
}
module.exports = Scope;
```
##### 2.4.2.2 useScope.js
useScope.js
```js
let Scope = require('./scope');
var a = 1;
function one() {
    var b = 2;
    function two() {
        var c = 3;
        console.log(a, b, c);
    }
    two();
}
one();
let globalScope = new Scope({ name: 'global', params: ['a'], parent: null });
let oneScope = new Scope({ name: 'one', params: ['b'], parent: globalScope });
let twoScope = new Scope({ name: 'two', params: ['c'], parent: oneScope });

let aScope = twoScope.findDefiningScope('a');
console.log(aScope.name);
let bScope = twoScope.findDefiningScope('b');
console.log(bScope.name);
let cScope = twoScope.findDefiningScope('c');
console.log(cScope.name);
let dScope = twoScope.findDefiningScope('d');
console.log(dScope);
```
## 3. 实现rollup
### 3.1 目录结构
- [rollup代码仓库地址](https://gitee.com/zhufengpeixun/rollup)
```
.
├── package.json
├── README.md
├── src
    ├── ast
    │   ├── analyse.js //分析AST节点的作用域和依赖项
    │   ├── Scope.js //有些语句会创建新的作用域实例
    │   └── walk.js //提供了递归遍历AST语法树的功能
    ├── Bundle//打包工具，在打包的时候会生成一个Bundle实例，并收集其它模块，最后把所有代码打包在一起输出
    │   └── index.js 
    ├── Module//每个文件都是一个模块
    │   └── index.js
    ├── rollup.js //打包的入口模块
    └── utils
        ├── map-helpers.js
        ├── object.js
        └── promise.js
```
### 3.2 src\main.js
src\main.js
```js
console.log('hello');
```
### 3.3 debugger.js
```js
const path = require('path');
const rollup = require('./lib/rollup');
let entry = path.resolve(__dirname, 'src/main.js');
rollup(entry,'bundle.js');
```
### 3.4 rollup.js
lib\rollup.js
```js
const Bundle = require('./bundle')
const fs = require('fs')

function rollup(entry, filename) {
    const bundle = new Bundle({ entry });
    bundle.build(filename);
}
module.exports = rollup;
```
### 3.5 bundle.js
lib\bundle.js
```js
let fs = require('fs');
let path = require('path');
let Module = require('./module');
let MagicString = require('magic-string');
class Bundle {
    constructor(options) {
        //入口文件数据
        this.entryPath = path.resolve(options.entry.replace(/\.js$/, '') + '.js');
        this.modules = {};//存放所有的模块
    }
    build(filename) {
        let entryModule = this.fetchModule(this.entryPath);
        this.statements = entryModule.expandAllStatements(true);
        const { code } = this.generate({});
        fs.writeFileSync(filename, code);
    }
    fetchModule(importee) {
        let route = importee;
        if (route) {
            let code = fs.readFileSync(route, 'utf8');
            const module = new Module({
                code,
                path: importee,
                bundle: this
            })
            return module;
        }
    }
    generate(options) {
        let magicString = new MagicString.Bundle();
        this.statements.forEach(statement => {
            const source = statement._source.clone();
            magicString.addSource({
                content: source,
                separator: '\n'
            })

        })
        return { code: magicString.toString() }
    }
}
module.exports = Bundle;
```
### 3.6 module.js
lib\module.js
```js
const MagicString = require('magic-string');
const { parse } = require('acorn');
let analyse = require('./ast/analyse');
class Module {
    constructor({ code, path, bundle }) {
        this.code = new MagicString(code, { filename: path });
        this.path = path;
        this.bundle = bundle;
        this.ast = parse(code, {
            ecmaVersion: 7,
            sourceType: 'module'
        })
        this.analyse();
    }
    analyse() {
        analyse(this.ast, this.code, this);
    }
    expandAllStatements() {
        let allStatements = [];
        this.ast.body.forEach(statement => {
            let statements = this.expandStatement(statement);
            allStatements.push(...statements);
        });
        return allStatements;
    }
    expandStatement(statement) {
        statement._included = true;
        let result = [];
        result.push(statement);
        return result;
    }
}
module.exports = Module;
```
### 3.7 analyse.js
lib\ast\analyse.js
```js
function analyse(ast, magicString) {
    ast.body.forEach(statement => {
        Object.defineProperties(statement, {
            _source: { value: magicString.snip(statement.start, statement.end) }
        })
    });
}
module.exports = analyse;
```
## 4. 实现tree-shaking
### 4.1 msg.js
src\msg.js
```js
export var name = 'zhufeng';
export var age = 12;
```

![](/public/images/4324fd261744db779245477d4cc336a6.png)

### 4.2 src\main.js
src\main.js
```js
import {name,age} from './msg';
function say(){
    console.log('hello',name);
}
say();
```

![](/public/images/d6bcbc174f98e9aa834ea48397efb177.png)

### 4.3 bundle.js 
lib\bundle.js
```js
let fs = require('fs');
+let path = require('path');
let Module = require('./module');
let MagicString = require('magic-string');
class Bundle {
    constructor(options) {
        //入口文件数据
        this.entryPath = path.resolve(options.entry.replace(/\.js$/, '') + '.js');
        this.modules = {};//存放所有的模块
    }
    build(filename) {
        let entryModule = this.fetchModule(this.entryPath);
        this.statements = entryModule.expandAllStatements(true);
        const { code } = this.generate({});
        fs.writeFileSync(filename, code);
    }
+   fetchModule(importee,importer) {
+       let route;
+       if (!importer) {
+           route = importee;
+       }else{
+           if (path.isAbsolute(importee)) {
+               route = importee;
+           } else if (importee[0] == '.') {
+               route = path.resolve(path.dirname(importer), importee.replace(/\.js$/, '') + '.js')
+           }
+       }
        if (route) {
            let code = fs.readFileSync(route, 'utf8');
            const module = new Module({
                code,
                path: importee,
                bundle: this
            })
            return module;
        }
    }
    generate(options) {
        let magicString = new MagicString.Bundle();
        this.statements.forEach(statement => {
            const source = statement._source.clone();
+           if (/^Export/.test(statement.type)) {
+               if (statement.type === 'ExportNamedDeclaration' 
+                   && statement.declaration.type === 'VariableDeclaration') {
+                   source.remove(statement.start, statement.declaration.start);
+               }
+           }
            magicString.addSource({
                content: source,
                separator: '\n'
            })

        })
        return { code: magicString.toString() }
    }
}
module.exports = Bundle
```
### 4.4 scope.js
lib\ast\scope.js
```js
class Scope {
    constructor(options = {}) {
        this.parent = options.parent;
        this.names = options.params || [];//存放着这个作用域内的所有变量
    }
    add(name) {
        this.names.push(name);
    }
    findDefiningScope(name) {
        if (this.names.includes(name)) {
            return this
        }
        if (this.parent) {
            return this.parent.findDefiningScope(name)
        }
        return null;
    }
}
module.exports = Scope;
```
### 4.5 module.js
lib\module.js
```js
const MagicString = require('magic-string');
const { parse } = require('acorn');
let analyse = require('./ast/analyse');
+function hasOwnProperty(obj, prop) {
+    return Object.prototype.hasOwnProperty.call(obj, prop);
+}
class Module {
    constructor({ code, path, bundle }) {
        this.code = new MagicString(code, { filename: path });
        this.path = path;
        this.bundle = bundle;
        this.ast = parse(code, {
            ecmaVersion: 7,
            sourceType: 'module'
        })
        this.analyse();
    }
    analyse() {
+        this.imports = {};//导入
+        this.exports = {};//导出
+        this.ast.body.forEach(node => {
+            if (node.type === 'ImportDeclaration') {
+                let source = node.source.value;//./title
+                node.specifiers.forEach(specifier => {
+                    const localName = specifier.local.name;//name
+                    const name = specifier.imported.name;//name
+                    //this.imports['name']= {source:'./title',name:'name',localName:'name'};
+                    //从哪个模块导入了哪个变量，本地变量叫什么
+                    this.imports[localName] = { source, name, localName };
+                })
+            } else if (/^Export/.test(node.type)) {
+                if (node.type === 'ExportNamedDeclaration') {
+                    const declaration = node.declaration;
+                    if (declaration.type === 'VariableDeclaration') {
+                        let name = declaration.declarations[0].id.name;//name
+                        //this.exports['name']={node:ExportNamedDeclaration,localName:'name',expression:VariableDeclaration}
+                        //导出了哪个变量，通过哪个变量声明声明的，叫什么名字,导出节点是什么
+                        this.exports[name] = { node, localName: name, expression: declaration };
+                    }
+                }
+            }
+        })
        analyse(this.ast, this.code, this);
+       this.definitions = {};//存着所有的变量定义语句
+       this.ast.body.forEach(statement => {//收集所有语句上定义的变量,建立变量和声明语句之间的对应关系
+           Object.keys(statement._defines).forEach(name => {
+               this.definitions[name] = statement;
+           })
+       })
    }
    expandAllStatements() {
        let allStatements = [];
        this.ast.body.forEach(statement => {
+           if (statement.type === 'ImportDeclaration') return;
            let statements = this.expandStatement(statement);
            allStatements.push(...statements);
        });
        return allStatements;
    }
    expandStatement(statement) {
        statement._included = true;
        let result = [];
+       const dependencies = Object.keys(statement._dependsOn);
+       dependencies.forEach(name => {
+           let definition = this.define(name);
+           result.push(...definition)
+       });
        result.push(statement);
        return result;
    }
+   define(name) {
+       if (hasOwnProperty(this.imports, name)) {
+           const importDeclaration = this.imports[name];//获取 导入语句
+           let module = this.bundle.fetchModule(importDeclaration.source, this.path);//获取导入模块
+           const exportDeclaration = module.exports[importDeclaration.name];//获取那个模块的导出语句
+           if (!exportDeclaration) {
+               throw new Error(`Module ${module.path} does not export ${importDeclaration.name} (imported by ${this.path})`)
+           };
+           return module.define(exportDeclaration.localName);
+       } else {
+           let statement;
+           statement = this.definitions[name];
+           if (statement && !statement._included) {
+               return this.expandStatement(statement);
+           } else {
+               return [];
+           }
+       }
+   }
}
module.exports = Module
```
### 4.6 walk.js 
lib\ast\walk.js
```js
function walk(ast, { enter, leave }) {
    visit(ast, null, enter, leave)
}
function visit(node, parent, enter, leave) {
    if (enter) {
        enter.call(null, node, parent)
    }
    let keys = Object.keys(node).filter(key => typeof node[key] === 'object');
    keys.forEach(key => {
        let value = node[key];
        if (Array.isArray(value)) {
            value.forEach(val => {
                visit(val, node, enter, leave);
            });
+       } else if (value && value.type) {//节点必须存在，并且是一个有type的对象节点
            visit(value, node, enter, leave);
        }
    });
    if (leave) {
        leave(node, parent)
    }
}

module.exports = walk
```
### 4.7 analyse.js
lib\ast\analyse.js
```js
+const Scope = require('./scope');
+let walk = require('./walk');
function analyse(ast, magicString) {
+   let scope = new Scope();//创建作用域
+   ast.body.forEach(statement => {
+       function addToScope(declarator) {
+           var name = declarator.id.name;//声明的变量名
+           scope.add(name);//添加当前作用作用域中
+           if (!scope.parent) {//如果没有低级作用作用域说明是模块内的顶级作用域
+               statement._defines[name] = true;
+           }
+       }
+       Object.defineProperties(statement, {
+           _defines: { value: {} },//定义的变量
+           _dependsOn: { value: {} },//当前模块没有定义的变量，也就是外部依赖的变量
+           _included: { value: false, writable: true },//是否已经包含在输出语句中
+           _source: { value: magicString.snip(statement.start, statement.end) },
+       })
+       //收集每个statement上的定义的变量，创建作用域链
+       walk(statement, {
+           enter(node) {
+               let newScope;
+               switch (node.type) {
+                   case 'FunctionExpression':
+                       const names = node.params.map(getName);
+                       if (node.type === 'FunctionDeclaration') {
+                           addToScope(node)
+                       }
+                       newScope = new Scope({
+                           parent: scope,
+                           params: names
+                       })
+                       break;
+                   case 'VariableDeclaration':
+                       node.declarations.forEach(addToScope);
+                       break;
+               }
+               if (newScope) {
+                   Object.defineProperty(node, '_scope', { value: newScope })
+                   scope = newScope;
+               }
+           },
+           leave(node) {
+               if (node._scope) {
+                   scope = scope.parent;
+               }
+           }
+       });
+   });
+   ast.body.forEach(statement => {
+       walk(statement, {
+           enter(node) {
+               if (node._scope) scope = node._scope;
+               if (node.type === 'Identifier') {
+                   const definingScope = scope.findDefiningScope(node.name);
+                   //上级使用域内找不到，并且当前语句是没有定义这个变量
+                   if (!definingScope) {
+                       statement._dependsOn[node.name] = true;//添加外部依赖
+                   }
+               }
+           },
+            leave (node) {
+                if (node._scope) scope = scope.parent
+            }
+       })
+   });
}
module.exports = analyse;
```
## 5. 包含修改语句
### 5.1 src\index.js
src\index.js
```js
import {name,age} from './msg';
console.log(name);
```
### 5.2 msg.js
src\msg.js
```js
export var name = 'zhufeng';
+name += 'jiagou';
export var age = 12;
```
### 5.3 lib\module.js
lib\module.js
```js
const MagicString = require('magic-string');
const { parse } = require('acorn');
let analyse = require('./ast/analyse');
function hasOwnProperty(obj, prop) {
    return Object.prototype.hasOwnProperty.call(obj, prop);
}
class Module {
    constructor({ code, path, bundle }) {
        this.code = new MagicString(code, { filename: path });
        this.path = path;
        this.bundle = bundle;
        this.ast = parse(code, {
            ecmaVersion: 7,
            sourceType: 'module'
        })
        this.analyse();
    }
    analyse() {
        this.imports = {};//导入
        this.exports = {};//导出
+       this.modifications = {};
        this.ast.body.forEach(node => {
            if (node.type === 'ImportDeclaration') {
                let source = node.source.value;//./title
                node.specifiers.forEach(specifier => {
                    const localName = specifier.local.name;//name
                    const name = specifier.imported.name;//name
                    //this.imports['name']= {source:'./title',name:'name',localName:'name'};
                    //从哪个模块导入了哪个变量，本地变量叫什么
                    this.imports[localName] = { source, name, localName };
                })
            } else if (/^Export/.test(node.type)) {
                if (node.type === 'ExportNamedDeclaration') {
                    const declaration = node.declaration;
                    if (declaration.type === 'VariableDeclaration') {
                        let name = declaration.declarations[0].id.name;//name
                        //this.exports['name']={node:ExportNamedDeclaration,localName:'name',expression:VariableDeclaration}
                        //导出了哪个变量，通过哪个变量声明声明的，叫什么名字,导出节点是什么
                        this.exports[name] = { node, localName: name, expression: declaration };
                    }
                }
            }
        })
        analyse(this.ast, this.code, this);
        this.definitions = {};//存着所有的变量定义语句
        this.ast.body.forEach(statement => {//收集所有语句上定义的变量,建立变量和声明语句之间的对应关系
            Object.keys(statement._defines).forEach(name => {
                this.definitions[name] = statement;
            })
+           Object.keys(statement._modifies).forEach(name => {
+               if (!hasOwnProperty(this.modifications, name)) {
+                   this.modifications[name] = [];
+               }
+               this.modifications[name].push(statement);
+           })
+       })
    }
    expandAllStatements() {
        let allStatements = [];
        this.ast.body.forEach(statement => {
            if (statement.type === 'ImportDeclaration') return;
            let statements = this.expandStatement(statement);
            allStatements.push(...statements);
        });
        return allStatements;
    }
    expandStatement(statement) {
        statement._included = true;
        let result = [];
        const dependencies = Object.keys(statement._dependsOn);
        dependencies.forEach(name => {
            let definition = this.define(name);
            result.push(...definition);
        });
        result.push(statement);
+       const defines = Object.keys(statement._defines);
+       defines.forEach(name => {
+           const modifications = hasOwnProperty(this.modifications, name) && this.modifications[name];
+            if (modifications) {
+                modifications.forEach(statement => {
+                    if (!statement._included) {
+                        let statements = this.expandStatement(statement);
+                        result.push(...statements);
+                    }
+                });
+            }
+        });
        return result;
    }
    define(name) {
        if (hasOwnProperty(this.imports, name)) {
            const importDeclaration = this.imports[name];//获取 导入语句
            let module = this.bundle.fetchModule(importDeclaration.source, this.path);//获取导入模块
            const exportDeclaration = module.exports[importDeclaration.name];//获取那个模块的导出语句
            if (!exportDeclaration) {
                throw new Error(`Module ${module.path} does not export ${importDeclaration.name} (imported by ${this.path})`)
            };
            return module.define(exportDeclaration.localName);
        } else {
            let statement;
            statement = this.definitions[name];
            if (statement && !statement._included) {
                return this.expandStatement(statement);
            } else {
                return [];
            }
        }
    }
}
module.exports = Module
```
### 5.4 analyse.js
lib\ast\analyse.js
```js
const Scope = require('./scope');
let walk = require('./walk');
function analyse(ast, magicString) {
    let scope = new Scope();//创建作用域
    ast.body.forEach(statement => {
        function addToScope(declarator) {
            var name = declarator.id.name;//声明的变量名
            scope.add(name);//添加当前作用作用域中
            if (!scope.parent) {//如果没有低级作用作用域说明是模块内的顶级作用域
                statement._defines[name] = true;
            }
        }
        Object.defineProperties(statement, {
            _defines: { value: {} },//定义的变量
+           _modifies:{ value: {} },//修改
            _dependsOn: { value: {} },//当前模块没有定义的变量，也就是外部依赖的变量
            _included: { value: false, writable: true },//是否已经包含在输出语句中
            _source: { value: magicString.snip(statement.start, statement.end) },
        })
        //收集每个statement上的定义的变量，创建作用域链
        walk(statement, {
            enter(node) {
                let newScope;
                switch (node.type) {
                    case 'FunctionExpression':
                        const names = node.params.map(getName);
                        if (node.type === 'FunctionDeclaration') {
                            addToScope(node)
                        }
                        newScope = new Scope({
                            parent: scope,
                            params: names
                        })
                        break;
                    case 'VariableDeclaration':
                        node.declarations.forEach(addToScope);
                        break;
                }
                if (newScope) {
                    Object.defineProperty(node, '_scope', { value: newScope })
                    scope = newScope;
                }
            },
            leave(node) {
                if (node._scope) {
                    scope = scope.parent;
                }
            }
        });
    });
    ast.body.forEach(statement => {
+        function checkForReads (node) {
+            if (node.type === 'Identifier') {
+                const definingScope = scope.findDefiningScope(node.name);
+                //上级使用域内找不到，并且当前语句是没有定义这个变量
+                if (!definingScope) {
+                    statement._dependsOn[node.name] = true;//添加外部依赖
+                }
+            }
+        }
+        function checkForWrites(node) {
+            function addNode (node){
+                while (node.type === 'MemberExpression') {
+                    node = node.object;
+                }
+                if (node.type !== 'Identifier') {
+                    return
+                }
+                statement._modifies[node.name] = true;
+            }
+            if (node.type === 'AssignmentExpression') {// name='jiagou'
+                addNode(node.left, true)
+            }else if (node.type === 'UpdateExpression') {//name+='jiagou'
+                addNode(node.argument, true)
+            }
+        }
        walk(statement, {
+           enter(node) {
+               if (node._scope) scope = node._scope;
+               checkForReads(node)
+                checkForWrites(node)
+           },
+            leave (node) {
+              if (node._scope) scope = scope.parent
+            }
+       })
    });
}
module.exports = analyse;
```
## 6. 支持块级作用域
### 6.1 src\index.js
src\index.js
```js
if(true){
    var age = 12;
}
console.log(age);
```
### 6.2 scope.js
lib\ast\scope.js
```js
class Scope {
    constructor(options = {}) {
        this.parent = options.parent;
        this.names = options.params || [];//存放着这个作用域内的所有变量
+       this.isBlockScope = !!options.block// 是否块作用域
    }
+    add(name,isBlockDeclaration) {
+        if (!isBlockDeclaration && this.isBlockScope) {
+            //这是一个var或者函数声明，并且这是一个块级作用域，所以我们需要向上提升
+            this.parent.add(name, isBlockDeclaration)
+        } else {
+            this.names.push(name)
+        }
+    }
    findDefiningScope(name) {
        if (this.names.includes(name)) {
            return this
        }
        if (this.parent) {
            return this.parent.findDefiningScope(name)
        }
        return null;
    }
}
module.exports = Scope;
```
### 6.3 analyse.js
lib\ast\analyse.js
```js
const Scope = require('./scope');
let walk = require('./walk');
function analyse(ast, magicString) {
    let scope = new Scope();//创建作用域
    ast.body.forEach(statement => {
+       function addToScope(declarator,isBlockDeclaration=false) {
            var name = declarator.id.name;//声明的变量名
+           scope.add(name,isBlockDeclaration);//添加当前作用作用域中
            if (!scope.parent) {//如果没有低级作用作用域说明是模块内的顶级作用域
                statement._defines[name] = true;
            }
        }
        Object.defineProperties(statement, {
            _defines: { value: {} },//定义的变量
            _modifies:{ value: {} },//修改
            _dependsOn: { value: {} },//当前模块没有定义的变量，也就是外部依赖的变量
            _included: { value: false, writable: true },//是否已经包含在输出语句中
            _source: { value: magicString.snip(statement.start, statement.end) },
        })
        //收集每个statement上的定义的变量，创建作用域链
        walk(statement, {
            enter(node) {
                let newScope;
                switch (node.type) {
                    case 'FunctionExpression':
                        const names = node.params.map(getName);
                        if (node.type === 'FunctionDeclaration') {
                            addToScope(node)
                        }
                        newScope = new Scope({
                            parent: scope,
                            params: names,
+                           block: false
                        })
                        break;
+                   case 'BlockStatement':
+                       newScope = new Scope({
+                           parent: scope,
+                           block: true
+                       })
+                       break;
                    case 'VariableDeclaration':
+                        node.declarations.forEach((variableDeclarator)=>{
+                            if(node.kind === 'let'||node.kind === 'const'){
+                                addToScope(variableDeclarator,true);
+                            }else{
+                                addToScope(variableDeclarator)
+                            }
+                        });
                        break;
                }
                if (newScope) {
                    Object.defineProperty(node, '_scope', { value: newScope })
                    scope = newScope;
                }
            },
            leave(node) {
                if (node._scope) {
                    scope = scope.parent;
                }
            }
        });
    });
    ast.body.forEach(statement => {
        function checkForReads (node) {
            if (node.type === 'Identifier') {
                const definingScope = scope.findDefiningScope(node.name);
                //上级使用域内找不到，并且当前语句是没有定义这个变量
                if (!definingScope) {
                    statement._dependsOn[node.name] = true;//添加外部依赖
                }
            }
        }
        function checkForWrites(node) {
            function addNode (node){
                while (node.type === 'MemberExpression') {
                    node = node.object;
                }
                if (node.type !== 'Identifier') {
                    return
                }
                statement._modifies[node.name] = true;
            }
            if (node.type === 'AssignmentExpression') {// name='jiagou'
                addNode(node.left, true)
            }else if (node.type === 'UpdateExpression') {//name+='jiagou'
                addNode(node.argument, true)
            }
        }
        walk(statement, {
            enter(node) {
                if (node._scope) scope = node._scope;
                checkForReads(node);
                checkForWrites(node);
            },
            leave (node) {
                if (node._scope) scope = scope.parent
            }
        })
    });
}
module.exports = analyse;
```
### 6.4 module.js
lib\module.js
```js
const MagicString = require('magic-string');
const { parse } = require('acorn');
let analyse = require('./ast/analyse');
+const SYSTEM_VARIABLE = ['console','log'];
function hasOwnProperty(obj, prop) {
    return Object.prototype.hasOwnProperty.call(obj, prop);
}
class Module {
    constructor({ code, path, bundle }) {
        this.code = new MagicString(code, { filename: path });
        this.path = path;
        this.bundle = bundle;
        this.ast = parse(code, {
            ecmaVersion: 7,
            sourceType: 'module'
        })
        this.analyse();
    }
    analyse() {
        this.imports = {};//导入
        this.exports = {};//导出
        this.modifications = {};
        this.ast.body.forEach(node => {
            if (node.type === 'ImportDeclaration') {
                let source = node.source.value;//./title
                node.specifiers.forEach(specifier => {
                    const localName = specifier.local.name;//name
                    const name = specifier.imported.name;//name
                    //this.imports['name']= {source:'./title',name:'name',localName:'name'};
                    //从哪个模块导入了哪个变量，本地变量叫什么
                    this.imports[localName] = { source, name, localName };
                })
            } else if (/^Export/.test(node.type)) {
                if (node.type === 'ExportNamedDeclaration') {
                    const declaration = node.declaration;
                    if (declaration.type === 'VariableDeclaration') {
                        let name = declaration.declarations[0].id.name;//name
                        //this.exports['name']={node:ExportNamedDeclaration,localName:'name',expression:VariableDeclaration}
                        //导出了哪个变量，通过哪个变量声明声明的，叫什么名字,导出节点是什么
                        this.exports[name] = { node, localName: name, expression: declaration };
                    }
                }
            }
        })
        analyse(this.ast, this.code, this);
        this.definitions = {};//存着所有的变量定义语句
        this.ast.body.forEach(statement => {//收集所有语句上定义的变量,建立变量和声明语句之间的对应关系
            Object.keys(statement._defines).forEach(name => {
                this.definitions[name] = statement;
            })
            Object.keys(statement._modifies).forEach(name => {
                if (!hasOwnProperty(this.modifications, name)) {
                    this.modifications[name] = []
                }
                this.modifications[name].push(statement)
            })
        })
    }
    expandAllStatements() {
        let allStatements = [];
        this.ast.body.forEach(statement => {
            if (statement.type === 'ImportDeclaration') return;
            let statements = this.expandStatement(statement);
            allStatements.push(...statements);
        });
        return allStatements;
    }
    expandStatement(statement) {
        statement._included = true;
        let result = [];
        const dependencies = Object.keys(statement._dependsOn);
        dependencies.forEach(name => {
            let definition = this.define(name);
            result.push(...definition);
        });
        result.push(statement);
        const defines = Object.keys(statement._defines);
        defines.forEach(name => {
            const modifications = hasOwnProperty(this.modifications, name) && this.modifications[name];
            if (modifications) {
                modifications.forEach(statement => {
                    if (!statement._included) {
                        let statements = this.expandStatement(statement);
                        result.push(...statements);
                    }
                });
            }
        });
        return result;
    }
    define(name) {
        if (hasOwnProperty(this.imports, name)) {
            const importDeclaration = this.imports[name];//获取 导入语句
            let module = this.bundle.fetchModule(importDeclaration.source, this.path);//获取导入模块
            const exportDeclaration = module.exports[importDeclaration.name];//获取那个模块的导出语句
            if (!exportDeclaration) {
                throw new Error(`Module ${module.path} does not export ${importDeclaration.name} (imported by ${this.path})`)
            };
            return module.define(exportDeclaration.localName);
        } else {
            let statement;
            statement = this.definitions[name];
            if (statement && !statement._included) {
                return this.expandStatement(statement);
+           } else if(SYSTEM_VARIABLE.includes(name)) {
                return [];
+           }else{
+                throw new Error(`变量${name}没有既没有从外部导入，也没有在当前模块内声明！`)
+           }
        }
    }
}
module.exports = Module
```
## 7. 入口tree-shaking
### 7.1 src\index.js
src\index.js
```js
var name = 'zhufeng';
var age = 12;
console.log(age);
```
### 7.2 analyse.js
lib\ast\analyse.js
```js
const Scope = require('./scope');
let walk = require('./walk');
function analyse(ast, magicString) {
    let scope = new Scope();//创建作用域
    ast.body.forEach(statement => {
        function addToScope(declarator,isBlockDeclaration=false) {
            var name = declarator.id.name;//声明的变量名
            scope.add(name,isBlockDeclaration);//添加当前作用作用域中
            if (!scope.parent) {//如果没有低级作用作用域说明是模块内的顶级作用域
                statement._defines[name] = true;
            }
        }
        Object.defineProperties(statement, {
            _defines: { value: {} },//定义的变量
            _modifies:{ value: {} },//修改
            _dependsOn: { value: {} },//当前模块没有定义的变量，也就是外部依赖的变量
            _included: { value: false, writable: true },//是否已经包含在输出语句中
            _source: { value: magicString.snip(statement.start, statement.end) },
        })
        //收集每个statement上的定义的变量，创建作用域链
        walk(statement, {
            enter(node) {
                let newScope;
                switch (node.type) {
                    case 'FunctionExpression':
                        const names = node.params.map(getName);
                        if (node.type === 'FunctionDeclaration') {
                            addToScope(node)
                        }
                        newScope = new Scope({
                            parent: scope,
                            params: names,
                            block: false
                        })
                        break;
                    case 'BlockStatement':
                        newScope = new Scope({
                            parent: scope,
                            block: true
                        })
                        break;
                    case 'VariableDeclaration':
                        node.declarations.forEach((variableDeclarator)=>{
                            if(node.kind === 'let'||node.kind === 'const'){
                                addToScope(variableDeclarator,true);
                            }else{
                                addToScope(variableDeclarator)
                            }
                        });
                        break;
                }
                if (newScope) {
                    Object.defineProperty(node, '_scope', { value: newScope })
                    scope = newScope;
                }
            },
            leave(node) {
                if (node._scope) {
                    scope = scope.parent;
                }
            }
        });
    });
    ast.body.forEach(statement => {
        function checkForReads (node) {
            if (node.type === 'Identifier') {
                const definingScope = scope.findDefiningScope(node.name);
                //上级使用域内找不到，并且当前语句是没有定义这个变量
+               //if (!definingScope) {
                    statement._dependsOn[node.name] = true;//添加外部依赖
+               //}
            }
        }
        function checkForWrites(node) {
            function addNode (node){
                while (node.type === 'MemberExpression') {
                    node = node.object;
                }
                if (node.type !== 'Identifier') {
                    return
                }
                statement._modifies[node.name] = true;
            }
            if (node.type === 'AssignmentExpression') {// name='jiagou'
                addNode(node.left, true)
            }else if (node.type === 'UpdateExpression') {//name+='jiagou'
                addNode(node.argument, true)
            }
        }
        walk(statement, {
            enter(node) {
                if (node._scope) scope = node._scope;
                checkForReads(node);
                checkForWrites(node);
            },
            leave (node) {
                if (node._scope) scope = scope.parent
            }
        })
    });
}
module.exports = analyse;
```
### 7.3 module.js
lib\module.js
```js
const MagicString = require('magic-string');
const { parse } = require('acorn');
let analyse = require('./ast/analyse');
const SYSTEM_VARIABLE = ['console','log'];
function hasOwnProperty(obj, prop) {
    return Object.prototype.hasOwnProperty.call(obj, prop);
}
class Module {
    constructor({ code, path, bundle }) {
        this.code = new MagicString(code, { filename: path });
        this.path = path;
        this.bundle = bundle;
        this.ast = parse(code, {
            ecmaVersion: 7,
            sourceType: 'module'
        })
        this.analyse();
    }
    analyse() {
        this.imports = {};//导入
        this.exports = {};//导出
        this.modifications = {};
        this.ast.body.forEach(node => {
            if (node.type === 'ImportDeclaration') {
                let source = node.source.value;//./title
                node.specifiers.forEach(specifier => {
                    const localName = specifier.local.name;//name
                    const name = specifier.imported.name;//name
                    //this.imports['name']= {source:'./title',name:'name',localName:'name'};
                    //从哪个模块导入了哪个变量，本地变量叫什么
                    this.imports[localName] = { source, name, localName };
                })
            } else if (/^Export/.test(node.type)) {
                if (node.type === 'ExportNamedDeclaration') {
                    const declaration = node.declaration;
                    if (declaration.type === 'VariableDeclaration') {
                        let name = declaration.declarations[0].id.name;//name
                        //this.exports['name']={node:ExportNamedDeclaration,localName:'name',expression:VariableDeclaration}
                        //导出了哪个变量，通过哪个变量声明声明的，叫什么名字,导出节点是什么
                        this.exports[name] = { node, localName: name, expression: declaration };
                    }
                }
            }
        })
        analyse(this.ast, this.code, this);
        this.definitions = {};//存着所有的变量定义语句
        this.ast.body.forEach(statement => {//收集所有语句上定义的变量,建立变量和声明语句之间的对应关系
            Object.keys(statement._defines).forEach(name => {
                this.definitions[name] = statement;
            })
            Object.keys(statement._modifies).forEach(name => {
                if (!hasOwnProperty(this.modifications, name)) {
                    this.modifications[name] = []
                }
                this.modifications[name].push(statement)
            })
        })
    }
    expandAllStatements() {
        let allStatements = [];
        this.ast.body.forEach(statement => {
            if (statement.type === 'ImportDeclaration') return;
+           if (statement.type === 'VariableDeclaration') return;
            let statements = this.expandStatement(statement);
            allStatements.push(...statements);
        });
        return allStatements;
    }
    expandStatement(statement) {
        statement._included = true;
        let result = [];
        const dependencies = Object.keys(statement._dependsOn);
        dependencies.forEach(name => {
            let definition = this.define(name);
            result.push(...definition);
        });
        result.push(statement);
        const defines = Object.keys(statement._defines);
        defines.forEach(name => {
            const modifications = hasOwnProperty(this.modifications, name) && this.modifications[name];
            if (modifications) {
                modifications.forEach(statement => {
                    if (!statement._included) {
                        let statements = this.expandStatement(statement);
                        result.push(...statements);
                    }
                });
            }
        });
        return result;
    }
    define(name) {
        if (hasOwnProperty(this.imports, name)) {
            const importDeclaration = this.imports[name];//获取 导入语句
            let module = this.bundle.fetchModule(importDeclaration.source, this.path);//获取导入模块
            const exportDeclaration = module.exports[importDeclaration.name];//获取那个模块的导出语句
            if (!exportDeclaration) {
                throw new Error(`Module ${module.path} does not export ${importDeclaration.name} (imported by ${this.path})`)
            };
            return module.define(exportDeclaration.localName);
        } else {
            let statement;
+           statement = this.definitions[name];
+           if (statement) {
+               if(statement._included){
+                   return [];
+               }else{
+                   return this.expandStatement(statement);
+               }
            } else if(SYSTEM_VARIABLE.includes(name)) {
                return [];
            }else{
                throw new Error(`变量${name}没有既没有从外部导入，也没有在当前模块内声明！`)
            }
        }
    }
}
module.exports = Module
```
## 8. 实现变量名重命名
### 8.1 src\index.js
src\index.js
```js
import {age1} from './age1';
import {age2} from './age2';
console.log(age1,age2);
```
### 8.2 src\index.js
src\index.js
```js
const name = 'zhufeng';
export const name1 = name+'1';
```
### 8.3 src\index.js
src\index.js
```js
const name = 'zhufeng';
export const name2 = name+'1';
```
### 8.4 bundle.js
```js
const _age = '年龄';
const age2 = _age+'2';
const age = '年龄';
const age1 = age+'1';
console.log(age1,age2);
```
### 8.4 lib\utils.js
lib\utils.js
```js
let walk = require('./ast/walk');
function has(obj, prop) {
    return Object.prototype.hasOwnProperty.call(obj, prop);
}
function keys(obj) {
    return Object.keys(obj);
}
function replaceIdentifiers(statement, source, replacements) {
    walk(statement, {
        enter(node) {
            if (node.type === 'Identifier') {
                if(node.name && replacements[node.name]){
                    source.overwrite(node.start, node.end, replacements[node.name]);
                }
            }
        }
    })
}
module.exports = {has,keys,replaceIdentifiers};
```
### 8.5 analyse.js
lib\ast\analyse.js
```js
const Scope = require('./scope');
let walk = require('./walk');
+function analyse(ast, magicString,module) {
    let scope = new Scope();//创建作用域
    ast.body.forEach(statement => {
        function addToScope(declarator,isBlockDeclaration=false) {
            var name = declarator.id.name;//声明的变量名
            scope.add(name,isBlockDeclaration);//添加当前作用作用域中
            if (!scope.parent) {//如果没有低级作用作用域说明是模块内的顶级作用域
                statement._defines[name] = true;
            }
        }
        Object.defineProperties(statement, {
+           _module: { value: module },
            _defines: { value: {} },//定义的变量
            _modifies:{ value: {} },//修改
            _dependsOn: { value: {} },//当前模块没有定义的变量，也就是外部依赖的变量
            _included: { value: false, writable: true },//是否已经包含在输出语句中
            _source: { value: magicString.snip(statement.start, statement.end) },
        })
        //收集每个statement上的定义的变量，创建作用域链
        walk(statement, {
            enter(node) {
                let newScope;
                switch (node.type) {
                    case 'FunctionExpression':
                        const names = node.params.map(getName);
                        if (node.type === 'FunctionDeclaration') {
                            addToScope(node)
                        }
                        newScope = new Scope({
                            parent: scope,
                            params: names,
                            block: false
                        })
                        break;
                    case 'BlockStatement':
                        newScope = new Scope({
                            parent: scope,
                            block: true
                        })
                        break;
                    case 'VariableDeclaration':
                        node.declarations.forEach((variableDeclarator)=>{
                            if(node.kind === 'let'||node.kind === 'const'){
                                addToScope(variableDeclarator,true);
                            }else{
                                addToScope(variableDeclarator)
                            }
                        });
                        break;
                }
                if (newScope) {
                    Object.defineProperty(node, '_scope', { value: newScope })
                    scope = newScope;
                }
            },
            leave(node) {
                if (node._scope) {
                    scope = scope.parent;
                }
            }
        });
    });
    ast.body.forEach(statement => {
        function checkForReads (node) {
            if (node.type === 'Identifier') {
                const definingScope = scope.findDefiningScope(node.name);
                //上级使用域内找不到，并且当前语句是没有定义这个变量
                //if (!definingScope) {
                    statement._dependsOn[node.name] = true;//添加外部依赖
                //}
            }
        }
        function checkForWrites(node) {
            function addNode (node){
                while (node.type === 'MemberExpression') {
                    node = node.object;
                }
                if (node.type !== 'Identifier') {
                    return
                }
                statement._modifies[node.name] = true;
            }
            if (node.type === 'AssignmentExpression') {// name='jiagou'
                addNode(node.left, true)
            }else if (node.type === 'UpdateExpression') {//name+='jiagou'
                addNode(node.argument, true)
            }
        }
        walk(statement, {
            enter(node) {
                if (node._scope) scope = node._scope;
                checkForReads(node);
                checkForWrites(node);
            },
            leave (node) {
                if (node._scope) scope = scope.parent
            }
        })
    });
}
module.exports = analyse;
```
### 8.6 module.js
lib\module.js
```js
const MagicString = require('magic-string');
const { parse } = require('acorn');
let analyse = require('./ast/analyse');
const SYSTEM_VARIABLE = ['console','log'];
+let {has} = require('./utils');
class Module {
    constructor({ code, path, bundle }) {
        this.code = new MagicString(code, { filename: path });
        this.path = path;
        this.bundle = bundle;
+       this.canonicalNames = {};
        this.ast = parse(code, {
            ecmaVersion: 7,
            sourceType: 'module'
        })
        this.analyse();
    }
    analyse() {
        this.imports = {};//导入
        this.exports = {};//导出
        this.modifications = {};
        this.ast.body.forEach(node => {
            if (node.type === 'ImportDeclaration') {
                let source = node.source.value;//./title
                node.specifiers.forEach(specifier => {
                    const localName = specifier.local.name;//name
                    const name = specifier.imported.name;//name
                    //this.imports['name']= {source:'./title',name:'name',localName:'name'};
                    //从哪个模块导入了哪个变量，本地变量叫什么
                    this.imports[localName] = { source, name, localName };
                })
            } else if (/^Export/.test(node.type)) {
                if (node.type === 'ExportNamedDeclaration') {
                    const declaration = node.declaration;
                    if (declaration.type === 'VariableDeclaration') {
                        let name = declaration.declarations[0].id.name;//name
                        //this.exports['name']={node:ExportNamedDeclaration,localName:'name',expression:VariableDeclaration}
                        //导出了哪个变量，通过哪个变量声明声明的，叫什么名字,导出节点是什么
                        this.exports[name] = { node, localName: name, expression: declaration };
                    }
                }
            }
        })
        analyse(this.ast, this.code, this);
        this.definitions = {};//存着所有的变量定义语句
        this.ast.body.forEach(statement => {//收集所有语句上定义的变量,建立变量和声明语句之间的对应关系
            Object.keys(statement._defines).forEach(name => {
                this.definitions[name] = statement;
            })
            Object.keys(statement._modifies).forEach(name => {
                if (!has(this.modifications, name)) {
                    this.modifications[name] = []
                }
                this.modifications[name].push(statement)
            })
        })
    }
    expandAllStatements() {
        let allStatements = [];
        this.ast.body.forEach(statement => {
            if (statement.type === 'ImportDeclaration') return;
            if (statement.type === 'VariableDeclaration') return;
            let statements = this.expandStatement(statement);
            allStatements.push(...statements);
        });
        return allStatements;
    }
    expandStatement(statement) {
        statement._included = true;
        let result = [];
        const dependencies = Object.keys(statement._dependsOn);
        dependencies.forEach(name => {
            let definition = this.define(name);
            result.push(...definition);
        });
        result.push(statement);
        const defines = Object.keys(statement._defines);
        defines.forEach(name => {
            const modifications = has(this.modifications, name) && this.modifications[name];
            if (modifications) {
                modifications.forEach(statement => {
                    if (!statement._included) {
                        let statements = this.expandStatement(statement);
                        result.push(...statements);
                    }
                });
            }
        });
        return result;
    }
    define(name) {
        if (has(this.imports, name)) {
            const importDeclaration = this.imports[name];//获取 导入语句
            let module = this.bundle.fetchModule(importDeclaration.source, this.path);//获取导入模块
            const exportDeclaration = module.exports[importDeclaration.name];//获取那个模块的导出语句
            if (!exportDeclaration) {
                throw new Error(`Module ${module.path} does not export ${importDeclaration.name} (imported by ${this.path})`)
            };
            return module.define(exportDeclaration.localName);
        } else {
            let statement;
            statement = this.definitions[name];
            if (statement) {
                if(statement._included){
                    return [];
                }else{
                    return this.expandStatement(statement);
                }
            } else if(SYSTEM_VARIABLE.includes(name)) {
                return [];
            }else{
                throw new Error(`变量${name}没有既没有从外部导入，也没有在当前模块内声明！`)
            }
        }
    }
+    rename(name, replacement) {
+        this.canonicalNames[name] = replacement;
+    }
+    //获取规范定义的格式输出
+    getCanonicalName(localName) {
+        if (!has(this.canonicalNames, localName)) {
+            this.canonicalNames[localName] = localName;
+        }
+        return this.canonicalNames[localName];
+    }
}
module.exports = Module
```
### 8.7 lib\bundle.js
lib\bundle.js
```js
+let {has,keys,replaceIdentifiers} = require('./utils');
let fs = require('fs');
let path = require('path');
let Module = require('./module');
let MagicString = require('magic-string');
class Bundle {
    constructor(options) {
        //入口文件数据
        this.entryPath = path.resolve(options.entry.replace(/\.js$/, '') + '.js');
    }
    build(filename) {
        let entryModule = this.fetchModule(this.entryPath);
        this.statements = entryModule.expandAllStatements(true);
+        this.deConflict();
        const { code } = this.generate({});
        fs.writeFileSync(filename, code);
    }
+   deConflict() {
+       const defines = {};
+       const conflicts = {};
+       this.statements.forEach(statement => {
+           keys(statement._defines).forEach(name => {
+               if (has(defines, name)) {
+                   conflicts[name] = true;
+               } else {
+                   defines[name] = [];
+               }
+               defines[name].push(statement._module);
+           })
+       })
+       Object.keys(conflicts).forEach(name => {
+           const modules = defines[name];
+           modules.pop();
+           modules.forEach(module => {
+               const replacement = getSafeName(name)
+               module.rename(name, replacement)
+           })
+       })
+       function getSafeName(name) {
+           while (has(conflicts, name)) {
+               name = `_${name}`;
+           }
+           conflicts[name] = true;
+           return name;
+       }
+   }
    fetchModule(importee, importer) {
        let route;
        if (!importer) {
            route = importee;
        } else {
            if (path.isAbsolute(importee)) {
                route = importee;
            } else if (importee[0] == '.') {
                route = path.resolve(path.dirname(importer), importee.replace(/\.js$/, '') + '.js')
            }
        }
        if (route) {
            let code = fs.readFileSync(route, 'utf8');
            const module = new Module({
                code,
                path: importee,
                bundle: this
            })
            return module;
        }
    }
    generate(options) {
        let magicString = new MagicString.Bundle();
        this.statements.forEach(statement => {
+           let replacements = {};
+           Object.keys(statement._dependsOn)
+               .concat(Object.keys(statement._defines))
+               .forEach(name => {
+                   const canonicalName = statement._module.getCanonicalName(name)
+                   if (name !== canonicalName) {
+                       replacements[name] = canonicalName;
+                   }
+               })
            const source = statement._source.clone();
            if (/^Export/.test(statement.type)) {
                if (statement.type === 'ExportNamedDeclaration'
                    && statement.declaration.type === 'VariableDeclaration') {
                    source.remove(statement.start, statement.declaration.start);
                }
            }
+           replaceIdentifiers(statement, source, replacements);
            magicString.addSource({
                content: source,
                separator: '\n'
            })

        })
        return { code: magicString.toString() }
    }
}
module.exports = Bundle
```