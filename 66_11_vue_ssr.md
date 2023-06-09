## 什么是服务端渲染(Server Side Render) 

放在服务器进行就是服务器渲染，放在浏览器进行就是浏览器渲染

- 客户端渲染不利于 SEO 搜索引擎优化
- 服务端渲染是可以被爬虫抓取到的，客户端异步渲染是很难被爬虫抓取到的
- SSR直接将HTML字符串传递给浏览器。大大加快了首屏加载时间。
- SSR占用更多的CPU和内存资源
- 一些常用的浏览器API可能无法正常使用
- 在vue中只支持beforeCreate和created两个生命周期

## 开始vue-ssr之旅
```sh
yarn add vue-server-renderer vue
yarn add express
```
createRenderer,创建一个渲染函数 renderToString, 渲染出一个字符串
```js
let Vue = require('vue');
let render = require('vue-server-renderer');
let vm = new Vue({ 
    data:{
        msg:'jw',
    },
    template:('<h1>{{msg}}</h1>')
})
let express = require('express');
let app = express();
app.get('/',async (req,res)=>{
    let code = await render.createRenderer().renderToString(vm);
    res.send(`
        <!DOCTYPE html>
        <html lang="en">
        <head>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <meta http-equiv="X-UA-Compatible" content="ie=edge">
            <title>Document</title>
        </head>
        <body>
            ${code}
        </body>
        </html>
    `)
});
app.listen(3000);
```
## 采用模板渲染
```html
<!DOCTYPE html>
<html lang="en">
  <head><title>Hello</title></head>
  <body>
    <!--vue-ssr-outlet-->
  </body>
</html>
```
传入template 替换掉注释标签
```js
let Vue = require('vue');
let fs = require('fs');
let template = fs.readFileSync('template.html','utf8')
let render = require('vue-server-renderer');
let vm = new Vue({ 
    data:{
        msg:'jw',
    },
    template:('<h1>{{msg}}</h1>')
})
let express = require('express');
let app = express();
app.get('/',async (req,res)=>{
    let code = await render.createRenderer({
        template
    }).renderToString(vm);
    res.send(code);
});
app.listen(3000);
```
## 目录创建
```js
├── config
│   ├── webpack.base.js
│   ├── webpack.client.js
│   └── webpack.server.js
├── dist
│   ├── client.bundle.js
│   ├── index.html
│   ├── index.ssr.html
│   ├── server.bundle.js
│   ├── vue-ssr-client-manifest.json
│   └── vue-ssr-server-bundle.json
├── package.json
├── public
│   ├── index.html
│   └── index.ssr.html
├── server.js
├── src
│   ├── App.vue
│   ├── components
│   │   ├── Bar.vue
│   │   └── Foo.vue
│   ├── entry-client.js
│   ├── entry-server.js
│   ├── main.js
│   ├── router.js
│   └── store.js
├── webpack.config.js
```
## 通过webpack实现编译vue项目
安装插件
```sh
yarn add webpack webpack-cli webpack-dev-server vue-loader vue-style-loader css-loader html-webpack-plugin @babel/core @babel/preset-env babel-loader vue-template-compiler webpack-merge
```
```js
let path = require('path');
let HtmlWebpackPlugin = require('html-webpack-plugin');
let VueLoaderPlugin = require('vue-loader/lib/plugin')
module.exports = {
    entry:'./src/main.js',
    output:{
        filename:'bundle.js',
        path:path.resolve(__dirname)
    },
    module:{
        rules:[
            {test:/\.css/,use:['vue-style-loader','css-loader']},
            {
                test:/\.js/,
                use:{
                    loader:'babel-loader',
                    options:{
                        presets:['@babel/preset-env']
                     },
                },
                exclude:/node_modules/,
            },
            {test:/\.vue/,use:'vue-loader'}
        ]
    },
    plugins:[
        new VueLoaderPlugin(),
        new HtmlWebpackPlugin({
            template:'./src/index.html'
        })
    ]
}
```
## 配置客户端打包和服务端打包
- webpack.base.js
```js
let path = require('path');
let VueLoaderPlugin = require('vue-loader/lib/plugin')
module.exports = {
  entry:'./src/main.js',
  output:{
      filename:'[name].bundle.js',
      path:path.resolve(__dirname)
  },
  module:{
      rules:[
          {test:/\.css/,use:['vue-style-loader','css-loader']},
          {
              test:/\.js/,
              use:{
                  loader:'babel-loader',
                  options:{
                      presets:['@babel/preset-env']
                   },
              },
              exclude:/node_modules/,
          },
          {test:/\.vue/,use:'vue-loader'}
      ]
  },
  plugins:[
      new VueLoaderPlugin()
  ]
}
```
- webpack.client.js
```js
let path = require('path');
let HtmlWebpackPlugin = require('html-webpack-plugin');
let merge  = require('webpack-merge');
let base = require('./webpack.base');
module.exports = merge(base, {
    entry:{
        'client':path.resolve(__dirname,'../src/client.js')
    },
    plugins:[
        new HtmlWebpackPlugin({
            template:'./src/index.html'
        })
    ],
});
```
- webpack.server.js
```js
let path = require('path');
let HtmlWebpackPlugin = require('html-webpack-plugin');
let merge  = require('webpack-merge');
let base = require('./webpack.base');

module.exports =merge(base,{
    target:'node', // 打包类型node
    entry:{
        server:path.resolve(__dirname,'../src/server.js')
    },
    output:{
        libraryTarget: 'commonjs2' // 以commonjs规范导出
    },  
    plugins:[
        new HtmlWebpackPlugin({
            template:'./src/index.html',
            excludeChunks:['server']
        })
    ]
})
```
## 服务端配置
在App.vue上增加id="app"可以保证元素被正常激活
```js
let express = require('express');
let vueServerRenderer = require('vue-server-renderer');
let path = require('path')
let app = express();
let fs = require('fs');

let render = vueServerRenderer.createBundleRenderer(fs.readFileSync('./dist/server.bundle.js','utf8'),{
    template:fs.readFileSync('./dist/index.ssr.html','utf8')
});

app.get('/',(req,res)=>{
    render.renderToString((err,html)=>{
        res.send(html);
    })
});

app.use(express.static(path.join(__dirname,'dist')))
app.listen(3000,()=>{
    console.log('server start 3000')
});
```
## 集成路由
```js
import Vue from 'vue';
import VueRouter from 'vue-router';
Vue.use(VueRouter);
import Bar from './components/Bar.vue';
import Foo from './components/Foo.vue';
export default ()=>{
    let router = new VueRouter({
        mode:'history',
        routes:[
            {
                path:'/', component:Bar,
            },
            {
                path:'/foo',component:Foo
            }
        ]
    });
    return router
}
```
## 配置入口文件
```js
import createRouter from './router';
export default ()=>{
    let router = createRouter(); // 增加路由
    let app = new Vue({
        router,
        render:(h)=>h(App);
    })
    return {app,router}
}
```
## 配置组件信息
```js
<template>
    <div id="app">
        <router-link to="/"> bar</router-link>
        <router-link to="/foo"> foo</router-link>
        <router-view></router-view>
        {{$store.state.username}}
    </div>
</template>
```
## 防止刷新页面不存在
```js
app.get('*',(req,res)=>{
    render.renderToString({url:req.url},(err,html)=>{
        res.send(html);
    })
});
```
## vuex配置
```js
import Vue from 'vue';
import Vuex from 'vuex';
Vue.use(Vuex);

export default ()=>{
    let store = new Vuex.Store({
        state:{
            username:'zf'
        },
        mutations:{
            set_user(state){
                state.username = 'hello;
            }
        },
        actions:{
            set_user({commit}){
                return new Promise((resolve,reject)=>{
                    setTimeout(() => {
                        commit('set_user');
                        resolve();
                    }, 1000);
                })
            }
        }
    });
    return store
}

// 引用vuex
import createRouter from './router';
import createStore from './store'
export default ()=>{
    let router = createRouter();
    let store = createStore();
    let app = new Vue({
        router,
        store,
        render:(h)=>h(App)
    })
    return {app,router,store}
}

```
### 在后端更新vuex
```js
import createApp from './main';
export default (context)=>{
    return new Promise((resolve)=>{
        let {app,router,store} = createApp();
        router.push(context.url); // 默认访问到/a就跳转到/a
        router.onReady(()=>{
            let matchs = router.getMatchedComponents(); // 获取路由匹配到的组件

            Promise.all(matchs.map(component=>{
                if(component.asyncData){
                    return component.asyncData(store);
                }
            })).then(()=>{
                context.state = store.state; // 将store挂载在window.__INITIAL_STATE__
                resolve(app);

            });
        })
    })
}
```
### 在浏览器运行时替换store
```js
// 在浏览器运行代码
if(typeof window !== 'undefined' && window.__INITIAL_STATE__){
    store.replaceState(window.__INITIAL_STATE__);
}
```
## 通过json配置createBundleRenderer方法
实现热更新,自动增加preload和prefetch,以及可以使用sourceMap
```js
const VueSSRClientPlugin = require('vue-server-renderer/client-plugin');
const VueSSRServerPlugin = require('vue-server-renderer/server-plugin');
const nodeExternals = require('webpack-node-externals');
let template = fs.readFileSync('./dist/index.ssr.html','utf8');
let manifest = require('./dist/vue-ssr-client-manifest.json');
let bundle = require('./dist/vue-ssr-server-bundle.json');

let render = vueServerRenderer.createBundleRenderer(bundle,{
    template,
    clientManifest:manifest
})
```
## 管理vue-meta标签
自己更方便的管理vue的meta标签
```js
import Vue from 'vue'
import Meta from 'vue-meta';

Vue.use(Meta);

// 将meta挂载在上下文中
const meta = app.$meta()
context.meta = meta;

// 组件中配置
metaInfo: {
    title:'嘿嘿'
}

// 在模板中取值
{{{ meta.inject().title.text() }}} 
```