## 1. webpack介绍
- `Webpack`是一个前端资源加载/打包工具。它将根据模块的依赖关系进行静态分析，然后将这些模块按照指定的规则生成对应的静态资源。
![](/public/images/webpack_intro.gif)

## 2.预备知识
### 2.1 toStringTag
- `Symbol.toStringTag` 是一个内置 symbol，它通常作为对象的属性键使用，对应的属性值应该为字符串类型，这个字符串用来表示该对象的自定义类型标签，通常只有内置的 `Object.prototype.toString()` 方法会去读取这个标签并把它包含在自己的返回值里。
```js
console.log(Object.prototype.toString.call('foo'));     // "[object String]"
console.log(Object.prototype.toString.call([1, 2]));    // "[object Array]"
console.log(Object.prototype.toString.call(3));         // "[object Number]"
console.log(Object.prototype.toString.call(true));      // "[object Boolean]"
console.log(Object.prototype.toString.call(undefined)); // "[object Undefined]"
console.log(Object.prototype.toString.call(null));      // "[object Null]"
let myExports={};
Object.defineProperty(myExports, Symbol.toStringTag, { value: 'Module' });
console.log(Object.prototype.toString.call(myExports));
```
### 2.2 Object.create(null)
- 使用`create`创建的对象，没有任何属性,把它当作一个非常纯净的map来使用，我们可以自己定义`hasOwnProperty`、`toString`方法,完全不必担心会将原型链上的同名方法覆盖掉
- 在我们使用`for..in`循环的时候会遍历对象原型链上的属性，使用`create(null)`就不必再对属性进行检查了
```js
var ns = Object.create(null);
if (typeof Object.create !== "function") {
    Object.create = function (proto) {
        function F() {}
        F.prototype = proto;
        return new F();
    };
}
console.log(ns)
console.log(Object.getPrototypeOf(ns));
```
### 2.3 getter
- [defineProperty](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性， 并返回这个对象。
    - obj 要在其上定义属性的对象。
    - prop 要定义或修改的属性的名称。
    - descriptor 将被定义或修改的属性描述符。
#### 2.3.1 描述符可同时具有的键值
| | configurable | enumerable | value | writable | get | set |
| --- | --- | --- | --- | --- | --- | --- |
| 数据描述符 | Yes | Yes | Yes | Yes | No | No |
| 存取描述符 | Yes | Yes No | No | Yes | Yes | |

#### 2.3.2 示例
```js
var ageValue;
Object.defineProperty(obj, "age", {
  value : 10,//数据描述符和存取描述符不能混合使用
  get(){
    return ageValue;
  },
  set(newValue){
    ageValue = newValue;
  }
  writable : true,//是否可修改
  enumerable : true,//是否可枚举
  configurable : true//是否可配置可删除
});
```
## 3.同步加载
```sh
cnpm i webpack webpack-cli html-webpack-plugin clean-webpack-plugin -D
```
### 3.1 webpack.config.js
```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
module.exports = {
    mode: 'development',
    devtool: 'none',
    entry: './src/index.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js'
    },
    module: {},
    plugins: [
        new CleanWebpackPlugin({ cleanOnceBeforeBuildPatterns: ['**/*'] }),
        new HtmlWebpackPlugin({
            template: './src/index.html',
            filename: 'index.html'
        })],
    devServer: {}
}
```
### 3.2 index.js
src\index.js
```js
let title = require('./title.js');
console.log(title);
```
### 3.3 title.js
src\title.js
```js
module.exports = "title";
```
### 3.4 打包文件分析
```js
(function(modules) {
  // webpack的启动函数
  //模块的缓存
  var installedModules = {};

  //定义在浏览器中使用的require方法
  function __webpack_require__(moduleId) {
    //检查模块是否在缓存中
    if (installedModules[moduleId]) {
      return installedModules[moduleId].exports;
    }
    //创建一个新的模块并且放到模块的缓存中
    var module = (installedModules[moduleId] = {
      i: moduleId,
      l: false,
      exports: {}
    });

    //执行模块函数
    modules[moduleId].call(
      module.exports,
      module,
      module.exports,
      __webpack_require__
    );

    //把模块设置为已经加载
    module.l = true;

    //返回模块的导出对象
    return module.exports;
  }

  //暴露出模块对象
  __webpack_require__.m = modules;

  //暴露出模块缓存
  __webpack_require__.c = installedModules;

  //为harmony导出定义getter函数
  __webpack_require__.d = function(exports, name, getter) {
    if (!__webpack_require__.o(exports, name)) {
      Object.defineProperty(exports, name, { enumerable: true, get: getter });
    }
  };

  //在导出对象上定义__esModule属性
  __webpack_require__.r = function(exports) {
    if (typeof Symbol !== "undefined" && Symbol.toStringTag) {
      Object.defineProperty(exports, Symbol.toStringTag, { value: "Module" });
    }
    Object.defineProperty(exports, "__esModule", { value: true });
  };

  /**
   * 创建一个模拟的命名空间对象
   * mode & 1 value是模块ID直接用__webpack_require__加载
   * mode & 2 把所有的属性合并到命名空间ns上
   * mode & 4 当已经是命名空间的时候(__esModule=true)可以直接返回值
   * mode & 8|1 行为类似于require
   */
  __webpack_require__.t = function(value, mode) {
    if (mode & 1) value = __webpack_require__(value);
    if (mode & 8) return value;
    if (mode & 4 && typeof value === "object" && value && value.__esModule)
      return value;
    var ns = Object.create(null); //定义一个空对象
    __webpack_require__.r(ns);
    Object.defineProperty(ns, "default", { enumerable: true, value: value });
    if (mode & 2 && typeof value != "string")
      for (var key in value)
        __webpack_require__.d(
          ns,
          key,
          function(key) {
            return value[key];
          }.bind(null, key)
        );
    return ns;
  };

  // getDefaultExport函数为了兼容那些非non-harmony模块
  __webpack_require__.n = function(module) {
    var getter =
      module && module.__esModule
        ? function getDefault() {
            return module["default"];
          }
        : function getModuleExports() {
            return module;
          };
    __webpack_require__.d(getter, "a", getter);
    return getter;
  };

  //判断对象身上是否拥有此属性
  __webpack_require__.o = function(object, property) {
    return Object.prototype.hasOwnProperty.call(object, property);
  };

  //公共路径
  __webpack_require__.p = "";

  //加载入口模块并且返回导出对象
  return __webpack_require__((__webpack_require__.s = "./src/index.js"));
})({
  "./src/index.js": function(module, exports, __webpack_require__) {
    var title = __webpack_require__("./src/title.js");
    console.log(title);
  },
  "./src/title.js": function(module, exports) {
    module.exports = "title";
  }
});
```
### 3.5 实现
```js
(function(modules){
    var installedModules = {};
    function __webpack_require__(moduleId){
        if(installedModules[moduleId]){
            return installedModules[moduleId];
        }
        var module = installedModules[moduleId] = {
            i:moduleId,
            l:false,
            exports:{}
        }
        modules[moduleId].call(modules.exports,module,module.exports,__webpack_require__);
        module.l = true;
        return module.exports;
    }
    return __webpack_require__((__webpack_require__.s = "./src/index.js"));
})({
    "./src/index.js":function(module,exports,__webpack_require__){
        var title = __webpack_require__('./src/title.js');
        console.log(title);
    },
    "./src/title.js":function(module,exports){
       module.exports = "title";
    }
})
```
## 4. harmony
### 4.1 common.js加载 common.js
#### 4.1.1 index.js
```js
let title = require('./title');
console.log(title.name);
console.log(title.age);
```
#### 4.1.2 title.js
```js
exports.name = 'title_name';
exports.age = 'title_age';
```
#### 4.1.3 bundle.js
```js
{
"./src/index.js":
  (function(module, exports, __webpack_require__) {
    var title = __webpack_require__("./src/title.js");
    console.log(title.name);
    console.log(title.age);
  }),
"./src/title.js":
  (function(module, exports) {
    exports.name = 'title_name';
    exports.age = 'title_age';
  })
}
```
### 4.2 common.js加载 ES6 modules
#### 4.2.1 index.js
```js
let title = require('./title');
console.log(title.name);
console.log(title.age);
```
#### 4.2.2 title.js
```js
exports.name = 'title_name';
exports.age = 'title_age';
```
#### 4.2.3 bundle.js
```js
{
 "./src/index.js":
 (function(module, exports, __webpack_require__) {
    var title = __webpack_require__("./src/title.js");
    console.log(title["default"]);
    console.log(title.age);
 }),
 "./src/title.js":
 (function(module, __webpack_exports__, __webpack_require__) {
    __webpack_require__.r(__webpack_exports__);//__esModule=true
    __webpack_require__.d(__webpack_exports__, "age", function() { return age; });
    __webpack_exports__["default"] = 'title_name';
    var age = 'title_age';
 })
}
```
### 4.3 ES6 modules 加载 ES6 modules
#### 4.3.1 index.js
```js
import name,{age} from './title';
console.log(name);
console.log(age);
```
#### 4.3.2 title.js
```js
export default name  = 'title_name';
export const age = 'title_age';
```
#### 4.3.3 bundle.js
```js
{
 "./src/index.js":
 (function(module, __webpack_exports__, __webpack_require__) {
    __webpack_require__.r(__webpack_exports__);//__esModule=true
    var _title__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__("./src/title.js");
    console.log(_title__WEBPACK_IMPORTED_MODULE_0__["default"]);
    console.log(_title__WEBPACK_IMPORTED_MODULE_0__["age"]);
 }),
 "./src/title.js":
 (function(module, __webpack_exports__, __webpack_require__) {
    __webpack_require__.r(__webpack_exports__);//__esModule=true
    __webpack_require__.d(__webpack_exports__, "age", function() { return age; });
    __webpack_exports__["default"] = 'title_name';
    var age = 'title_age';
 })
}
```
### 4.4 ES6 modules 加载 common.js
#### 4.4.1 index.js
```js
import name,{age} from './title';
console.log(name);
console.log(age);
```
#### 4.4.2 title.js
```js
export default name  = 'title_name';
export const age = 'title_age';
```
#### 4.4.3 bundle.js
```js
{
"./src/index.js":
(function(module, __webpack_exports__, __webpack_require__) {
  __webpack_require__.r(__webpack_exports__);//__esModule=true
  /* 兼容common.js导出 */ var _title__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__("./src/title.js");
  /* 兼容common.js导出 */ var _title__WEBPACK_IMPORTED_MODULE_0___default = __webpack_require__.n(_title__WEBPACK_IMPORTED_MODULE_0__);
  console.log(_title__WEBPACK_IMPORTED_MODULE_0___default.a.name);
  console.log(_title__WEBPACK_IMPORTED_MODULE_0___default.a.age);
}),
"./src/title.js":
(function(module, __webpack_exports__,__webpack_require__) {
  __webpack_exports__.name = 'title_name';
  __webpack_exports__.age = 'title_age';
}),
"./src/title_esm.js":
(function(module, __webpack_exports__,__webpack_require__) {
  __webpack_require__.r(__webpack_exports__);//__esModule=true
  __webpack_exports__.name = 'title_name';
  __webpack_exports__.age = 'title_age';
  __webpack_exports__.default = {name:'default_name',age:'default_age'};
})
}
```
## 5.异步加载
### 5.1 webpack.config.js
```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
module.exports = {
    mode: 'development',
    devtool: 'none',
    entry: './src/main.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: '[name].js'
    },
    module: {},
    plugins: [
        new CleanWebpackPlugin({ cleanOnceBeforeBuildPatterns: ['**/*'] }),
        new HtmlWebpackPlugin({
            template: './src/index.html',
            filename: 'index.html',
            chunks: ['main1']
        })],
    devServer: {}
}
```
### 5.3 src\main.js
```js
import(/* webpackChunkName: "c" */'./c').then(c => {
    console.log(c)
})
```
### 5.4 src\c.js
```js
export default {
    name: 'zhufeng'
}
```
### 5.5 dist\main.js
```js
(function (modules) {
  function webpackJsonpCallback(data) {
    var chunkIds = data[0];
    var moreModules = data[1];
    var moduleId, chunkId, i = 0, resolves = [];
    for (; i < chunkIds.length; i++) {
      chunkId = chunkIds[i];
      if (Object.prototype.hasOwnProperty.call(installedChunks, chunkId) && installedChunks[chunkId]) {
        resolves.push(installedChunks[chunkId][0]);
      }
      installedChunks[chunkId] = 0;
    }
    for (moduleId in moreModules) {
      if (Object.prototype.hasOwnProperty.call(moreModules, moduleId)) {
        modules[moduleId] = moreModules[moduleId];
      }
    }
    if (parentJsonpFunction) parentJsonpFunction(data);
    while (resolves.length) {
      resolves.shift()();
    }
  };
  var installedModules = {};
  var installedChunks = {
    "main": 0
  };
  function jsonpScriptSrc(chunkId) {
    return __webpack_require__.p + "" + ({ "c": "c" }[chunkId] || chunkId) + ".js"
  }
  function __webpack_require__(moduleId) {
    if (installedModules[moduleId]) {
      return installedModules[moduleId].exports;
    }
    var module = installedModules[moduleId] = {
      i: moduleId,
      l: false,
      exports: {}
    };
    modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
    module.l = true;
    return module.exports;
  }
  __webpack_require__.e = function requireEnsure(chunkId) {
    var promises = [];
    var installedChunkData = installedChunks[chunkId];
    if (installedChunkData !== 0) {
      if (installedChunkData) {
        promises.push(installedChunkData[2]);
      } else {
        var promise = new Promise(function (resolve, reject) {
          installedChunkData = installedChunks[chunkId] = [resolve, reject];
        });
        promises.push(installedChunkData[2] = promise);
        var script = document.createElement('script');
        var onScriptComplete;
        script.charset = 'utf-8';
        script.timeout = 120;
        if (__webpack_require__.nc) {
          script.setAttribute("nonce", __webpack_require__.nc);
        }
        script.src = jsonpScriptSrc(chunkId);
        var error = new Error();
        onScriptComplete = function (event) {
          script.onerror = script.onload = null;
          clearTimeout(timeout);
          var chunk = installedChunks[chunkId];
          if (chunk !== 0) {
            if (chunk) {
              var errorType = event && (event.type === 'load' ? 'missing' : event.type);
              var realSrc = event && event.target && event.target.src;
              error.message = 'Loading chunk ' + chunkId + ' failed.\n(' + errorType + ': ' + realSrc + ')';
              error.name = 'ChunkLoadError';
              error.type = errorType;
              error.request = realSrc;
              chunk[1](error);
            }
            installedChunks[chunkId] = undefined;
          }
        };
        var timeout = setTimeout(function () {
          onScriptComplete({ type: 'timeout', target: script });
        }, 120000);
        script.onerror = script.onload = onScriptComplete;
        document.head.appendChild(script);
      }
    }
    return Promise.all(promises);
  };
  __webpack_require__.m = modules;
  __webpack_require__.c = installedModules;
  __webpack_require__.d = function (exports, name, getter) {
    if (!__webpack_require__.o(exports, name)) {
      Object.defineProperty(exports, name, { enumerable: true, get: getter });
    }
  };
  __webpack_require__.r = function (exports) {
    if (typeof Symbol !== 'undefined' && Symbol.toStringTag) {
      Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
    }
    Object.defineProperty(exports, '__esModule', { value: true });
  };
  __webpack_require__.t = function (value, mode) {
    if (mode & 1) value = __webpack_require__(value);
    if (mode & 8) return value;
    if ((mode & 4) && typeof value === 'object' && value && value.__esModule) return value;
    var ns = Object.create(null);
    __webpack_require__.r(ns);
    Object.defineProperty(ns, 'default', { enumerable: true, value: value });
    if (mode & 2 && typeof value != 'string') for (var key in value) __webpack_require__.d(ns, key, function (key) { return value[key]; }.bind(null, key));
    return ns;
  };
  __webpack_require__.n = function (module) {
    var getter = module && module.__esModule ?
      function getDefault() { return module['default']; } :
      function getModuleExports() { return module; };
    __webpack_require__.d(getter, 'a', getter);
    return getter;
  };
  __webpack_require__.o = function (object, property) { return Object.prototype.hasOwnProperty.call(object, property); };
  __webpack_require__.p = "";
  __webpack_require__.oe = function (err) { console.error(err); throw err; };
  var jsonpArray = window["webpackJsonp"] = window["webpackJsonp"] || [];
  var oldJsonpFunction = jsonpArray.push.bind(jsonpArray);
  jsonpArray.push = webpackJsonpCallback;
  jsonpArray = jsonpArray.slice();
  for (var i = 0; i < jsonpArray.length; i++) webpackJsonpCallback(jsonpArray[i]);
  var parentJsonpFunction = oldJsonpFunction;
  return __webpack_require__(__webpack_require__.s = "./src/main.js");
})
  ({
    "./src/main.js":
      (function (module, exports, __webpack_require__) {
        __webpack_require__.e("c").then(__webpack_require__.bind(null, "./src/c.js")).then(c => {
          console.log(c)
        })
      })
  });
```
### 5.6 dist\c.js
```js
(window["webpackJsonp"] = window["webpackJsonp"] || []).push([["c"], {
  "./src/c.js":
    (function (module, __webpack_exports__, __webpack_require__) {
      "use strict";
      __webpack_require__.r(__webpack_exports__);
      __webpack_exports__["default"] = ({
        name: 'zhufeng'
      });
    })
}]);
```
### 5.7 实现
```js
(function (modules) {
  function webpackJsonpCallback(data) {
    var chunkIds = data[0];
    var moreModules = data[1];
    var moduleId, chunkId, i = 0, resolves = [];
    for (; i < chunkIds.length; i++) {
      chunkId = chunkIds[i];
      if (installedChunks[chunkId]) {
        resolves.push(installedChunks[chunkId][0]);
      }
      installedChunks[chunkId] = 0;
    }
    for (moduleId in moreModules) {
      modules[moduleId] = moreModules[moduleId];
    }
    while (resolves.length) {
      resolves.shift()();
    }
  }
  var installedModules = {};
  var installedChunks = { main: 0 };
  __webpack_require__.p = "";
  function jsonpScriptSrc(chunkId) {
    return __webpack_require__.p + "" + chunkId + ".bundle.js";
  }
  function __webpack_require__(moduleId) {
    if (installedModules[moduleId]) {
      return installedModules[moduleId].exports;
    }
    var module = (installedModules[moduleId] = { i: moduleId, l: false, exports: {} });
    modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
    module.l = true;
    return module.exports;
  }
  __webpack_require__.t = function (value, mode) {
    value = __webpack_require__(value);
    var ns = Object.create(null);
    Object.defineProperty(ns, "__esModule", { value: true });
    Object.defineProperty(ns, "default", { enumerable: true, value: value });
    return ns;
  };

  __webpack_require__.e = function requireEnsure(chunkId) {
    var promises = [];
    var installedChunkData = installedChunks[chunkId];
    var promise = new Promise(function (resolve, reject) {
      installedChunkData = installedChunks[chunkId] = [resolve, reject];
    });
    promises.push((installedChunkData[2] = promise));
    var script = document.createElement("script");
    script.src = jsonpScriptSrc(chunkId);
    document.head.appendChild(script);
    return Promise.all(promises);
  }
  var jsonpArray = (window["webpackJsonp"] = window["webpackJsonp"] || []);
  jsonpArray.push = webpackJsonpCallback;
  return __webpack_require__((__webpack_require__.s = "./src/index.js"));
})({
  "./src/main.js":
    (function (module, exports, __webpack_require__) {
      __webpack_require__.e("c").then(__webpack_require__.bind(null, "./src/c.js")).then(c => {
        console.log(c)
      })
    })
});
```
## 6.异步加载

![](/public/images/webpacklazyload2.png)

### 6.1 src\index.html
```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN">
<html>
<head>
    <title>import</title>
</head>
<body>
<button id="main1-loadC1">main1中加载c1模块</button>
<button id="main1-loadC2">main1中加载c2模块</button>
<button id="loadMain2">main2.js文件</button>
<button id="main2-loadC1">main2中加载c1模块</button>
<button id="main2-loadC2">main2中加载c2模块</button>
</body>
</html>
```
### 6.2 webpack.config.js 
```js
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
module.exports = {
    mode: 'development',
    devtool: 'none',
    entry: {
        main1: './src/main1.js',
        main2: './src/main2.js'
    },
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: '[name].js'
    },
    module: {},
    plugins: [
        new CleanWebpackPlugin({ cleanOnceBeforeBuildPatterns: ['**/*'] }),
        new HtmlWebpackPlugin({
            template: './src/index.html',
            filename: 'index.html',
            chunks: ['main1']
        })],
    devServer: {}
}
```
### 6.3 src\main1.js
```js
document.getElementById('main1-loadC1').addEventListener('click', () => {
    import(/* webpackChunkName: "c1" */'./c1').then(c1 => {
        console.log(c1.default)
    })
});
document.getElementById('main1-loadC2').addEventListener('click', () => {
    import(/* webpackChunkName: "c2" */'./c2').then(c1 => {
        console.log(c1.default)
    })
});
document.getElementById('loadMain2').addEventListener('click', () => {
    let script = document.createElement('script');
    script.src = 'main2.js';
    document.body.appendChild(script);
});
```
### 6.4 src\main2.js 
```js
document.getElementById('main2-loadC1').addEventListener('click', () => {
    import(/* webpackChunkName: "c1" */'./c1').then(c1 => {
        console.log(c1.default)
    })
});
document.getElementById('main2-loadC2').addEventListener('click', () => {
    import(/* webpackChunkName: "c2" */'./c2').then(c1 => {
        console.log(c1.default)
    })
});
```
### 6.5 src\c1.js
```js
export default {
    name: 'c1'
}
```
### 6.6 src\c2.js
```js
export default {
    name: 'c2'
}
```
### 6.7 dist\main1.js
```js
(function (modules) {
    function webpackJsonpCallback(data) {
        var chunkIds = data[0];
        var moreModules = data[1];
        var moduleId, chunkId, i = 0, resolves = [];
        for (; i < chunkIds.length; i++) {
            chunkId = chunkIds[i];
            if (Object.prototype.hasOwnProperty.call(installedChunks, chunkId) && installedChunks[chunkId]) {
                resolves.push(installedChunks[chunkId][0]);
            }
            installedChunks[chunkId] = 0;
        }
        for (moduleId in moreModules) {
            if (Object.prototype.hasOwnProperty.call(moreModules, moduleId)) {
                modules[moduleId] = moreModules[moduleId];
            }
        }
        if (parentJsonpFunction) parentJsonpFunction(data);
        while (resolves.length) {
            resolves.shift()();
        }
    };
    var installedModules = {};
    var installedChunks = {
        "main1": 0
    };
    function jsonpScriptSrc(chunkId) {
        return __webpack_require__.p + "" + ({ "c1": "c1", "c2": "c2" }[chunkId] || chunkId) + ".js"
    }
    function __webpack_require__(moduleId) {
        if (installedModules[moduleId]) {
            return installedModules[moduleId].exports;
        }
        var module = installedModules[moduleId] = {
            i: moduleId,
            l: false,
            exports: {}
        };
        modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
        module.l = true;
        return module.exports;
    }
    __webpack_require__.e = function requireEnsure(chunkId) {
        var promises = [];
        var installedChunkData = installedChunks[chunkId];
        if (installedChunkData !== 0) {
            if (installedChunkData) {
                promises.push(installedChunkData[2]);
            } else {
                var promise = new Promise(function (resolve, reject) {
                    installedChunkData = installedChunks[chunkId] = [resolve, reject];
                });
                promises.push(installedChunkData[2] = promise);
                var script = document.createElement('script');
                var onScriptComplete;
                script.charset = 'utf-8';
                script.timeout = 120;
                if (__webpack_require__.nc) {
                    script.setAttribute("nonce", __webpack_require__.nc);
                }
                script.src = jsonpScriptSrc(chunkId);
                var error = new Error();
                onScriptComplete = function (event) {
                    script.onerror = script.onload = null;
                    clearTimeout(timeout);
                    var chunk = installedChunks[chunkId];
                    if (chunk !== 0) {
                        if (chunk) {
                            var errorType = event && (event.type === 'load' ? 'missing' : event.type);
                            var realSrc = event && event.target && event.target.src;
                            error.message = 'Loading chunk ' + chunkId + ' failed.\n(' + errorType + ': ' + realSrc + ')';
                            error.name = 'ChunkLoadError';
                            error.type = errorType;
                            error.request = realSrc;
                            chunk[1](error);
                        }
                        installedChunks[chunkId] = undefined;
                    }
                };
                var timeout = setTimeout(function () {
                    onScriptComplete({ type: 'timeout', target: script });
                }, 120000);
                script.onerror = script.onload = onScriptComplete;
                document.head.appendChild(script);
            }
        }
        return Promise.all(promises);
    };
    __webpack_require__.m = modules;
    __webpack_require__.c = installedModules;
    __webpack_require__.d = function (exports, name, getter) {
        if (!__webpack_require__.o(exports, name)) {
            Object.defineProperty(exports, name, { enumerable: true, get: getter });
        }
    };
    __webpack_require__.r = function (exports) {
        if (typeof Symbol !== 'undefined' && Symbol.toStringTag) {
            Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
        }
        Object.defineProperty(exports, '__esModule', { value: true });
    };
    __webpack_require__.t = function (value, mode) {
        if (mode & 1) value = __webpack_require__(value);
        if (mode & 8) return value;
        if ((mode & 4) && typeof value === 'object' && value && value.__esModule) return value;
        var ns = Object.create(null);
        __webpack_require__.r(ns);
        Object.defineProperty(ns, 'default', { enumerable: true, value: value });
        if (mode & 2 && typeof value != 'string') for (var key in value) __webpack_require__.d(ns, key, function (key) { return value[key]; }.bind(null, key));
        return ns;
    };
    __webpack_require__.n = function (module) {
        var getter = module && module.__esModule ?
            function getDefault() { return module['default']; } :
            function getModuleExports() { return module; };
        __webpack_require__.d(getter, 'a', getter);
        return getter;
    };
    __webpack_require__.o = function (object, property) { return Object.prototype.hasOwnProperty.call(object, property); };
    __webpack_require__.p = "";
    __webpack_require__.oe = function (err) { console.error(err); throw err; };
    var jsonpArray = window["webpackJsonp"] = window["webpackJsonp"] || [];
    var oldJsonpFunction = jsonpArray.push.bind(jsonpArray);
    jsonpArray.push = webpackJsonpCallback;
    jsonpArray = jsonpArray.slice();
    for (var i = 0; i < jsonpArray.length; i++) webpackJsonpCallback(jsonpArray[i]);
    var parentJsonpFunction = oldJsonpFunction;
    return __webpack_require__(__webpack_require__.s = "./src/main1.js");
})
    ({
        "./src/main1.js":
            (function (module, exports, __webpack_require__) {
                document.getElementById('main1-loadC1').addEventListener('click', () => {
                    __webpack_require__.e("c1").then(__webpack_require__.bind(null, "./src/c1.js")).then(c1 => {
                        console.log(c1.default)
                    })
                });
                document.getElementById('main1-loadC2').addEventListener('click', () => {
                    __webpack_require__.e("c2").then(__webpack_require__.bind(null, "./src/c2.js")).then(c1 => {
                        console.log(c1.default)
                    })
                });
                document.getElementById('loadMain2').addEventListener('click', () => {
                    let script = document.createElement('script');
                    script.src = 'main2.js';
                    document.body.appendChild(script);
                });
            })
    });
```
### 6.8 dist\main2.js
```js
(function (modules) {
    function webpackJsonpCallback(data) {
        var chunkIds = data[0];
        var moreModules = data[1];
        var moduleId, chunkId, i = 0, resolves = [];
        for (; i < chunkIds.length; i++) {
            chunkId = chunkIds[i];
            if (Object.prototype.hasOwnProperty.call(installedChunks, chunkId) && installedChunks[chunkId]) {
                resolves.push(installedChunks[chunkId][0]);
            }
            installedChunks[chunkId] = 0;
        }
        for (moduleId in moreModules) {
            if (Object.prototype.hasOwnProperty.call(moreModules, moduleId)) {
                modules[moduleId] = moreModules[moduleId];
            }
        }
        if (parentJsonpFunction) parentJsonpFunction(data);
        while (resolves.length) {
            resolves.shift()();
        }
    };
    var installedModules = {};
    var installedChunks = {
        "main2": 0
    };
    function jsonpScriptSrc(chunkId) {
        return __webpack_require__.p + "" + ({ "c1": "c1", "c2": "c2" }[chunkId] || chunkId) + ".js"
    }
    function __webpack_require__(moduleId) {
        if (installedModules[moduleId]) {
            return installedModules[moduleId].exports;
        }
        var module = installedModules[moduleId] = {
            i: moduleId,
            l: false,
            exports: {}
        };
        modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
        module.l = true;
        return module.exports;
    }
    __webpack_require__.e = function requireEnsure(chunkId) {
        var promises = [];
        var installedChunkData = installedChunks[chunkId];
        if (installedChunkData !== 0) {
            if (installedChunkData) {
                promises.push(installedChunkData[2]);
            } else {
                var promise = new Promise(function (resolve, reject) {
                    installedChunkData = installedChunks[chunkId] = [resolve, reject];
                });
                promises.push(installedChunkData[2] = promise);
                var script = document.createElement('script');
                var onScriptComplete;
                script.charset = 'utf-8';
                script.timeout = 120;
                if (__webpack_require__.nc) {
                    script.setAttribute("nonce", __webpack_require__.nc);
                }
                script.src = jsonpScriptSrc(chunkId);
                var error = new Error();
                onScriptComplete = function (event) {
                    script.onerror = script.onload = null;
                    clearTimeout(timeout);
                    var chunk = installedChunks[chunkId];
                    if (chunk !== 0) {
                        if (chunk) {
                            var errorType = event && (event.type === 'load' ? 'missing' : event.type);
                            var realSrc = event && event.target && event.target.src;
                            error.message = 'Loading chunk ' + chunkId + ' failed.\n(' + errorType + ': ' + realSrc + ')';
                            error.name = 'ChunkLoadError';
                            error.type = errorType;
                            error.request = realSrc;
                            chunk[1](error);
                        }
                        installedChunks[chunkId] = undefined;
                    }
                };
                var timeout = setTimeout(function () {
                    onScriptComplete({ type: 'timeout', target: script });
                }, 120000);
                script.onerror = script.onload = onScriptComplete;
                document.head.appendChild(script);
            }
        }
        return Promise.all(promises);
    };
    __webpack_require__.m = modules;
    __webpack_require__.c = installedModules;
    __webpack_require__.d = function (exports, name, getter) {
        if (!__webpack_require__.o(exports, name)) {
            Object.defineProperty(exports, name, { enumerable: true, get: getter });
        }
    };
    __webpack_require__.r = function (exports) {
        if (typeof Symbol !== 'undefined' && Symbol.toStringTag) {
            Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
        }
        Object.defineProperty(exports, '__esModule', { value: true });
    };
    __webpack_require__.t = function (value, mode) {
        if (mode & 1) value = __webpack_require__(value);
        if (mode & 8) return value;
        if ((mode & 4) && typeof value === 'object' && value && value.__esModule) return value;
        var ns = Object.create(null);
        __webpack_require__.r(ns);
        Object.defineProperty(ns, 'default', { enumerable: true, value: value });
        if (mode & 2 && typeof value != 'string') for (var key in value) __webpack_require__.d(ns, key, function (key) { return value[key]; }.bind(null, key));
        return ns;
    };
    __webpack_require__.n = function (module) {
        var getter = module && module.__esModule ?
            function getDefault() { return module['default']; } :
            function getModuleExports() { return module; };
        __webpack_require__.d(getter, 'a', getter);
        return getter;
    };
    __webpack_require__.o = function (object, property) { return Object.prototype.hasOwnProperty.call(object, property); };
    __webpack_require__.p = "";
    __webpack_require__.oe = function (err) { console.error(err); throw err; };
    var jsonpArray = window["webpackJsonp"] = window["webpackJsonp"] || [];
    var oldJsonpFunction = jsonpArray.push.bind(jsonpArray);
    jsonpArray.push = webpackJsonpCallback;
    jsonpArray = jsonpArray.slice();
    for (var i = 0; i < jsonpArray.length; i++) webpackJsonpCallback(jsonpArray[i]);
    var parentJsonpFunction = oldJsonpFunction;
    return __webpack_require__(__webpack_require__.s = "./src/main2.js");
})
    ({
        "./src/main2.js":
            (function (module, exports, __webpack_require__) {
                document.getElementById('main2-loadC1').addEventListener('click', () => {
                    __webpack_require__.e("c1").then(__webpack_require__.bind(null, "./src/c1.js")).then(c1 => {
                        console.log(c1.default)
                    })
                });
                document.getElementById('main2-loadC2').addEventListener('click', () => {
                    __webpack_require__.e("c2").then(__webpack_require__.bind(null, "./src/c2.js")).then(c1 => {
                        console.log(c1.default)
                    })
                });
            })
    });
```
### 6.9 dist\c1.js
```js
(window["webpackJsonp"] = window["webpackJsonp"] || []).push([["c1"], {
  "./src/c1.js":
    (function (module, __webpack_exports__, __webpack_require__) {
      "use strict";
      __webpack_require__.r(__webpack_exports__);
      __webpack_exports__["default"] = ({
        name: 'c1'
      });
    })
}]);
```
### 6.10 dist\c2.js
```js
(window["webpackJsonp"] = window["webpackJsonp"] || []).push([["c2"], {
    "./src/c2.js":
        (function (module, __webpack_exports__, __webpack_require__) {
            "use strict";
            __webpack_require__.r(__webpack_exports__);
            __webpack_exports__["default"] = ({
                name: 'c2'
            });
        })
}]);
```
### 6.11 实现
```js
(function (modules) {
    function webpackJsonpCallback(data) {
        var chunkIds = data[0];
        var moreModules = data[1];
        var moduleId, chunkId, i = 0, resolves = [];
        for (; i < chunkIds.length; i++) {
            chunkId = chunkIds[i];
            if (installedChunks[chunkId]) {
                resolves.push(installedChunks[chunkId][0]);
            }
            installedChunks[chunkId] = 0;
        }
        for (moduleId in moreModules) {
            modules[moduleId] = moreModules[moduleId];
        }
+       if (parentJsonpFunction) parentJsonpFunction(data);
        while (resolves.length) {
            resolves.shift()();
        }
    }
    var installedModules = {};
    var installedChunks = { main: 0 };
    __webpack_require__.p = "";
    function jsonpScriptSrc(chunkId) {
        return __webpack_require__.p + "" + chunkId + ".js";
    }
    function __webpack_require__(moduleId) {
        if (installedModules[moduleId]) {
            return installedModules[moduleId].exports;
        }
        var module = (installedModules[moduleId] = { i: moduleId, l: false, exports: {} });
        modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
        module.l = true;
        return module.exports;
    }
    __webpack_require__.r = function (exports) {
        if (typeof Symbol !== 'undefined' && Symbol.toStringTag) {
            Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
        }
        Object.defineProperty(exports, '__esModule', { value: true });
    };
    __webpack_require__.t = function (value, mode) {
        value = __webpack_require__(value);
        var ns = Object.create(null);
        Object.defineProperty(ns, "__esModule", { value: true });
        Object.defineProperty(ns, "default", { enumerable: true, value: value });
        return ns;
    };

    __webpack_require__.e = function requireEnsure(chunkId) {
        var promises = [];
        var installedChunkData = installedChunks[chunkId];
        var promise = new Promise(function (resolve, reject) {
            installedChunkData = installedChunks[chunkId] = [resolve, reject];
        });
        promises.push((installedChunkData[2] = promise));
        var script = document.createElement("script");
        script.src = jsonpScriptSrc(chunkId);
        document.head.appendChild(script);
        return Promise.all(promises);
    }
    var jsonpArray = (window["webpackJsonp"] = window["webpackJsonp"] || []);
    var oldJsonpFunction = jsonpArray.push.bind(jsonpArray);
    jsonpArray.push = webpackJsonpCallback;
+    jsonpArray = jsonpArray.slice();
+    for (var i = 0; i < jsonpArray.length; i++) webpackJsonpCallback(jsonpArray[i]);
+    var parentJsonpFunction = oldJsonpFunction;
    return __webpack_require__(__webpack_require__.s = "./src/main1.js");
})
    ({
        "./src/main1.js":
            (function (module, exports, __webpack_require__) {
                document.getElementById('main1-loadC1').addEventListener('click', () => {
                    __webpack_require__.e("c1").then(__webpack_require__.bind(null, "./src/c1.js")).then(c1 => {
                        console.log(c1.default)
                    })
                });
                document.getElementById('main1-loadC2').addEventListener('click', () => {
                    __webpack_require__.e("c2").then(__webpack_require__.bind(null, "./src/c2.js")).then(c1 => {
                        console.log(c1.default)
                    })
                });
                document.getElementById('loadMain2').addEventListener('click', () => {
                    let script = document.createElement('script');
                    script.src = 'main2.js';
                    document.body.appendChild(script);
                });
            })
    });
```