## 1.抽象语法树(Abstract Syntax Tree)
- webpack 和 Lint等很多的工具和库的核心都是通过Abstract Syntax Tree抽象语法树这个概念来实现对代码的检查、分析等操作的
## 2.抽象语法树用途
- 代码语法的检查、代码风格的检查、代码的格式化、代码的高亮、代码错误提示、代码自动补全等等
    - 如JSLint、JSHint对代码错误或风格的检查，发现一些潜在的错误
    - IDE的错误提示、格式化、高亮、自动补全等
- 代码混淆压缩
    - UglifyJS2等
- 优化变更代码，改变代码结构使达到想要的结构
    - 代码打包工具 webpack、rollup 等等
    - CommonJS、AMD、CMD、UMD等代码规范之间的转化
    - CoffeeScript、TypeScript、JSX等转化为原生Javascript
## 3.抽象语法树定义
- 这些工具的原理都是通过 JavaScript Parser 把代码转化为一颗抽象语法树(AST),这颗树定义了代码的结构，通过操纵这棵树，我们可以精准的定位到声明语句、赋值语句、运算语句等等，实现对代码的分析、优化、变更等操作
![](/public/images/ast.jpg)
## 4.JavaScript Parser
- JavaScript Parser是把JavaScript源码转化为抽象语法树的解析器。
- 浏览器会把JavaScript源码通过解析器转为抽象语法树，再进一步转化为字节码或直接生成机器码。
- 一般来说每个JavaScript引擎都会有自己的抽象语法树格式，Chrome的V8引擎，firefox的 SpiderMonkey引擎等等，MDN提供了详细[SpiderMonkey AST format](https://developer.mozilla.org/zh-CN/docs/MDN/Doc_status/SpiderMonkey)的详细说明，算是业界的标准。
### 4.1 常用的 JavaScript Parser
- esprima
- traceur
- acorn
- shift
### 4.2 遍历
```sh
cnpm i esprima estraverse -S
```
```js
let esprima = require("esprima"); // 把JS源代码转成AST语法树
let estraverse = require("estraverse"); // 遍历语法树，修改树上的节点
let escodegen = require("escodegen"); // 把AST语法树重新转换成代码
let code = `function ast() {}`;
let ast = esprima.parse(code);
let indent = 0;
const padding = () => " ".repeat(indent);
estraverse.traverse(ast, {
    enter(node) {
        console.log(padding()+node.type+"进入");
        if(node.type === 'FunctionDeclaration') {
            node.id.name = 'newAst';
        }
        index+=2;
    },
    leave(node) {
        indent -= 2;
        console.log(padding()+node.type+'离开');
    }
});
```
```sh
Program进入
  FunctionDeclaration进入
    Identifier进入
    Identifier离开
    BlockStatement进入
    BlockStatement离开
  FunctionDeclaration离开
Program离开
```
## 5.babel
- Babel能够转译 ECMAScript 2015+ 的代码，使它在旧的浏览器或者环境中也能够运行
- 工作过程分为三个部人
    - Parse(解析)将源代码转换成抽象语法树,树上有很多[estree节点](https://github.com/estree/estree)
    - Transform(转换)对抽象语法树进行转换
    - Generate(代码生成)将上一步经过转换过的抽象语法树生成新的代码
    ![](/public/images/ast-compiler-flow.png)
### 5.1 AST遍历
- AST是深度优先遍历
- 访问者模式 Visitor 对于某个对象或者一组对象，不同的访问者，产生的结果不同，执行操作也不同
- Visitor的对象定义了用于AST中获取具体节点的方法
- Visitor上挂载以节点 type 命名的方法，当遍历 AST 的时候，如果匹配上 type ,就会执行对应的方法
### 5.2 babel插件
- [@babel/core](https://www.npmjs.com/package/@babel/core)Babel的编译器,核心API都在这里面，比如常见的transform、parse
- babylon Babel的解析器
- [babel-types](https://github.com/babel/babel/tree/master/packages/babel-types)用于AST节点的Lodash式工具库，它包含了构造，验证以及变换AST节点的方法，对编写处理AST逻辑非常有用
- [babel-traverse](https://www.npmjs.com/package/babel-traverse)用于对AST的遍历，维护了整棵树的状态，并且负责替换、移除和添加节点
- [babel-types-api](https://babeljs.io/docs/en/next/babel-types.html)
- [Babel插件手册](https://github.com/brigand/babel-plugin-handbook/blob/master/translations/zh-Hans/README.md#asts)
- [babeljs.io](https://babeljs.io/en/repl.html)babel可视化编译器
#### 5.2.1 转换箭头函数
- [astexplorer](https://astexplorer.net/)
- [babel-plugin-transform-es2015-arrow-functions](https://www.npmjs.com/package/babel-plugin-transform-es2015-arrow-functions)
- [babeljs.io](https://babeljs.io/en/repl.html) babel可视化编译器
- [babel-handbook](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/README.md)
- [babel-types-api](https://babeljs.io/docs/en/next/babel-types.html)
转换前
```js
const sum = (a, b) => {
    console.log(this);
    return a + b;
}
```
转换后
```js
var _this = this;

const sum = function(a, b) {
    console.log(_this);
    return a + b;
};
```
```sh
npm i @babel/core babel-types -D
```
实现
```js
let core = require("@babel/core");
let types = require("babel-types");
let BabelPluginTransformEs2015ArrowFunctions = require('babel-plugin-transform-es2015-arrow-functions');
const sourceCode = `
    const sum = (a, b) => {
        console.log(this);
        return a + b;
    }
`;
// babel 插件其实是一个对象，它会有一个visotor访问器
let BabelPluginTransformEs2015ArrowFunctions2 = {
    // 每个插件都会有自己的访问器
    visitor: {
        // 属性就是节点的类型，babel在遍历到对应类型的节点的时候会调用此函数
        ArrowFunctionExpression(nodePath) { // 参数是节点的数据
            let node = nodePath.node; // 获取 当前路径上的节点
            // 处理 this 指针的问题
            hoistFunctionEnvironment(nodePath);
            node.type = 'FunctionExpression';
        }
    }
}
function hoistFunctionEnvironment(fnPath) {
    const thisEnvFn = fnPath.findParent(p => {
        // 是一个函数，不能是箭头函数 或者 是根节点也可以
        return (p.isFunction() && !p.isArrowFunctionExpression() || p.isProgram())
    });
    // 找一找当前作用域哪些地方用到了this的路径
    let thisPaths = getScopeInfoInformation(fnPath);
    // 声明了一个this的别名变量，默认是_this __this
    let thisBinding = '_this';
    if(thisPaths.length>0) {
        // 在thisEnvFn的作用域内添加一个变量，变量名_this，初始化的值为 this
        thisEnvFn.scope.push({
            id: types.identifier(thisBinding);
            init: types.thisExpression()
        });
        thisPaths.forEach(item => {
            // 创建一个 _this 的标识符
            let thisBindingRef = types.identifier(thisBinding);
            // 把老的路径 上的节点替换成新节点
            item.replaceWith(thisBindingRef);
        });
    }
}
function getScopeInfoInformation(fnPath) {
    let thisPaths = [];
    // 遍历当前 path 所有的子节点路径
    // 告诉 babel 我请帮我遍历 fnPath的子节点，遇到ThisExpression节点就执行函数，并且把对应的路径传进去
    fnPath.traverse({
        ThisExpression(thisPath){
            thisPaths.push(thisPath);
        }
    });
    return thisPaths;
}

let targetCode = core.transform(sourceCode, {
    plugins: [BabelPluginTransformEs2015ArrowFunctions2]
});
console.log(targetCode.code);
```
#### 5.2.2 把类编译为 Function
- [@babel/plugin-transform-classes](https://www.npmjs.com/package/@babel/plugin-transform-classes)
es6
```js
class Person {
    constructor(name) {
        this.name = name;
    }
    getName() {
        return this.name;
    }
}
```
![](/public/images/classast.png)
es5
```js
function Person(name) {
    this.name = name;
}
Person.prototype.getName = function() {
    return this.name;
};
```
![](/public/images/es5class1.png)
![](/public/images/es5class2.png)
实现
```js
// 核心库，提供了语法树的生成和遍历的功能
let babel = require("@babel/core");
// 工具类, 可能帮我们生成相应的节点
let t = require("babel-types");
// babel_plugin-transform-classes
let TransformClasses = require("@babel/plugin-transform-classes");
let esCode = `class Person {
    constructor(name) {
        this.name = name;
    }    
    getName() {
        return this.name;
    }
}`;
let transformClasses2 = {
    visitor: {
        ClassDeclaration(nodePath) {
            let {node} = nodePath;
            let id = node.id;//{type:'Identifier',name:'Person'}
            console.log(id);
            let methods = node.body.body;
            let nodes = [];
            methods.forEach(classMethod=>{
                if(classMethod.kind === 'constructor'){
                    let constructorFunction = t.functionDeclaration(
                        id, classMethod.params, classMethod.body,
                         classMethod.generator, classMethod.async);
                         nodes.push(constructorFunction);
                }else{
                    let prototypeMemberExpression = t.memberExpression(id, t.identifier('prototype'));
                    let keyMemberExpression = t.memberExpression(prototypeMemberExpression, classMethod.key);
                    let memberFunction = t.functionExpression(
                        id, classMethod.params, classMethod.body,
                         classMethod.generator, classMethod.async);
                    let assignmentExpression=t.assignmentExpression("=", 
                    keyMemberExpression,
                    memberFunction);
                    nodes.push(assignmentExpression);
                }
            });
            if(nodes.length==1){
                nodePath.replaceWith(nodes[0]);
            }else if(nodes.length>1){
                nodePath.replaceWithMultiple(nodes);
            }
        }
    }
}

let es5Code = babel.transform(es6Code,{
    plugins:[transformClasses2]
});
console.log(es5Code.code);
```
## 6.webpack TreeShaking插件
```js
var babel = require("@babel/core");
let { transform } = require("@babel/core");
```
### 6.1 实现按需加载
- [lodashjs](https://www.lodashjs.com/docs/4.17.5.html#concat)
- [babel-core](https://babeljs.io/docs/en/babel-core)
- [babel-plugin-import](https://www.npmjs.com/package/babel-plugin-import)
```js
import { flatten, concat } from "lodash";
```
![](/public/images/treeshakingleft.png)
转换为
```js
import flatten from "lodash/flatten";
import concat from "lodash/flatten";
```
![](/public/images/treeshakingright.png)
### 6.2 webpack 配置
```
cnpm install webpack webpack-cli -D
```
```js
const path = require("path");
module.exports = {
    mode: "development",
    entry: "./srcindex.js"
    output: {
        pah: path.resolve("dist"),
        filename: "bundle.js"
    },
    module: {
        rules: [
            {
                test: /\.js$/,
                use: {
                    loader: "babel-loader",
                    options: {
                        plugins: [
                            [
                                path.resolve(__dirname, "plubins/babel-plugin-import.js"),
                                {
                                    libraryName: "lodash"
                                }
                            ]
                        ]
                    }
                }
            }
        ]
    }
}
```
> 编译顺序为首先 plugins 从左往右，然后presets从右往左
### 6.3 babel插件
plubin/babel-plugin-import.js
```js
let babel = require("@babel/core");
let types = require("babel-types");
const visitor = {
  ImportDeclaration: {
    enter(path, state = { opts }) {
      const specifiers = path.node.specifiers;
      const source = path.node.source;
      if (
        state.opts.libraryName == source.value &&
        !types.isImportDefaultSpecifier(specifiers[0])
      ) {
        const declarations = specifiers.map((specifier, index) => {
          return types.ImportDeclaration(
            [types.importDefaultSpecifier(specifier.local)],
            types.stringLiteral(`${source.value}/${specifier.local.name}`)
          );
        });
        path.replaceWithMultiple(declarations);
      }
    },
  },
};
module.exports = function (babel) {
  return {
    visitor,
  };
};
```
## 7.参考
- [Babel插件手册](https://github.com/brigand/babel-plugin-handbook/blob/master/translations/zh-Hans/README.md#asts)
- [babel-types](https://github.com/babel/babel/tree/master/packages/babel-types)
- [不同的parser解析js代码后得到的AST](https://astexplorer.net/)
- [在线可视化的看到AST](http://resources.jointjs.com/demos/javascript-ast)
- [babel从入门到入门的知识归纳](https://zhuanlan.zhihu.com/p/28143410)
- [Babel内部原理分析](https://octman.com/blog/2016-08-27-babel-notes/)
- [babel-plugin-react-scope-binding](https://github.com/chikara-chan/babel-plugin-react-scope-binding)
- [transform-runtime](https://www.npmjs.com/package/babel-plugin-transform-runtime)Babel 默认只转换新的 JavaScript 语法，而不转换新的 API。例如，Iterator、Generator、Set、Maps、Proxy、Reflect、Symbol、Promise 等全局对象，以及一些定义在全局对象上的方法（比如 Object.assign）都不会转译,启用插件 babel-plugin-transform-runtime 后，Babel 就会使用 babel-runtime 下的工具函数
- [ast-spec](https://github.com/babel/babylon/blob/master/ast/spec.md)
- [babel-handbook](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/README.md)