## 1.webpack介绍
- Webpack是一个前端资源加载/打包工具。它将根据模块的依赖关系进行静态分析，然后将这些模块按照指定的规则生成对应的静态资源
![](/public/images/webpack_intro.gif)
## 2.预备知识
### 2.1 toString Tag
- Symbol.toStringTag 是一个内置 symbol，它通常作为对象的属性键使用，对应的属性值应该为字符串类型，这个字符串用来表示该对象的自定义类型标签
- 通常只有内置的 Object.prototype.toString() 方法会去读取这个标签并把它包含在自己的返回值里。
```js
console.log(Object.prototype.toString.call("foo")); // "[object String]"
console.log(Object.prototype.toString.call([1, 2])); // "[object Array]"
console.log(Object.prototype.toString.call(3)); // "[object Number]"
console.log(Object.prototype.toString.call(true)); // "[object Boolean]"
console.log(Object.prototype.toString.call(undefined)); // "[object Undefined]"
console.log(Object.prototype.toString.call(null)); // "[object Null]"
let myExports = {};
Object.defineProperty(myExports, Symbol.toStringTag, { value: "Module" });
console.log(Object.prototype.toString.call(myExports)); //[object Module]
```
```sh
[object String]
[object Array]
[object Number]
[object Boolean]
[object Undefined]
[object Null]
[object Module]
```
### 2.2 defineProperty
- [defineProperty]()方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回这个对象。
    - obj要在其上定义属性的对象。
    - prop 要定义或修改的属性的名称。
    - descriptor 将被定义或修改的属性描述符。
```js
let obj = {};
var ageValue = 10;

Object.defineProperty(obj, "age", {
    // writable: true, // 是否可修改
    // value: 10, //writeable 和 set 不能混用
    get() {
        return ageValue;
    },
    set(newValue) {
        ageValue = newValue;
    },

    enumerable: true, // 是否可枚举
    configurable: true, // 是否可配置可删除
});

console.log(obj.age);
obj.age = 20;
console.log(obj.age);
```
## 3.同步加载
### 3.1 安装模块
```sh
cnpm i webpack webpack-cli html-webpack-plugin clean-webpack-plugin -D
```
### 3.2 webpack.config.js
```js
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
module.exports = {
    mode: "development",
    devtool: "source-map",
    entry: "./src/index.js",
    output: {
        path: path.resolve(__dirname, "dist"),
        filename: "main.js",
    },
    module: {},
    plugins: [
        new CleanWebpackPlugin({ cleanOnceBeforeBuildPatterns: ["**/*"]}),
        new HtmlWebpackPlugin({
            template: "./src/index.html",
            filename: "index.html"
        }),
    ],
    devServer: {},
};
```
### 3.2 index.js
src\index.js
```js
let title = require("./title.js");
console.log(title);
```
### 3.3 title.js
src\title.js
```js
module.exports = "title";
```
### 3.4 index.html
src\index.html
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>webpack</title>
  </head>
  <body></body>
</html>
```
### 3.5 package.json
package.json
```json
"scripts": {
    "build": "webpack"
}
```
### 3.6 打包文件
```js
(() => {
    var modules = ({
        "./src/title.js": ((module) => {
            module.exports = "title";
        })
    });
    var cache = {};
    function require(moduleId) {
        if(cache[moduleId]) {
            return cache[moduleId].exports;
        }
        var module = cache[moduleId] = {
            exports: {}
        };
        modules[moduleId](module, module.exports, require);
        return module.exports;
    }
    (() => {
        let title = require("./src/title.js");
        console.log(title);
    })();
})();
```
## 4.兼容性实现
### 4.1 common.js加载common.js
#### 4.1.1 index.js
```js
let title = require("./title");
console.log(title.name);
console.log(title.age);
```
#### 4.1.2 title.js
```js
exports.name = "title_name";
exports.age = "title_age";
```
#### 4.1.3 main.js
```js
(() => {
    var modules = ({
        "./src/title.js":
        ((module, exports) => {
            exports.name = "title_name";
            exports.age = "title_age";
        })
    });
    var cache = {};
    function require(moduleId) {
        if(cache[moduleId]) {
            return cache[moduleId].exports;
        }
        var module = cache[moduleId] = {
            exports: {}
        };
        modules[moduleId](module, module.exports, require);
        return module.exports;
    }
    (() => {
        let title = require("./src/title.js");
        console.log(title.name);
        console.log(title.age);
    })();
})();
```
### 4.2 common.js 加载 ES6 modules
#### 4.2.1 index.js
```js
let title = require("./title");
console.log(title);
console.log(title.age);
```
#### 4.2.2 title.js
```js
export default = "title_name";
export const age = "title_age";
```
#### 4.2.3 main.js
```js
(() => {
    var modules = ({
        "./src/title.js":
        ((module, exports, require)) => {
            require.renderEsModule(exports);
            require.defineProperties(exports, {
                "default": () => DEFAULT_EXPORT,
                "age": () => age
            });
            const DEFAULT_EXPORT = "title_name";
            const age = "title_age";
        }
    });
    var cache = {};
    function require(moduleId) {
        if(cache[moduleId]) {
            return cache[moduleId].exports;
        }
        var module = cache[moduleId] = {
            exports: {}
        };
        modules[moduleId](module, module.exports, require);
        return module.exports;
    }
    require.defineProperties = (exports, definition) => {
        for(var key in definition) {
            Object.defineProperty(exports, key, { enumerable: true, get: definition[key] });
        }
    }
    require.renderEsModule = (exports) => {
        Object.defineProperty(exports, Symbol.toStringTag, { value: "Module" });
        Object.defineProperty(exports, '__esModule', { value: true });
    };
    (() => {
        let title = require("./src/title.js");
        console.log(title);
        console.log(title.age);
    })();
})();
```
### 4.3 ES6 modules 加载 ES6 modules
#### 4.3.1 index.js
```js
import name, { age } from "./title";
console.log(name);
console.log(age);
```
#### 4.3.2 title.js
```js
export default name = "title_name";
export const age = "title_age";
```
#### 4.3.3 main.js
```js
(() => {
    var modules = ({
        "./src/index.js":
        ((module,exports,require) => {
            require.renderEsModule(exports);
            var title = require("./src/title.js");
            console.log(title.default);
            console.log(title.age);
        }),
        "./src/title.js":
        ((module,exports,require) => {
            require.renderEsModule(exports);
            require.defineProperties(exports,{
                "default": () => DEFAULT_EXPORT,
                "age": () => age
            });
            const DEFAULT_EXPORT = (name = "title_age");
            const age = "title_age";
        });
    });
    var cache = {};
    function require(moduleId) {
        if(cache[moduleId]) {
            return cache[moduleId].exports;
        }
        var module = cache[moduleId] = {
            exports: {}
        };
        modules[moduleId](module, module.exports, require);
        return module.exports;
    }
    require.defineProperties = (exports, definition) => {
        for(var key in definition) {
            Object.defineProperty(exports, key, { enumerable: true, get: definition[key] });
        }
    };
    require.renderEsModule = (exports) => {
        Object.defineProperty(exports, Symbol.toStringTag, { value: "Module" });
        Object.defineProperty(exports, '__esModule', { value: true });
    };
    require("./src/index.js");
})();
```
### 4.4 ES6 modules 加载 common.js
#### 4.4.1 index.js
```js
import name, { age } from "./title";
console.log(name);
console.log(age);
```
#### 4.4.2 title.js
```js
module.exports = {
    name: "title_name",
    age: "title_age",
};
```
#### 4.4.3 main.js
```js
(() => {
    var modules = ({
        "./src/index.js":
        ((module, exports, require) => {
            require.renderEsModule(exports);
            var title = require("./src/title.js");
            var title_default = require.n(title);
            console.log(title_default());
            console.log(title.age);
        }),
        "./src/title.js":
        ((module) => {
            module.exports = {
                name: "title_name",
                age: "title_age",
            };
        })
    });
    var cache = {};
    function require(moduleId) {
        if(cache[moduleId]) {
            return cache[moduleId].exports;
        }
        var module = cache[moduleId] = {
            exports: {}
        };
        modules[moduleId](module, module.exports, require);
        return module.exports;
    }
    require.n = (module) => {
        var getter = module && module.__esModule ?
            () => module["default"] :
            () => module;
        return getter;
    };
    require.defineProperties = (exports, definition) => {
        for(var key in definition) {
            Object.defineProperty(exports, key, { enumerable: true, get: definition[key] });
        }
    };
    require.renderEsModule = (exports) => {
        Object.defineProperty(exports, Symbol.toStringTag, { value: "Module" });
        Object.defineProperty(exports, '__esModule', { value: true });
    };
    require("./src/index.js");
})();
```
## 5.异步加载
### 5.1 webpack.config.js
```js
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
module.exports = {
    mode: "development",
    devtool: "source-map",
    entry: "./src/index.js",
    output: {
        path: path.resolve(__dirname, "dist"),
        filename: "main.js",
    },
    module: {},
    plugins: [
        new CleanWebpackPlugin({ cleanOnceBeforeBuildPatterns: ["**/*"] }),
        new HtmlWebpackPlugin({
            template: "./src/index.html",
            filename: "index.html",
        }),
    ],
    devServer: {},
};
```
### 5.2 src\index.js
src\index.js
```js
import(/* webpackChunkName: "hello" */"./hello").then((result) => {
    console.log(result.default);
});
```
### 5.3 hello.js
src\hello.js
```js
export default "hello";
```
### 5.4 dist\main.js
```js
(() => {
    var modules = ({});
    var cache = {};
    function require(moduleId) {
        if(cache[moduleId]) {
            return cache[moduleId].exports;
        }
        var module = cache[moduleId] = {
            exports: {}
        };
        modules[moduleId](module, module.exports, require);
        return module.exports;
    }
    require.m = modules;
    require.defineProperties = (exports, definition) => {
        for(var key in definition) {
            if(require.ownProperty(definition, key) && !require.ownProperty(exports, key)) {
                Object.defineProperty(exports, key, { enumerable: true, get: definition[key] });
            }
        }
    };
    require.find = {};
    require.ensure = (chunkId) => {
        let promises = [];
        require.find.jsonp(chunkId, promises);
        return Promise.all(promises);
    };
    require.unionFileName = (chunkId) => {
        return "" + chunkId + ".main.js";
    };
    require.ownProperty = (obj, prop) => Object.prototype.hasOwnProperty.call(obj, prop);
    require.load = (url) => {
        var script = document.createElement("script");
        script.src = url;
        document.head.appendChild(script);
    };
    require.renderEsModule = (exports) => {
        if(typeof Symbol !== 'undefined' && Symbol.toStringTag) {
            Object.defineProperty(exports, Symbol.toStringTag, { value: "Module" });
        }
        Object.defineProperty(exports, '__esModule', { value: true });
    };
    require.publicPath = "";
    var installedChunks = {
        "main": 0
    };
    require.find.jsonp = (chunkId, promises) => {
        var promise = new Promise((resolve, reject) => {
            installedChunkData = installedChunks[chunkId] = [resolve, reject];
        });
        promises.push(installedChunkData[2] = promise);
        var url = require.publicPath + require.unionFileName(chunkId);
        require.load(url);
    };
    var webpackJsonpCallback = (data) => {
        var [chunkIds, moreModules] = data;
        var moduleId, chunkId, i = 0, resolves = [];
        for(; i< chunkIds.length; i++) {
            chunkId = chunkIds[i];
            resolves.push(installedChunks[chunkId][0]);
            installedChunks[chunkId] = 0;
        }
        for(moduleId in moreModules) {
            require.m[moduleId] = moreModules[moduleId];
        }
        while(resolves.length) {
            resolves.shift();
        }
    }
    var chunkLoadingGlobal = window["webpack5"] = window["webpack5"] || [];
    chunkLoadingGlobal.push = webpackJsonpCallback;
    require.ensure("hello").then(require.bind(require, "./src/hello.js")).then((result) => {
        console.log(result.default);
    });
})();
```
### 5.5 hello.main.js
hello.main.js
```js
(window["webpack5"] = window["webpack5"] || []).push([["hello"], {
    "./src/hello.js":
    ((module, exports, __webpack_require__) => {
        "use strict";
        __webpack_require__.renderEsModule(exports);
        __webpack_require__.defineProperties(exports, {
            "default": () => DEFAULT_EXPORT
        });
        const DEFAULT_EXPORT = ("hello");
    })
}]);
```