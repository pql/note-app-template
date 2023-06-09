## 1.什么是服务器端渲染
### 1.1 服务器端渲染
页面上的内容是由服务器生产的
```sh
cnpm i express -S
```
```js
let express = require('express');
let app = express();
app.get('/', (req, res) => {
    res.send(`
        <html>
            <body>
                <div id="root">hello</div>
            </body>
        </html>
    `);
});
app.listen(8080);
```
### 1.2 客户端渲染
页面上的内容由于浏览器运行JS脚本而渲染到页面上的
- 浏览器访问服务器
- 服务器返回一个空的HTML页面，里面有一个JS资源链接，比如bundle.js
- 浏览器下载JS代码并在浏览器中运行
- 内容呈现在页面上
```sh
cnpm i react react-dom -S
cnpm i webpack webpack-cli -D
cnpm i babel-loader @babel/core -D
cnpm i @babel/preset-env @babel/preset-react -D
cnpm i webpack-node-externals -D
```
```js
const express = require('express');
const app = express();
app.get('/', (req, res) => {
    res.send(`
        <html>
            <body>
                <div id="root"></div>
                <script>
                    document.getElementById('root').innerHTML = 'hello';
                </script>
            </body>
        </html>
    `);
});
app.listen(9090);
```
## 2.服务端打包React组件
### 2.1 服务端渲染React组件流程
- i.浏览器发送请求
- ii.服务器运行React代码生成页面
- iii.服务器返回页面
### 2.2 webpack.config.js
server/webpack.config.js
```js
const path = require('path');
const nodeExternals = require('webpack-node-externals');
module.exports = {
    target: 'node', // 打包的是服务端node文件
    mode: 'development', // 开发模式
    output: {
        path: path.resolve(__dirname, 'build'),
        filename: 'bundle.js'
    },
    externals: [nodeExternals()],
    module: {
        rules: [
            {
                test: /\.js$/,
                loader: 'babel-loader',
                exclude: /node_modules/,
                options: {
                    presets: [
                        [
                            "@babel/preset-env", {
                                targets: {
                                    browsers: ['last 2 versions']
                                }
                            }
                        ], "@babel/preset-react"
                    ]
                }
            }
        ]
    }
}
```
### 2.3 server/src/index.js
server/src/index.js
```js
import Home from './containers/Home';
const express = require('express');
const app = express();
app.get('/', (req, res) => {
    res.send(`
        <html>
            <body>
                <div id="root"></div>
                <script>
                    document.getElementById('root').innerHTML = 'hello';
                </script>
            </body>
        </html>
    `);
});
app.listen(9090);
```
### 2.4 Home/index.js
server/src/containers/Home/index.js
```js
import React, { Component } from 'react';
export default class Home extends Component {
    render() {
        return (<div>Home</div>)
    }
}
```
## 3.服务器端渲染
### 3.1 server/webpack.config.js
server/webpack.config.js
```js
// 浏览器 服务端
const path = require('path');
const nodeExternals = require('webpack-node-externals');
module.exports = {
    target: 'node', // 打包的是服务端node文件
    mode: 'development', // 开发模式
    entry: path.resolve(__dirname, './src/index.js'),
    output: {
        path: path.resolve(__dirname, 'build'),
        filename: 'bundle.js'
    },
    externals: [nodeExternals()],
    module: {
        rules: [
            {
                test: /\.js$/,
                loader: 'babel-loader',
                exclude: /node_modules/,
                options: {
                    presets: [
                        "@babel/preset-env",
                        "@babel/preset-react"
                    ]
                }
            }
        ]
    }
}
```
### 3.2 server/src/index.js
server/src/index.js
```js
import React from 'react';
import Home from './containers/Home';
import {renderToString} from 'react-dom/server';
import express from 'express';
const app = express();
const html = renderToString(<Home />);
app.get('/', (req, res) => {
    res.send(`
        <html>
            <body>
                <div id="root">${html}</div>
            </body>
        </html>
    `);
});
app.listen(9090);
```
### 3.3 package.json
package.json
```json
"scripts": {
    "start": "node ./server/build/bundle.js",
    "build": "webpack --config server/webpack.config.js"
}
```
## 4.优化启动流程
### 4.1 安装nodemon
```sh
cnpm i nodemon -g
```
### 4.2 package.json
```json
"scripts": {
    "dev":"npm-run-all --parallel dev:**",
    "dev:start":"nodemon './server/build/bundle.js'",
    "dev:build":"webpack --config server/webpack.config.js --watch"
}
```
### 4.3 npm-run-all
```sh
cnpm npm-run-all -g
```
## 5.计数器组件
- 服务器端运行React代码渲染出HTML
- 服务器把渲染出的HTML页面发送给了浏览器
- 浏览器接受到HTML会渲染到页面上
- 浏览器发现页面引用的index.js文件会去下载
- 浏览器下载得到的index.js文件并在浏览器端执行
- 浏览器中的代码接管了页面的所有内容，后面和客户端渲染是一样的
### 5.1 src/client/index.js
src/client/index.js
```js
import React from 'react';
import ReactDOM from 'react-dom';
import Counter from '../containers/Counter';

ReactDOM.hydrate(<Counter/>, document.querySelector('#root'));
```
### 5.2 Counter/index.js
src/containers/Counter/index.js
```js
import React, {Component} from 'react';
export default class Counter extends Component {
    state: {number: 0}
    render() {
        return (
            <div>
                <p>{this.state.number}</p>
                <button onClick={() => this.setState({number: this.state.number+1})}>+</button>
            </div>
        )
    }
}
```
### 5.3 webpack.client.js
webpack.client.js
```js
// 浏览器 服务器端
const path = require('path');
module.exports = {
    mode: 'development', // 开发模式
    entry: path.resolve(__dirname, './src/client/index.js'),
    output: {
        path: path.resolve(__dirname, 'public'),
        filename: 'index.js'
    },
    module: {
        rules: [
            {
                test: /\.js$/,
                loader: 'babel-loader',
                exclude: /node_modules/,
                options: {
                    presets: [
                        "@babel/preset-env",
                        "@babel/preset-react"
                    ],
                    plugins: [
                        "@babel/plugin-proposal-class-properties"
                    ]
                }
            }
        ]
    }
}
```
## 6.优化代码结构
### 6.1 webpack.base.js
```js
// 浏览器 服务器端
const path = require('path');
module.exports = {
    mode: 'development',
    module: {
        rules: [
            {
                test: /\.js$/,
                loader: 'babel-loader',
                exclude: /node_modules/,
                options: {
                    presets: [
                        "@babel/preset-env",
                        "@babel/preset-react"
                    ],
                    plugins: [
                        "@babel/plugin-proposal-class-properties"
                    ]
                }
            }
        ]
    }
}
```
### 6.2 webpack.client.js
webpack.client.js
```js
// 浏览器 服务器端
const path = require('path');
const merge = require('webpack-merge');
const base = require('./webpack.base');
module.exports = merge(base, {
    mode: 'development', // 开发模式
    entry: path.resolve(__dirname, './src/client/index.js'),
    output: {
        path: path.resolve(__dirname, 'public'),
        filename: 'index.js'
    }
})
```
### 6.3 webpack.server.js
webpack.server.js
```js
//浏览器 服务器端
let path=require('path');
const merge=require('webpack-merge');
const base=require('./webpack.base');
const nodeExternals=require('webpack-node-externals');
module.exports = merge(base,{
    target: 'node',//打包的是服务器端node文件
    mode: 'development',//开发模式
    entry:path.resolve(__dirname,'./src/server/index.js'),
    output:{
        path:path.resolve(__dirname,'build'),
        filename:'bundle.js'
    },
    externals:[nodeExternals()]
}) 
```
### 6.4 server/index.js
src/server/index.js
```js
import React from 'react';
import {renderToString} from 'react-dom/server';
import Home from '../containers/Home';
import express from 'express';
let app=express();
app.use(express.static('public'));
const content=renderToString(<Home />);
app.get('/',(req,res) => {
    res.send(`
    <!DOCTYPE html>
    <html lang="en">
        <head>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <meta http-equiv="X-UA-Compatible" content="ie=edge">
            <title></title>
        </head>
        <body>
            <div id="root">${content}</div>
            <script src="/index.js"></script>
        </body>
    </html>
    `);
});
app.listen(9090);
```
## 7.使用路由
```sh
cnpm i react-router-dom -S
```
### 7.1 客户端路由
- 客户端请求服务器
- 服务器返回HTML给浏览器，浏览器渲染显示页面
- 浏览器发现需要外链JS资源，加载JS资源
- 加载好的JS资源在浏览器端执行
- JS中的React代码开始实现路由功能
- 路由代码首先获取地址栏中的地址，然后根据不同的地址根据路由配置渲染对应内容
### 7.2 src/client/index.js
src/client/index.js
```js
import React from 'react';
import ReactDOM from 'react-dom';
import {BrowserRouter} from 'react-router-dom';
import routes from '../routes';
ReactDOM.hydrate(
    <BrowserRouter>
        {routes}
    </BrowserRouter>,
    document.querySelector('#root')
);
```
### 7.3 src/server/index.js
src/server/index.js
```js
import React from 'react';
import {renderToString} from 'react-dom/server';
import Home from '../containers/Home';
import express from 'express';
import {StaticRouter} from 'react-router-dom';
import routes from '../routes';
let app=express();
app.use(express.static('public'));
//context数据的传递 StaticRouter需要知道当前路径

app.get('*',(req,res) => {
    const content=renderToString(
        <StaticRouter context={{}} location={req.path}>
            {routes}
        </StaticRouter>
    );
    res.send(`
    <!DOCTYPE html>
    <html lang="en">
        <head>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <meta http-equiv="X-UA-Compatible" content="ie=edge">
            <title></title>
        </head>
        <body>
            <div id="root">${content}</div>
            <script src="/index.js"></script>
        </body>
    </html>
    `);
});
app.listen(9090);
```
### 7.4 src/routes.js
src/routes.js
```js
import React,{Fragment} from 'react';
import {Route} from 'react-router-dom';
import Home from './containers/Home';
import Counter from './containers/Counter';
export default (
    <Fragment>
        <Route path="/" exact component={Home}></Route>
        <Route path="/counter" exact component={Counter}></Route>
    </Fragment>
)
```
## 8.跳转路由
### 8.1 src/client/index.js
src/client/index.js
```js
import React,{Fragment} from 'react';
import ReactDOM from 'react-dom';
import {BrowserRouter} from 'react-router-dom';
import Header from '../components/Header';
import routes from '../routes';
ReactDOM.hydrate(
    <BrowserRouter>
        <Fragment>
            <Header/>
            <div className="container" style={{marginTop:50}}>
                {routes}
            </div>
        </Fragment>
    </BrowserRouter>
    ,document.querySelector('#root')
);
```
### 8.2 Counter/index.js
src/containers/Counter/index.js
```js
import React,{Component} from 'react';
export default class Counter extends Component{
    state={number:0}
    render() {
        return (
            <div>
                <p>{this.state.number}</p>
                <button className="btn btn-primary" onClick={()=>this.setState({number:this.state.number+1})}>+</button>
            </div>
        )
    }
}
```
### 8.3 Home/index.js
src/containers/Home/index.js
```js
import React,{Component} from 'react';
export default class Home extends Component{
    render() {
        return <div>首页</div>
    }
}
```
### 8.4 src/server/index.js
src/server/index.js
```js
import express from 'express';
import render from './render';
let app=express();
app.use(express.static('public'));
//context数据的传递 StaticRouter需要知道当前路径
app.get('*',(req,res) => {
    render(req,res);
});
app.listen(9090);
```
### 8.5 Header/index.js
src/components/Header/index.js
```js
import React,{Component} from 'react';
import {Link} from 'react-router-dom';
export default class Home extends Component{
    render() {
        return (
            <nav className="navbar navbar-inverse navbar-fixed-top">
                <div className="container">
                    <div className="navbar-header">
                        <a className="navbar-brand" href="#">珠峰SSR</a>
                    </div>
                    <div id="navbar" className="collapse navbar-collapse">
                        <ul className="nav navbar-nav">
                            <li><Link to="/">Home</Link></li>
                            <li><Link to="/counter">Counter</Link></li>
                        </ul>
                    </div>
                </div>
            </nav>
        )
    }
}
```
### 8.6 src/server/render.js
src/server/render.js
```js
import React,{Component,Fragment} from 'react';
import {StaticRouter} from 'react-router-dom';
import Header from '../components/Header';
import routes from '../routes';
import {renderToString} from 'react-dom/server';
export default function (req,res) {
    const content=renderToString(
        <StaticRouter context={{}} location={req.path}>
                <Fragment>
                    <Header/>
                    <div className="container" style={{marginTop:50}}>
                        {routes}
                    </div>
                </Fragment>
            </StaticRouter>
    );
    res.send(`
    <!DOCTYPE html>
    <html lang="en">
        <head>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <meta http-equiv="X-UA-Compatible" content="ie=edge">
            <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css" >
            <title>珠峰SSR</title>
        </head>
        <body>
            <div id="root">${content}</div>
            <script src="/index.js"></script>
        </body>
    </html>
    `);
}
```
## 9. redux计数器
### 9.1 src/client/index.js
src/client/index.js
```js
import React,{Fragment} from 'react';
import ReactDOM from 'react-dom';
import {BrowserRouter} from 'react-router-dom';
import Header from '../components/Header';
import routes from '../routes';
import {Provider} from 'react-redux';
import getStore from '../store';
ReactDOM.hydrate(
    <Provider store={getStore()}>
        <BrowserRouter>
            <Fragment>
                <Header/>
                <div className="container" style={{marginTop:50}}>
                    {routes}
                </div>
            </Fragment>
        </BrowserRouter>
    </Provider>
    ,document.querySelector('#root')
);
```
### 9.2 Counter/index.js
src/containers/Counter/index.js
```js
import React,{Component} from 'react';
import {connect} from 'react-redux';
import actions from '../../store/actions';
class Counter extends Component{
    render() {
        return (
            <div>
                <p>{this.props.number}</p>
                <button className="btn btn-primary" onClick={this.props.increment}>+</button>
            </div>
        )
    }
}
export default connect(
    state => state,
    actions
)(Counter);
```
### 9.3 server/render.js
src/server/render.js
```js
import React,{Component,Fragment} from 'react';
import {StaticRouter} from 'react-router-dom';
import Header from '../components/Header';
import routes from '../routes';
import {renderToString} from 'react-dom/server';
import getStore from '../store';
import {Provider} from 'react-redux';
export default function (req,res) {
    const content=renderToString(
        <Provider store={getStore()}>
            <StaticRouter context={{}} location={req.path}>
                    <Fragment>
                        <Header/>
                        <div className="container" style={{marginTop:50}}>
                            {routes}
                        </div>
                    </Fragment>
                </StaticRouter>
        </Provider>
    );
    res.send(`
    <!DOCTYPE html>
    <html lang="en">
        <head>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <meta http-equiv="X-UA-Compatible" content="ie=edge">
            <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css" >
            <title>珠峰SSR</title>
        </head>
        <body>
            <div id="root">${content}</div>
            <script src="/index.js"></script>
        </body>
    </html>
    `);
}
```
### 9.4 src/store/index.js
src/store/index.js
```js
import reducer from './reducer';
import {createStore,applyMiddleware} from 'redux';
import thunk from 'redux-thunk';
import logger from 'redux-logger';
function getStore() {
    return createStore(reducer,applyMiddleware(thunk,logger));
}
export default getStore;
```
### 9.5 store/reducer.js
src/store/reducer.js
```js
import * as types from './action-types';
let initState = {
    number: 0
};
export default function (state = initState, action) {
    switch (action.type) {
        case types.INCREMENT:
            return {
                number: state.number + 1
            };
        default:
            return state;
    }
}
```
### 9.6 action-types.js
src/store/action-types.js
```js
export const INCREMENT='INCREMENT';
```
### 9.7 store/actions.js
src/store/actions.js
```js
import * as types from './action-types';
export default {
    increment() {
        return {type:types.INCREMENT}
    }
}
```
## 10.合并reducers
### 10.1 Counter/index.js
src/containers/Counter/index.js
```js
import React,{Component} from 'react';
import {connect} from 'react-redux';
import actions from '../../store/actions';
class Counter extends Component{
    render() {
        return (
            <div>
                <p>{this.props.number}</p>
                <button className="btn btn-primary" onClick={this.props.increment}>+</button>
            </div>
        )
    }
}
export default connect(
    state => state.counter,
    actions
)(Counter);
```
### 10.2 src/store/index.js
src/store/index.js
```js
import reducers from './reducers';
import {createStore,applyMiddleware} from 'redux';
import thunk from 'redux-thunk';
import logger from 'redux-logger';
function getStore() {
    return createStore(reducers,applyMiddleware(thunk,logger));
}
export default getStore;
```
### 10.3 reducers/index.js
src/store/reducers/index.js
```js
import {combineReducers} from 'redux';
import home from './home';
import counter from './counter';
let reducers=combineReducers({
    home,
    counter
});
export default reducers;
```
### 10.4 reducers/home.js
src/store/reducers/home.js
```js
import * as types from '../action-types';
let initState = {
    list:[]
};
export default function (state = initState, action) {
    switch (action.type) {
        default:
          return state;
    }
}
```
### 10.5 reducers/counter.js
src/store/reducers/counter.js
```js
import * as types from '../action-types';
let initState = {
    number: 0
};
export default function (state = initState, action) {
    switch (action.type) {
        case types.INCREMENT:
            return {
                number: state.number + 1
            };
        default:
            return state;
    }
}
```
## 11.客户端加载数据
### 11.1 src/client/index.js
src/client/index.js
```js
import React,{Fragment} from 'react';
import ReactDOM from 'react-dom';
import {BrowserRouter} from 'react-router-dom';
import Header from '../components/Header';
import routes from '../routes';
import {Provider} from 'react-redux';
import getStore from '../store';
ReactDOM.hydrate(
    <Provider store={getStore()}>
        <BrowserRouter>
            <Fragment>
                <Header/>
                <div className="container" style={{marginTop:70}}>
                    {routes}
                </div>
            </Fragment>
        </BrowserRouter>
    </Provider>
    ,document.querySelector('#root'));
```
### 11.2 Counter/index.js
src/containers/Counter/index.js
```js
import React,{Component} from 'react';
import {connect} from 'react-redux';
import actions from '../../store/actions/counter';
class Counter extends Component{
    render() {
        return (
            <div>
                <p>{this.props.number}</p>
                <button className="btn btn-primary" onClick={this.props.increment}>+</button>
            </div>
        )
    }
}
export default connect(
    state => state.counter,
    actions
)(Counter);
```
### 11.3 Home/index.js
src/containers/Home/index.js
```js
import React,{Component} from 'react';
import {connect} from 'react-redux';
import actions from '../../store/actions/home';
class Home extends Component{
    componentDidMount() {
        this.props.getHomeList();
    }
    render() {
        return (
            <div className="row">
                <div className="col-md-12">
                    <ul className="list-group">
                        {
                            this.props.list.map(item => (
                                <li className="list-group-item" key={item.id}>{item.name}</li>
                            ))
                        }
                    </ul>
                </div>
            </div>
        )
    }
}
export default connect(
    state => state.home,
    actions
)(Home);
```
### 11.4 src/server/render.js
src/server/render.js
```js
import React,{Component,Fragment} from 'react';
import {StaticRouter} from 'react-router-dom';
import Header from '../components/Header';
import routes from '../routes';
import {renderToString} from 'react-dom/server';
import getStore from '../store';
import {Provider} from 'react-redux';
export default function (req,res) {
    const content=renderToString(
        <Provider store={getStore()}>
            <StaticRouter context={{}} location={req.path}>
                    <Fragment>
                        <Header/>
                        <div className="container" style={{marginTop:70}}>
                            {routes}
                        </div>
                    </Fragment>
                </StaticRouter>
        </Provider>
    );
    res.send(`
    <!DOCTYPE html>
    <html lang="en">
        <head>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <meta http-equiv="X-UA-Compatible" content="ie=edge">
            <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css" >
            <title>珠峰SSR</title>
        </head>
        <body>
            <div id="root">${content}</div>
            <script src="/index.js"></script>
        </body>
    </html>
    `);
}
```
### 11.5 src/store/action-types.js
src/store/action-types.js
```js
//counter
export const INCREMENT='INCREMENT';

//home
export const SET_HOME_LIST='SET_HOME_LIST';
```
### 11.6 reducers/home.js
src/store/reducers/home.js
```js
import * as types from '../action-types';
let initState = {
    list:[]
};
export default function (state = initState, action) {
    switch (action.type) {
        case types.SET_HOME_LIST:
            return {...state,list:action.payload};
        default:
          return state;
    }
}
```
### 11.7 actions/counter.js
src/store/actions/counter.js
```js
import * as types from '../action-types';
export default {
    increment() {
        return {type:types.INCREMENT}
    }
}
```
### 11.8 reducers/home.js
src/store/reducers/home.js
```js
import * as types from '../action-types';
import axios from 'axios';
export default {
    getHomeList() {
        return function (dispatch,getState) {
            axios.get('http://localhost:4000/api/users').then(result => {
                let list=result.data;
                dispatch({
                    type: types.SET_HOME_LIST,
                    payload:list
                });
            });
        }
    }
}
```
### 11.9 api/server.js
api/server.js
```js
let express=require('express');
let cors=require('cors');
let app=express();
var corsOptions = {
    origin: 'http://localhost:9090',
    optionsSuccessStatus: 200 
}
app.use(cors(corsOptions));
let users=[{id:1,name:'zfpx1'},{id:2,name:'zfpx2'}];
app.get('/api/users',function (req,res) {
    res.json(users);
});
app.listen(4000);
```
## 12.服务器端路由
### 12.1 src/client/index.js
```js
import React,{Fragment} from 'react';
import ReactDOM from 'react-dom';
import {BrowserRouter} from 'react-router-dom';
import Header from '../components/Header';
import routes from '../routes';
import {Provider} from 'react-redux';
import {Route} from 'react-router-dom';
import getStore from '../store';
ReactDOM.hydrate(
    <Provider store={getStore()}>
        <BrowserRouter>
            <Fragment>
                <Header/>
                <div className="container" style={{marginTop: 70}}>
                    <Fragment>
                      {routes.map(route => (
                            <Route {...route}/>
                      ))}
                    </Fragment>

                </div>
            </Fragment>
        </BrowserRouter>
    </Provider>
    ,document.querySelector('#root'));
```
### 12.2 Home/index.js
src/containers/Home/index.js
```js
import React,{Component} from 'react';
import {connect} from 'react-redux';
import actions from '../../store/actions/home';
class Home extends Component{
    static loadData=() => {
        console.log('加载数据');
    }
    //componentDidMount在服务器端是不执行的
    componentDidMount() {
        this.props.getHomeList();
    }
    render() {
        return (
            <div className="row">
                <div className="col-md-12">
                    <ul className="list-group">
                        {
                            this.props.list.map(item => (
                                <li className="list-group-item" key={item.id}>{item.name}</li>
                            ))
                        }
                    </ul>
                </div>
            </div>
        )
    }
}
export default connect(
    state => state.home,
    actions
)(Home);
```
### 12.3 src/routes.js
src/routes.js
```js
import React,{Fragment} from 'react';
import {Route} from 'react-router-dom';
import Home from './containers/Home';
import Counter from './containers/Counter';
export default [
    {
        path: '/',
        component: Home,
        exact: true,
        key:'home',
        loadData:Home.loadData
    },
    {
        path: '/counter',
        component: Counter,
        key:'login',
        exact: true
    }
]
/**
export default (
    <Fragment>
        <Route path="/" exact component={Home}></Route>
        <Route path="/counter" exact component={Counter}></Route>
    </Fragment>
)
*/
```
### 12.4 src/server/render.js
src/server/render.js
```js
import React,{Component,Fragment} from 'react';
import {StaticRouter} from 'react-router-dom';
import Header from '../components/Header';
import routes from '../routes';
import {renderToString} from 'react-dom/server';
import {Route,matchPath} from 'react-router-dom';
import getStore from '../store';
import {Provider} from 'react-redux';
export default function (req,res) {
    let store=getStore();
    let matchedRoutes=routes.filter(route => {
        return matchPath(req.path,route);
    });
    console.log(matchedRoutes);
    const content=renderToString(
        <Provider store={store}>
            <StaticRouter context={{}} location={req.path}>
                    <Fragment>
                        <Header/>
                        <div className="container" style={{marginTop:70}}>
                        {routes.map(route => (
                            <Route {...route}/>
                            ))}
                        </div>
                    </Fragment>
                </StaticRouter>
        </Provider>
    );
    res.send(`
    <!DOCTYPE html>
    <html lang="en">
        <head>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <meta http-equiv="X-UA-Compatible" content="ie=edge">
            <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css" >
            <title>珠峰SSR</title>
        </head>
        <body>
            <div id="root">${content}</div>
            <script src="/index.js"></script>
        </body>
    </html>
    `);
}
```
## 13.多级路由
### 13.1 src/client/index.js
src/client/index.js
```js
import React,{Fragment} from 'react';
import ReactDOM from 'react-dom';
import {BrowserRouter} from 'react-router-dom';
import Header from '../components/Header';
import routes from '../routes';
import {Provider} from 'react-redux';
import {Route,matchPath} from 'react-router-dom';
import {matchRoutes,renderRoutes} from 'react-router-config';
import getStore from '../store';
ReactDOM.hydrate(
    <Provider store={getStore()}>
        <BrowserRouter>
            <Fragment>
                <Header/>
                <div className="container" style={{marginTop: 70}}>
                    <Fragment>
                      {renderRoutes(routes)}
                    </Fragment>

                </div>
            </Fragment>
        </BrowserRouter>
    </Provider>
    ,document.querySelector('#root'));
```
### 13.2 Header/index.js
src/components/Header/index.js
```js
import React,{Component} from 'react';
import {Link} from 'react-router-dom';
export default class Home extends Component{
    render() {
        return (
            <nav className="navbar navbar-inverse navbar-fixed-top">
                    <div className="container">
                        <div className="navbar-header">
                            <a className="navbar-brand" href="#">珠峰SSR</a>
                        </div>
                        <div id="navbar" className="collapse navbar-collapse">
                            <ul className="nav navbar-nav">
                              <li><Link to="/">Home</Link></li>
                              <li><Link to="/user/list">用户列表</Link></li>
                              <li><Link to="/counter">Counter</Link></li>
                            </ul>
                        </div>
                    </div>
                </nav>
        )
    }
}
```
### 13.3 src/routes.js
src/routes.js
```js
import React,{Fragment} from 'react';
import Home from './containers/Home';
import User from './containers/User';
import UserList from './containers/User/components/UserList';
import Counter from './containers/Counter';
export default [
    {
        path: '/',
        component: Home,
        exact: true,
        key:'/home',
        loadData:Home.loadData
    },
    {
        path: '/user',
        component: User,
        key: '/user',
        routes: [
            {
                path: '/user/list',
                component: UserList,
                key:'/user/list'
            }
        ]
    },
    {
        path: '/counter',
        component: Counter,
        key:'login',
        exact: true
    }
]
/**
export default (
    <Fragment>
        <Route path="/" exact component={Home}></Route>
        <Route path="/counter" exact component={Counter}></Route>
    </Fragment>
)
*/
```
### 13.4 src/server/render.js
src/server/render.js
```js
import React,{Component,Fragment} from 'react';
import {StaticRouter} from 'react-router-dom';
import Header from '../components/Header';
import routes from '../routes';
import {renderToString} from 'react-dom/server';
import {Route,matchPath} from 'react-router-dom';
import {matchRoutes,renderRoutes} from 'react-router-config';
import getStore from '../store';
import {Provider} from 'react-redux';
export default function (req,res) {
    let store=getStore();
    /**
    let matchedRoutes=routes.filter(route => {
        return matchPath(req.path,route);
    });
    */
    let matchedRoutes= matchRoutes(routes,req.path);
    console.log(matchedRoutes);
    const content=renderToString(
        <Provider store={store}>
            <StaticRouter context={{}} location={req.path}>
                    <Fragment>
                        <Header/>
                        <div className="container" style={{marginTop:70}}>
                          {renderRoutes(routes)}
                        </div>
                    </Fragment>
                </StaticRouter>
        </Provider>
    );
    res.send(`
    <!DOCTYPE html>
    <html lang="en">
        <head>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <meta http-equiv="X-UA-Compatible" content="ie=edge">
            <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css" >
            <title>珠峰SSR</title>
        </head>
        <body>
            <div id="root">${content}</div>
            <script src="/index.js"></script>
        </body>
    </html>
    `);
}
```
### 13.5 src/server/render.js
src/server/render.js
```js
import React,{Component} from 'react';
import {Link} from 'react-router-dom';
import {matchRoutes,renderRoutes} from 'react-router-config';
export default class User extends Component{
    render() {
        console.log(this.props.children);
        return (
            <div className="row">
                <div className="col-md-3">
                    <ul className="list-group">
                        <li className="list-group-item"><Link to="/user/list">用户列表</Link></li>
                        <li className="list-group-item"><Link to="/user/add">添加用户</Link></li>
                    </ul>
                </div>
                <div className="col-md-9">
                    {renderRoutes(this.props.route.routes)}
                </div>
            </div>
        )
    }
}
```
### 13.6 components/UserList.js
src/containers/User/components/UserList.js
```js
import React,{Component} from 'react';
import {connect} from 'react-redux';
import actions from '../../../store/actions/home';

class UserList extends Component{
    static loadData=() => {
        console.log('加载数据');
    }
    //componentDidMount在服务器端是不执行的
    componentDidMount() {
        this.props.getHomeList();
    }
    render() {
        return (
            <ul className="list-group">
                {
                    this.props.list.map(item => (
                        <li className="list-group-item" key={item.id}>{item.name}</li>
                    ))
                }
            </ul>
        )
    }
}
export default connect(
    state => state.home,
    actions
)(UserList);
```
## 14.后台获取数据
### 14.1 src/client/index.js
src/client/index.js
```js
import React,{Fragment} from 'react';
import ReactDOM from 'react-dom';
import {BrowserRouter} from 'react-router-dom';
import Header from '../components/Header';
import routes from '../routes';
import {Provider} from 'react-redux';
import {renderRoutes} from 'react-router-config';
import {getClientStore} from '../store';
ReactDOM.hydrate(
    <Provider store={getClientStore()}>
        <BrowserRouter>
            <Fragment>
                <Header/>
                <div className="container" style={{marginTop: 70}}>
                    <Fragment>
                      {renderRoutes(routes)}
                    </Fragment>

                </div>
            </Fragment>
        </BrowserRouter>
    </Provider>
    ,document.querySelector('#root'));
```
### 14.2 Home/index.js
src/containers/Home/index.js
```js
import React,{Component} from 'react';
import {connect} from 'react-redux';
import actions from '../../store/actions/home';
class Home extends Component{
    static loadData=(store) => {
        //dispatch方法的返回值是action
        //https://github.com/reduxjs/redux/blob/master/src/createStore.js
        return store.dispatch(actions.getHomeList());
    }
    //componentDidMount在服务器端是不执行的
    componentDidMount() {
        if(this.props.list.length==0)
            this.props.getHomeList();
    }
    render() {
        return (
            <div className="row">
                <div className="col-md-12">
                    <ul className="list-group">
                        {
                            this.props.list.map(item => (
                                <li className="list-group-item" key={item.id}>{item.name}</li>
                            ))
                        }
                    </ul>
                </div>
            </div>
        )
    }
}
export default connect(
    state => state.home,
    actions
)(Home);
```
### 14.3 src/server/render.js
src/server/render.js
```js
import React,{Component,Fragment} from 'react';
import {StaticRouter} from 'react-router-dom';
import Header from '../components/Header';
import routes from '../routes';
import {renderToString} from 'react-dom/server';
import {Route,matchPath} from 'react-router-dom';
import {matchRoutes,renderRoutes} from 'react-router-config';
import {getStore} from '../store';
import {Provider} from 'react-redux';
export default function (req,res) {
    let store=getStore();
    /**
    let matchedRoutes=routes.filter(route => {
        return matchPath(req.path,route);
    });
    */
    let matchedRoutes=matchRoutes(routes,req.path);
    let promises=[];
    matchedRoutes.forEach(item => {
        if (item.route.loadData)
            promises.push(item.route.loadData(store));
    });
    Promise.all(promises).then(result => {
        const content=renderToString(
            <Provider store={store}>
                <StaticRouter context={{}} location={req.path}>
                        <Fragment>
                            <Header/>
                            <div className="container" style={{marginTop:70}}>
                              {renderRoutes(routes)}
                            </div>
                        </Fragment>
                    </StaticRouter>
            </Provider>
        );
        res.send(`
        <!DOCTYPE html>
        <html lang="en">
            <head>
                <meta charset="UTF-8">
                <meta name="viewport" content="width=device-width, initial-scale=1.0">
                <meta http-equiv="X-UA-Compatible" content="ie=edge">
                <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css" >
                <title>珠峰SSR</title>
            </head>
            <body>
                <div id="root">${content}</div>
                <script>
                  window.context = {
                      state:${JSON.stringify(store.getState())}
                  }
                </script>
                <script src="/index.js"></script>
            </body>
        </html>
        `);
    });
}
```
### 14.4 actions/home.js
src/store/actions/home.js
```js
import * as types from '../action-types';
import axios from 'axios';
export default {
    getHomeList() {
        //https://github.com/reduxjs/redux-thunk/blob/master/src/index.js
        return function (dispatch,getState) {
            return axios.get('http://localhost:4000/api/users').then(result => {
                let list=result.data;
                dispatch({
                    type: types.SET_HOME_LIST,
                    payload:list
                });
            });
        }
    }
}
```
### 14.5 src/store/index.js
src/store/index.js
```js
import reducers from './reducers';
import {createStore,applyMiddleware} from 'redux';
import thunk from 'redux-thunk';
import logger from 'redux-logger';
export function getStore() {
    return createStore(reducers,applyMiddleware(thunk,logger));
}
export function getClientStore() {
    let initState=window.context.state;
    return createStore(reducers,initState,applyMiddleware(thunk,logger));
}
```
### 15.Node代理服务器
### 15.1 src/server/index.js
src/server/index.js
```js
import express from 'express';
import proxy from 'express-http-proxy';
import render from './render';
let app=express();
app.use(express.static('public'));
app.use('/api',proxy('http://127.0.0.1:4000',{
    //修改请求路径
    proxyReqPathResolver: function (req) {
        return `/api/${req.url}`;
    }
}));
//context数据的传递 StaticRouter需要知道当前路径
app.get('*',(req,res) => {
    render(req,res);
});
app.listen(9090);
```
### 15.2 src/server/render.js
src/server/render.js
```js
import React,{Component,Fragment} from 'react';
import {StaticRouter} from 'react-router-dom';
import Header from '../components/Header';
import routes from '../routes';
import {renderToString} from 'react-dom/server';
import {matchRoutes,renderRoutes} from 'react-router-config';
import {getStore} from '../store';
import {Provider} from 'react-redux';
export default function (req,res) {
    let store=getStore();
    /**
    let matchedRoutes=routes.filter(route => {
        return matchPath(req.path,route);
    });
    */
    let matchedRoutes=matchRoutes(routes,req.path);
    let promises=[];
    matchedRoutes.forEach(item => {
        if (item.route.loadData)
            promises.push(item.route.loadData(store));
    });
    Promise.all(promises).then(result => {
        const content=renderToString(
            <Provider store={store}>
                <StaticRouter context={{}} location={req.path}>
                        <Fragment>
                            <Header/>
                            <div className="container" style={{marginTop:70}}>
                              {renderRoutes(routes)}
                            </div>
                        </Fragment>
                    </StaticRouter>
            </Provider>
        );
        res.send(`
        <!DOCTYPE html>
        <html lang="en">
            <head>
                <meta charset="UTF-8">
                <meta name="viewport" content="width=device-width, initial-scale=1.0">
                <meta http-equiv="X-UA-Compatible" content="ie=edge">
                <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css" >
                <title>珠峰SSR</title>
            </head>
            <body>
                <div id="root">${content}</div>
                <script>
                  window.context = {
                      state:${JSON.stringify(store.getState())}
                  }
                </script>
                <script src="/index.js"></script>
            </body>
        </html>
        `);
    });
}
```
### 15.3 actions/home.js
src/store/actions/home.js
```js
import * as types from '../action-types';
import axios from 'axios';
export default {
    getHomeList() {
        //https://github.com/reduxjs/redux-thunk/blob/master/src/index.js
        return function (dispatch,getState,request) {
            //http://localhost:4000/api/users
            return request.get('/api/users').then(result => {
                let list=result.data;
                dispatch({
                    type: types.SET_HOME_LIST,
                    payload:list
                });
            });
        }
    }
}
```
### 15.4 src/store/index.js
src/store/index.js
```js
import reducers from './reducers';
import {createStore,applyMiddleware} from 'redux';
import clientRequest from '../client/request';
import serverRequest from '../server/request';
import thunk from 'redux-thunk';
import logger from 'redux-logger';
export function getStore() {
    return createStore(reducers,applyMiddleware(thunk.withExtraArgument(serverRequest),logger));
}
export function getClientStore() {
    let initState=window.context.state;
    return createStore(reducers,initState,applyMiddleware(thunk.withExtraArgument(clientRequest),logger));
}
```
### 15.5 src/client/request.js
src/client/request.js
```js
import axios from 'axios';
export default axios.create({
    baseURL:'/'
});
```
### 15.6 src/server/request.js
src/server/request.js
```js
import axios from 'axios';
export default axios.create({
    baseURL:'http://localhost:4000/'
});
```
## 16.抽取App.js
### 16.1 src/client/index.js
src/client/index.js
```js
import React,{Fragment} from 'react';
import ReactDOM from 'react-dom';
import {BrowserRouter} from 'react-router-dom';
import {Provider} from 'react-redux';
import routes from '../routes';
import {renderRoutes} from 'react-router-config';
import {getClientStore} from '../store';
ReactDOM.hydrate(
    <Provider store={getClientStore()}>
        <BrowserRouter>
          {renderRoutes(routes)}
      </BrowserRouter>
  </Provider>
    ,document.querySelector('#root')
);
```
### 16.2 User/index.js
src/containers/User/index.js
```js
import React,{Component} from 'react';
import {Link} from 'react-router-dom';
import {matchRoutes,renderRoutes} from 'react-router-config';
export default class User extends Component{
    render() {
        return (
            <div className="row">
                <div className="col-md-3">
                    <ul className="list-group">
                        <li className="list-group-item"><Link to="/user/list">用户列表</Link></li>
                        <li className="list-group-item"><Link to="/user/add">添加用户</Link></li>
                    </ul>
                </div>
                <div className="col-md-9">
                    {renderRoutes(this.props.route.routes)}
                </div>
            </div>
        )
    }
}
```
### 16.3 src/routes.js
src/routes.js
```js
import React,{Fragment} from 'react';
import Home from './containers/Home';
import User from './containers/User';
import UserList from './containers/User/components/UserList';
import Counter from './containers/Counter';
import App from './containers/App';
export default [
    {
        path: '/',
        component: App,
        routes: [
            {
                path: '/',
                component: Home,
                exact: true,
                key:'/home',
                loadData:Home.loadData
            },
            {
                path: '/user',
                component: User,
                key: '/user',
                routes: [
                    {
                        path: '/user/list',
                        component: UserList,
                        key:'/user/list'
                    }
                ]
            },
            {
                path: '/counter',
                component: Counter,
                key:'login',
                exact: true
            }
        ]
    }

]
/**
export default (
    <Fragment>
        <Route path="/" exact component={Home}></Route>
        <Route path="/counter" exact component={Counter}></Route>
    </Fragment>
)
*/
```
### 16.4 src/server/index.js
src/server/index.js
```js
import express from 'express';
import proxy from 'express-http-proxy';
import render from './render';
let app=express();
app.use(express.static('public'));
app.use('/api',proxy('http://localhost:4000',{
    //修改请求路径
    proxyReqPathResolver: function (req) {
        return `/api${req.url}`;
    }
}));
//context数据的传递 StaticRouter需要知道当前路径
app.get('*',(req,res) => {
    render(req,res);
});
app.listen(9090);
```
### 16.5 src/server/render.js
src/server/render.js
```js
import React,{Component,Fragment} from 'react';
import {StaticRouter} from 'react-router-dom';
import Header from '../components/Header';
import routes from '../routes';
import {renderToString} from 'react-dom/server';
import {matchRoutes,renderRoutes} from 'react-router-config';
import {getStore} from '../store';
import {Provider} from 'react-redux';
import App from '../containers/App';
export default function (req,res) {
    let store=getStore();
    /**
    let matchedRoutes=routes.filter(route => {
        return matchPath(req.path,route);
    });
    */
    let matchedRoutes=matchRoutes(routes,req.path);
    let promises=[];
    matchedRoutes.forEach(item => {
        if (item.route.loadData)
            promises.push(item.route.loadData(store));
    });
    Promise.all(promises).then(result => {
        const content=renderToString(
            <Provider store={store}>
                <StaticRouter context={{}} location={req.path}>
                    {renderRoutes(routes)}
                </StaticRouter>
            </Provider>
        );
        res.send(`
        <!DOCTYPE html>
        <html lang="en">
            <head>
                <meta charset="UTF-8">
                <meta name="viewport" content="width=device-width, initial-scale=1.0">
                <meta http-equiv="X-UA-Compatible" content="ie=edge">
                <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css" >
                <title>珠峰SSR</title>
            </head>
            <body>
                <div id="root">${content}</div>
                <script>
                  window.context = {
                      state:${JSON.stringify(store.getState())}
                  }
                </script>
                <script src="/index.js"></script>
            </body>
        </html>
        `);
    });
}
```
### 16.6 src/containers/App.js
src/containers/App.js
```js
import React,{Component,Fragment} from 'react';
import {renderRoutes} from 'react-router-config';
import Header from '../components/Header';
export default class App extends Component{
    render() {
        return (
            <Fragment>
                <Header/>
                <div className="container" style={{marginTop: 70}}>
                    <Fragment>
                      {renderRoutes(this.props.route.routes)}
                    </Fragment>
                </div>
            </Fragment>
        )
    }
}
```
## 17.实现头部导航
### 17.1 src/client/index.js
src/client/index.js
```js
import React from 'react';
import ReactDOM from 'react-dom';
import {BrowserRouter} from 'react-router-dom';
import {Provider} from 'react-redux';
import routes from '../routes';
import {renderRoutes} from 'react-router-config';
import {getClientStore} from '../store';
ReactDOM.hydrate(
    <Provider store={getClientStore()}>
        <BrowserRouter>
          {renderRoutes(routes)}
      </BrowserRouter>
  </Provider>
    ,document.querySelector('#root')
);
```
### 17.2 src/components/Header/index.js
src/components/Header/index.js
```js
import React,{Component} from 'react';
import {Link} from 'react-router-dom';
import {connect} from 'react-redux';
class Header extends Component{
    render() {
        return (
            <nav className="navbar navbar-inverse navbar-fixed-top">
                    <div className="container">
                        <div className="navbar-header">
                            <a className="navbar-brand" href="#">珠峰SSR</a>
                        </div>
                        <div id="navbar" className="collapse navbar-collapse">
                            <ul className="nav navbar-nav">
                              <li><Link to="/">首页</Link></li>
                              <li><Link to="/user/list">用户列表</Link></li>
                              {
                                !this.props.user&&<li><Link to="/login">登录</Link></li>
                              }
                              {
                                this.props.user&&<Fragment><li><Link to="/logout">退出</Link></li><li><Link to="/profile">个人中心</Link></li></Fragment>
                              }
                            </ul>
                        </div>
                    </div>
                </nav>
        )
    }
}
export default connect(
    state=>state.session
)(Header);
```
### 17.3 src/store/action-types.js
src/store/action-types.js
```js
//counter
export const INCREMENT='INCREMENT';
//home
export const SET_HOME_LIST='SET_HOME_LIST';
//登录
export const LOGIN='LOGIN';
//退出
export const LOGOUT='LOGOUT';
```
### 17.4 reducers/index.js
src/store/reducers/index.js
```js
import {combineReducers} from 'redux';
import home from './home';
import counter from './counter';
import session from './session';
let reducers=combineReducers({
    home,
    counter,
    session
});
export default reducers;
```
### 17.5 reducers/session.js
src/store/reducers/session.js
```js
import * as types from '../action-types';
let initState = {
    user:null
};
export default function (state = initState, action) {
    switch (action.type) {
        default:
          return state;
    }
}
```
## 18.实现登录功能
### 18.1 api/server.js
```js
let express=require('express');
let cors=require('cors');
let morgan=require('morgan');
let session=require('express-session');
let bodyParser=require('body-parser');
let app=express();
app.use(morgan('tiny'));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({extended: true}));
app.use(session({
    resave: true,
    saveUninitialized: true,
    secret:'zfpx'
}));
var corsOptions = {
    origin: '*',
    optionsSuccessStatus: 200 
}
app.use(cors(corsOptions));
let users=[{id:1,name:'zfpx1'},{id:2,name:'zfpx2'}];
app.get('/api/users',function (req,res) {
    res.json(users);
});
app.post('/api/login',function (req,res) {
    let user=req.body;
    req.session.user=user;
    res.json({
        code: 0,
        user,
        success:'登录成功'
    });
});
app.get('/api/logout',function (req,res) {
    req.session.user=null;
    res.json({
        code: 0,
        success:'退出成功'
    });
});
app.get('/api/user',function (req,res) {
    if (req.session.user) {
        res.json({
            code: 0,
            user,
            success:'获取用户信息成功'
        });
    } else {
        res.json({
            code: 1,
            error:'此用户未登录'
        });
    }
});
app.listen(4000);
```
### 18.2 Header/index.js
src/components/Header/index.js
```js
import React,{Component,Fragment} from 'react';
import {Link} from 'react-router-dom';
import {connect} from 'react-redux';
class Header extends Component{
    render() {
        return (
            <nav className="navbar navbar-inverse navbar-fixed-top">
                    <div className="container">
                        <div className="navbar-header">
                            <a className="navbar-brand" href="#">珠峰SSR</a>
                        </div>
                        <div id="navbar" className="collapse navbar-collapse">
                            <ul className="nav navbar-nav">
                              <li><Link to="/">首页</Link></li>
                              <li><Link to="/user/list">用户列表</Link></li>
                              {
                                !this.props.user&&<li><Link to="/login">登录</Link></li>
                              }
                              {
                                this.props.user&&<Fragment><li><Link to="/logout">退出</Link></li><li><Link to="/profile">个人中心</Link></li></Fragment>
                              }
                        </ul>
                        {
                            this.props.user&&(
                                <ul class="nav navbar-nav navbar-right">
                                    <li><a href="#">欢迎 {this.props.user.username}</a></li>
                                </ul>
                            )
                         }
                        </div>
                    </div>
                </nav>
        )
    }
}
export default connect(
    state=>state.session
)(Header);
```
### 18.3 src/containers/App.js
src/containers/App.js
```js
import React,{Component,Fragment} from 'react';
import {renderRoutes} from 'react-router-config';
import Header from '../components/Header';
export default class App extends Component{
    static loadData=(store) => {
        store.dispatch();
    }
    render() {
        return (
            <Fragment>
                <Header/>
                <div className="container" style={{marginTop: 70}}>
                    <Fragment>
                      {renderRoutes(this.props.route.routes)}
                    </Fragment>
                </div>
            </Fragment>
        )
    }
}
```
### 18.4 src/routes.js
src/routes.js
```js
import React,{Fragment} from 'react';
import Home from './containers/Home';
import User from './containers/User';
import UserList from './containers/User/components/UserList';
import Counter from './containers/Counter';
import Login from './containers/Login';
import Logout from './containers/Logout';
import Profile from './containers/Profile';
import App from './containers/App';
export default [
    {
        path: '/',
        component: App,
        routes: [
            {
                path: '/',
                component: Home,
                exact: true,
                key:'/home',
                loadData:Home.loadData
            },
            {
                path: '/user',
                component: User,
                key: '/user',
                routes: [
                    {
                        path: '/user/list',
                        component: UserList,
                        key:'/user/list'
                    }
                ]
            },
            {
                path: '/counter',
                component: Counter,
                key:'counter',
                exact: true
            },
            {
                path: '/login',
                component: Login,
                key:'/login',
                exact: true
            },
            {
                path: '/logout',
                component: Logout,
                key:'/logout',
                exact: true
            },
            {
                path: '/profile',
                component: Profile,
                key:'/profile',
                exact: true
            }
        ]
    }

]
/**
export default (
    <Fragment>
        <Route path="/" exact component={Home}></Route>
        <Route path="/counter" exact component={Counter}></Route>
    </Fragment>
)
*/
```
### 18.5 src/server/index.js
src/server/index.js
```js
import express from 'express';
import proxy from 'express-http-proxy';
import render from './render';
let app=express();
app.use(express.static('public'));

app.use('/api',proxy('http://localhost:4000',{
    //修改请求路径
    proxyReqPathResolver: function (req) {
        return `/api${req.url}`;
    }
}));
//context数据的传递 StaticRouter需要知道当前路径
app.get('*',(req,res) => {
    render(req,res);
});
app.listen(9090);
```
### 18.6 action-types.js
src/store/action-types.js
```js
//counter
export const INCREMENT='INCREMENT';
//home
export const SET_HOME_LIST='SET_HOME_LIST';
//登录
export const LOGIN='LOGIN';
//退出
export const LOGOUT='LOGOUT';
//设置会话
export const SET_SESSION='SET_SESSION';
```
### 18.7 reducers/session.js
src/store/reducers/session.js
```js
import * as types from '../action-types';
let initState = {
    user: null,
    success: null,
    error:null
};
export default function (state = initState, action) {
    switch (action.type) {
        case types.SET_SESSION:
            return {...action.payload};
        default:
          return state;
    }
}
```
### 18.7 Login/index.js
src/containers/Login/index.js
```js
import React,{Component} from 'react';
import actions from '../../store/actions/session';
import {connect} from 'react-redux';
class Login extends Component{
    state={
        username:''
    }
    handleChange=(event) => {
        this.setState({username:event.target.value});    
    }
    handleSubmit=(event) => {
        event.preventDefault();
        this.props.login(this.state);
    }
    render() {
        return (
            <div className="row">
                <div className="col-md-12">
                    <form onSubmit={this.handleSubmit}>
                        <div className="form-group">
                            <label htmlFor="username">用户名</label>
                            <input value={this.state.username} onChange={this.handleChange} type="text" className="form-control"/>
                        </div>
                        <div className="form-group">
                            <input type="submit" className="btn btn-primary"/>
                        </div>
                    </form>

                </div>
            </div>
        )
    }
}
export default connect(
    state => state.session,
    actions
)(Login)
```
### 18.8 Logout/index.js
src/containers/Logout/index.js
```js
import React,{Component} from 'react';
export default class Profile extends Component{
    render() {
        return (
            <div className="row">
                <div className="col-md-12">
                    <button className="btn btn-primary">退出</button>
                </div>
            </div>
        )
    }
}
```
### 18.9 Profile/index.js
src/containers/Profile/index.js
```js
import React,{Component} from 'react';
export default class Profile extends Component{
    render() {
        return (
            <div className="row">
                <div className="col-md-12">
                    个人中心
                </div>
            </div>
        )
    }
}
```
### 18.10 actions/session.js
src/store/actions/session.js
```js
import * as types from '../action-types';
export default {
    login(user) {
        //https://github.com/reduxjs/redux-thunk/blob/master/src/index.js
        return function (dispatch,getState,request) {
            //http://localhost:4000/api/users
            return request.post('/api/login',user).then(result => {
                dispatch({
                    type: types.SET_SESSION,
                    payload:result.data
                });
            });
        }
    }
}
```
## 19.退出功能
### 19.1 Logout/index.js
src/containers/Logout/index.js
```js
import React,{Component} from 'react';
import actions from '../../store/actions/session';
import {connect} from 'react-redux';
class Profile extends Component{
    handleLogout=() => {
        this.props.logout();
    }
    render() {
        return (
            <div className="row">
                <div className="col-md-12">
                    <button className="btn btn-primary" onClick={this.handleLogout}>退出</button>
                </div>
            </div>
        )
    }
}
export default connect(
    state => state.session,
    actions
)(Profile)
```
### 19.2 actions/session.js
src/store/actions/session.js
```js
import * as types from '../action-types';
export default {
    login(user) {
        //https://github.com/reduxjs/redux-thunk/blob/master/src/index.js
        return function (dispatch,getState,request) {
            //http://localhost:4000/api/users
            return request.post('/api/login',user).then(result => {
                dispatch({
                    type: types.SET_SESSION,
                    payload:result.data
                });
            });
        }
    },
    logout() {
        //https://github.com/reduxjs/redux-thunk/blob/master/src/index.js
        return function (dispatch,getState,request) {
            //http://localhost:4000/api/users
            return request.get('/api/logout').then(result => {
                dispatch({
                    type: types.SET_SESSION,
                    payload:result.data
                });
            });
        }
    }
}
```
## 20. 加载用户信息
### 20.1 api/server.js
```js
let express=require('express');
let cors=require('cors');
let morgan=require('morgan');
let session=require('express-session');
let bodyParser=require('body-parser');
let app=express();
app.use(morgan('tiny'));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({extended: true}));
app.use(session({
    resave: true,
    saveUninitialized: true,
    secret:'zfpx'
}));
var corsOptions = {
    origin: '*',
    optionsSuccessStatus: 200 
}
app.use(cors(corsOptions));
let users=[{id:1,name:'zfpx1'},{id:2,name:'zfpx2'}];
app.get('/api/users',function (req,res) {
    res.json(users);
});
app.post('/api/login',function (req,res) {
    let user=req.body;
    req.session.user=user;
    res.json({
        code: 0,
        data: {
            user,
            success:'登录成功'
        }
    });
});
app.get('/api/logout',function (req,res) {
    req.session.user=null;
    res.json({
        code: 0,
        data: {
            success:'退出成功'
        }

    });
});
app.get('/api/user',function (req,res) {
    if (req.session.user) {
        res.json({
            code: 0,
            data: {
                user:req.session.user,
                success:'获取用户信息成功'
            }
        });
    } else {
        res.json({
            code: 1,
            data: {
                error:'此用户未登录'
            }
        });
    }

});
app.listen(4000);
```
### 20.2 containers/App.js
src/containers/App.js
```js
import React,{Component,Fragment} from 'react';
import {renderRoutes} from 'react-router-config';
import Header from '../components/Header';
import actions from '../store/actions/session';
export default class App extends Component{
    static loadData=(store) => {
        return store.dispatch(actions.getUser());
    }
    render() {
        return (
            <Fragment>
                <Header/>
                <div className="container" style={{marginTop: 70}}>
                    <Fragment>
                      {renderRoutes(this.props.route.routes)}
                    </Fragment>
                </div>
            </Fragment>
        )
    }
}
```
### 20.3 Profile/index.js
src/containers/Profile/index.js
```js
import React,{Component} from 'react';
import actions from '../../store/actions/session';
import {Redirect} from 'react-router-dom';
import {connect} from 'react-redux';
class Profile extends Component{
    render() {
        return this.props.user? (
            <div className="row">
                <div className="col-md-12">
                    {this.props.user.username}
                </div>
            </div>
        ):<Redirect to="/login"/>;
    }
}
export default connect(
    state => state.session,
    actions
)(Profile)
```
### 20.4 src/routes.js
src/routes.js
```js
import React,{Fragment} from 'react';
import Home from './containers/Home';
import User from './containers/User';
import UserList from './containers/User/components/UserList';
import Counter from './containers/Counter';
import Login from './containers/Login';
import Logout from './containers/Logout';
import Profile from './containers/Profile';
import App from './containers/App';
export default [
    {
        path: '/',
        component: App,
        loadData:App.loadData,
        routes: [
            {
                path: '/',
                component: Home,
                exact: true,
                key:'/home',
                loadData:Home.loadData
            },
            {
                path: '/user',
                component: User,
                key: '/user',
                routes: [
                    {
                        path: '/user/list',
                        component: UserList,
                        key:'/user/list'
                    }
                ]
            },
            {
                path: '/counter',
                component: Counter,
                key:'counter',
                exact: true
            },
            {
                path: '/login',
                component: Login,
                key:'/login',
                exact: true
            },
            {
                path: '/logout',
                component: Logout,
                key:'/logout',
                exact: true
            },
            {
                path: '/profile',
                component: Profile,
                key:'/profile',
                exact: true
            }
        ]
    }

]
/**
export default (
    <Fragment>
        <Route path="/" exact component={Home}></Route>
        <Route path="/counter" exact component={Counter}></Route>
    </Fragment>
)
*/
```
### 20.5 src/server/render.js
src/server/render.js
```js
import React,{Component,Fragment} from 'react';
import {StaticRouter} from 'react-router-dom';
import Header from '../components/Header';
import routes from '../routes';
import {renderToString} from 'react-dom/server';
import {matchRoutes,renderRoutes} from 'react-router-config';
import {getStore} from '../store';
import {Provider} from 'react-redux';
import App from '../containers/App';
export default function (req,res) {
    let store=getStore(req);
    /**
    let matchedRoutes=routes.filter(route => {
        return matchPath(req.path,route);
    });
    */
    let matchedRoutes=matchRoutes(routes,req.path);
    let promises=[];
    matchedRoutes.forEach(item => {
        if (item.route.loadData)
            promises.push(item.route.loadData(store));
    });
    Promise.all(promises).then(result => {
        const content=renderToString(
            <Provider store={store}>
                <StaticRouter context={{}} location={req.path}>
                    {renderRoutes(routes)}
                </StaticRouter>
            </Provider>
        );
        res.send(`
        <!DOCTYPE html>
        <html lang="en">
            <head>
                <meta charset="UTF-8">
                <meta name="viewport" content="width=device-width, initial-scale=1.0">
                <meta http-equiv="X-UA-Compatible" content="ie=edge">
                <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css" >
                <title>珠峰SSR</title>
            </head>
            <body>
                <div id="root">${content}</div>
                <script>
                  window.context = {
                      state:${JSON.stringify(store.getState())}
                  }
                </script>
                <script src="/index.js"></script>
            </body>
        </html>
        `);
    });
}
```
### 20.6 src/server/request.js
src/server/request.js
```js
import axios from 'axios';
export default (req)=>axios.create({
    baseURL: 'http://localhost:4000/',
    headers: {
        cookie:req.get('cookie')||''
    }
});
```
### 20.7 actions/session.js
src/store/actions/session.js
```js
import * as types from '../action-types';
export default {
    login(user) {
        //https://github.com/reduxjs/redux-thunk/blob/master/src/index.js
        return function (dispatch,getState,request) {
            //http://localhost:4000/api/users
            return request.post('/api/login',user).then(result => {
                dispatch({
                    type: types.SET_SESSION,
                    payload:result.data.data
                });
            });
        }
    },
    logout() {
        //https://github.com/reduxjs/redux-thunk/blob/master/src/index.js
        return function (dispatch,getState,request) {
            //http://localhost:4000/api/users
            return request.get('/api/logout').then(result => {
                dispatch({
                    type: types.SET_SESSION,
                    payload:result.data.data
                });
            });
        }
    },
    getUser() {
        return function (dispatch,getState,request) {
            //http://localhost:4000/api/users
            return request.get('/api/user').then(result => {
                dispatch({
                    type: types.SET_SESSION,
                    payload:result.data.data
                });
            });
        }
    }
}
```
### 20.7 src/store/index.js
src/store/index.js
```js
import reducers from './reducers';
import {createStore,applyMiddleware} from 'redux';
import clientRequest from '../client/request';
import getServerRequest from '../server/request';
import thunk from 'redux-thunk';
import logger from 'redux-logger';
export function getStore(req) {
    return createStore(reducers,applyMiddleware(thunk.withExtraArgument(getServerRequest(req)),logger));
}
export function getClientStore() {
    let initState=window.context.state;
    return createStore(reducers,initState,applyMiddleware(thunk.withExtraArgument(clientRequest),logger));
}
```
## 21. 404
### 21.1 src/routes.js
```js
import Home from './containers/Home';
import User from './containers/User';
import UserList from './containers/User/components/UserList';
import Counter from './containers/Counter';
import Login from './containers/Login';
import Logout from './containers/Logout';
import Profile from './containers/Profile';
import NotFound from './containers/NotFound';
import App from './containers/App';
export default [
    {
        path: '/',
        component: App,
        loadData:App.loadData,
        routes: [
            {
                path: '/',
                component: Home,
                exact: true,
                key:'/home',
                loadData:Home.loadData
            },
            {
                path: '/user',
                component: User,
                key: '/user',
                routes: [
                    {
                        path: '/user/list',
                        component: UserList,
                        key:'/user/list'
                    }
                ]
            },
            {
                path: '/counter',
                component: Counter,
                key:'counter',
                exact: true
            },
            {
                path: '/login',
                component: Login,
                key:'/login',
                exact: true
            },
            {
                path: '/logout',
                component: Logout,
                key:'/logout',
                exact: true
            },
            {
                path: '/profile',
                component: Profile,
                key:'/profile',
                exact: true
            },
            {
                component: NotFound
            }
        ]
    }

]
/**
export default (
    <Fragment>
        <Route path="/" exact component={Home}></Route>
        <Route path="/counter" exact component={Counter}></Route>
    </Fragment>
)
*/
```
### 21.2 src/server/render.js
src/server/render.js
```js
import React,{Component,Fragment} from 'react';
import {StaticRouter} from 'react-router-dom';
import Header from '../components/Header';
import routes from '../routes';
import {renderToString} from 'react-dom/server';
import {matchRoutes,renderRoutes} from 'react-router-config';
import {getStore} from '../store';
import {Provider} from 'react-redux';
import App from '../containers/App';
export default function (req,res) {
    let store=getStore(req);
    /**
    let matchedRoutes=routes.filter(route => {
        return matchPath(req.path,route);
    });
    */
    let matchedRoutes=matchRoutes(routes,req.path);
    let promises=[];
    matchedRoutes.forEach(item => {
        if (item.route.loadData)
            promises.push(item.route.loadData(store));
    });
    Promise.all(promises).then(result => {
        let context={};
        const content=renderToString(
            <Provider store={store}>
                <StaticRouter context={context} location={req.path}>
                    {renderRoutes(routes)}
                </StaticRouter>
            </Provider>
        );
        if (context.notFound) {
            res.status(404);
        }
        res.send(`
        <!DOCTYPE html>
        <html lang="en">
            <head>
                <meta charset="UTF-8">
                <meta name="viewport" content="width=device-width, initial-scale=1.0">
                <meta http-equiv="X-UA-Compatible" content="ie=edge">
                <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css" >
                <title>珠峰SSR</title>
            </head>
            <body>
                <div id="root">${content}</div>
                <script>
                  window.context = {
                      state:${JSON.stringify(store.getState())}
                  }
                </script>
                <script src="/index.js"></script>
            </body>
        </html>
        `);
    });
}
```
### 21.3 NotFound/index.js
src/containers/NotFound/index.js
```js
import React,{Component} from 'react';
export default class NotFound extends Component{
    componentWillMount() {
        if (this.props.staticContext) {
            this.props.staticContext.notFound=true;
        }
    }
    render() {
        return (
            <div>404</div>
        )
    }
}
```
## 22. 301
### 22.1 server/render.js 
src/server/render.js
```js
import React,{Component,Fragment} from 'react';
import {StaticRouter} from 'react-router-dom';
import Header from '../components/Header';
import routes from '../routes';
import {renderToString} from 'react-dom/server';
import {matchRoutes,renderRoutes} from 'react-router-config';
import {getStore} from '../store';
import {Provider} from 'react-redux';
import App from '../containers/App';
export default function (req,res) {
    let store=getStore(req);
    /**
    let matchedRoutes=routes.filter(route => {
        return matchPath(req.path,route);
    });
    */
    let matchedRoutes=matchRoutes(routes,req.path);
    let promises=[];
    matchedRoutes.forEach(item => {
        if (item.route.loadData)
            promises.push(item.route.loadData(store));
    });
    Promise.all(promises).then(result => {
        let context={};
        const content=renderToString(
            <Provider store={store}>
                <StaticRouter context={context} location={req.path}>
                    {renderRoutes(routes)}
                </StaticRouter>
            </Provider>
        );
        if (context.action == 'REPLACE') {
            return res.redirect(301,context.url);
        } else if (context.notFound) {
            res.status(404);
        }
        res.send(`
        <!DOCTYPE html>
        <html lang="en">
            <head>
                <meta charset="UTF-8">
                <meta name="viewport" content="width=device-width, initial-scale=1.0">
                <meta http-equiv="X-UA-Compatible" content="ie=edge">
                <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css" >
                <title>珠峰SSR</title>
            </head>
            <body>
                <div id="root">${content}</div>
                <script>
                  window.context = {
                      state:${JSON.stringify(store.getState())}
                  }
                </script>
                <script src="/index.js"></script>
            </body>
        </html>
        `);
    });
}
```
## 23. promise.all
### 23.1 src/server/render.js
```js
let promises=[];
matchedRoutes.forEach(item => {
    if (item.route.loadData) {
        let promise=new Promise(function (resolve,reject) {
            return item.route.loadData(store).then(resolve,resolve);
        });
        promises.push(promise);
    }    
});
```
## 24.使用CSS
### 24.1 Header/index.js
src/components/Header/index.js
```js
import React,{Component,Fragment} from 'react';
import {Link} from 'react-router-dom';
import {connect} from 'react-redux';
import styles from './index.css';
class Header extends Component{
    render() {
        return (
            <nav className="navbar navbar-inverse navbar-fixed-top">
                    <div className="container">
                        <div className="navbar-header">
                            <a className="navbar-brand" href="#">珠峰SSR</a>
                        </div>
                        <div id="navbar" className="collapse navbar-collapse">
                            <ul className="nav navbar-nav">
                              <li><Link to="/">首页</Link></li>
                              <li><Link to="/user/list">用户列表</Link></li>
                              {
                                !this.props.user&&<li><Link to="/login">登录</Link></li>
                              }
                              {
                                this.props.user&&<Fragment><li><Link to="/logout">退出</Link></li><li><Link to="/profile">个人中心</Link></li></Fragment>
                              }
                        </ul>
                        {
                            this.props.user&&(
                                <ul className="nav navbar-nav navbar-right">
                                    <li><a href="#">欢迎 <span className={styles.user}>{this.props.user.username}</span></a></li>
                                </ul>
                            )
                         }
                        </div>
                    </div>
                </nav>
        )
    }
}
export default connect(
    state=>state.session
)(Header);
```
### 24.2 src/containers/App.js
src/containers/App.js
```js
import React,{Component,Fragment} from 'react';
import {renderRoutes} from 'react-router-config';
import Header from '../components/Header';
import actions from '../store/actions/session';
import styles from './App.css';
export default class App extends Component{
    static loadData=(store) => {
        return store.dispatch(actions.getUser());
    }
    render() {
        return (
            <Fragment>
                <Header/>
                <div className="container" className={styles.app}>
                    <Fragment>
                      {renderRoutes(this.props.route.routes)}
                    </Fragment>
                </div>
            </Fragment>
        )
    }
}
```
### 24.3 webpack.client.js
webpack.client.js
```js
module:{
    rules:[
        {
            test: /\.css$/,
            use: [
                'style-loader',
                {
                    loader: 'css-loader',
                    options: {
                        modules: true,
                        localIdentName:'[name]_[local]_[hash:base64:5]'
                    }
                }
            ]
        }
    ]
}
```
### 24.4 webpack.server.js
webpack.server.js
```js
module:{
    rules:[
        {
            test: /\.css$/,
            use: [
                'isomorphic-style-loader',
                {
                    loader: 'css-loader',
                    options: {
                        modules: true,
                        localIdentName:'[name]_[local]_[hash:base64:5]'
                    }
                }
            ]
        }
    ]
}
```
### 24.5 Header/index.css
src/components/Header/index.css
```css
.user{
    color:red;
}
```
### 24.6 src/containers/App.css
src/containers/App.css
```css
.app{
    margin-top:70px;
}
```
## 25. CSS服务端渲染
### 25.1 Header/index.js
src/components/Header/index.js
```js
+    componentWillMount() {
+        if (this.props.staticContext) {
+            this.props.staticContext.csses.push(styles._getCss());
+        }
+    }
```
### 25.2 containers/App.js
src/containers/App.js
```js
+    componentWillMount() {
+        if (this.props.staticContext) {
+            this.props.staticContext.csses.push(styles._getCss());
+        }
+    }
```
### 25.3 server/render.js
src/server/render.js
```js
let context={csses:[]};
        const content=renderToString(
            <Provider store={store}>
                <StaticRouter context={context} location={req.path}>
                    {renderRoutes(routes)}
                </StaticRouter>
            </Provider>
        );
+        let cssStr='';
+        if (context.csses.length>0) {
+            cssStr=context.csses.join('\r\n');
+        }
        if (context.action == 'REPLACE') {
            return res.redirect(301,context.url);
        } else if (context.notFound) {
            res.status(404);
        }
        res.send(`
        <!DOCTYPE html>
        <html lang="en">
            <head>
                <meta charset="UTF-8">
                <meta name="viewport" content="width=device-width, initial-scale=1.0">
                <meta http-equiv="X-UA-Compatible" content="ie=edge">
                <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css" >
+                <style>${cssStr}</style>
                <title>珠峰SSR</title>
            </head>
            <body>
                <div id="root">${content}</div>
                <script>
                  window.context = {
                      state:${JSON.stringify(store.getState())}
                  }
                </script>
                <script src="/index.js"></script>
            </body>
        </html>
        `);
```
## 26.优化CSS服务器端渲染
### 26.1 Header/index.js
src/components/Header/index.js
```js
import React,{Component,Fragment} from 'react';
import {Link} from 'react-router-dom';
import {connect} from 'react-redux';
import styles from './index.css';
import withStyles from '../../withStyles';
class Header extends Component{
    render() {
        return (
            <nav className="navbar navbar-inverse navbar-fixed-top">
                <div className="container">
                    <div className="navbar-header">
                        <a className="navbar-brand" href="#">珠峰SSR</a>
                    </div>
                    <div id="navbar" className="collapse navbar-collapse">
                        <ul className="nav navbar-nav">
                            <li><Link to="/">首页</Link></li>
                            <li><Link to="/user/list">用户列表</Link></li>
                            {
                            !this.props.user&&<li><Link to="/login">登录</Link></li>
                            }
                            {
                            this.props.user&&<Fragment><li><Link to="/logout">退出</Link></li><li><Link to="/profile">个人中心</Link></li></Fragment>
                            }
                    </ul>
                    {
                        this.props.user&&(
                            <ul className="nav navbar-nav navbar-right">
                                <li><a href="#">欢迎 <span className={styles.user}>{this.props.user.username}</span></a></li>
                            </ul>
                        )
                        }
                    </div>
                </div>
            </nav>
        )
    }
}
export default connect(
    state=>state.session
)(withStyles(Header,styles));
```
### 26.2 src/containers/App.js
src/containers/App.js
```js
import React,{Component,Fragment} from 'react';
import {renderRoutes} from 'react-router-config';
import Header from '../components/Header';
import actions from '../store/actions/session';
import styles from './App.css';
import withStyles from '../withStyles';
class App extends Component{
    componentWillMount() {
        if (this.props.staticContext) {
            this.props.staticContext.csses.push(styles._getCss());
        }
    }
    render() {
        return (
            <Fragment>
                <Header staticContext={this.props.staticContext}/>
                <div className="container" className={styles.app}>
                    <Fragment>
                      {renderRoutes(this.props.route.routes)}
                    </Fragment>
                </div>
            </Fragment>
        )
    }
}
let Proxy=withStyles(App,styles);
Proxy.loadData=(store) => {
    return store.dispatch(actions.getUser());
}
export default Proxy;
```
### 26.3 Home/index.js
src/containers/Home/index.js
```js
import React,{Component} from 'react';
import {connect} from 'react-redux';
import actions from '../../store/actions/home';
import withStyles from '../../withStyles';
class Home extends Component{
    static loadData=(store) => {
        //dispatch方法的返回值是action
        //https://github.com/reduxjs/redux/blob/master/src/createStore.js
        return store.dispatch(actions.getHomeList());
    }
    //componentDidMount在服务器端是不执行的
    componentDidMount() {
        if(this.props.list.length==0)
            this.props.getHomeList();
    }
    render() {
        return (
            <div className="row">
                <div className="col-md-12">
                    <ul className="list-group">
                        {
                            this.props.list.map(item => (
                                <li className="list-group-item" key={item.id}>{item.name}</li>
                            ))
                        }
                    </ul>
                </div>
            </div>
        )
    }
}
export default connect(
    state => state.home,
    actions
)(Home);
```
### 26.4 src/withStyles.js
src/withStyles.js
```js
import React,{Component,Fragment} from 'react';

export default function withStyles(OriginalComponent,styles) {
    class ProxyComponent extends Component{
        componentWillMount() {
            if (this.props.staticContext) {
                this.props.staticContext.csses.push(styles._getCss());
            }
        }
        render() {
            return <OriginalComponent {...this.props}/>
        }
    }
    return ProxyComponent;
}
```
## 27. SEO优化
### 27.1 Home/index.js
src/containers/Home/index.js
```js
import React,{Component,Fragment} from 'react';
import {connect} from 'react-redux';
import {Helmet} from 'react-helmet';
import actions from '../../store/actions/home';

class Home extends Component{
    static loadData=(store) => {
        //dispatch方法的返回值是action
        //https://github.com/reduxjs/redux/blob/master/src/createStore.js
        return store.dispatch(actions.getHomeList());
    }
    //componentDidMount在服务器端是不执行的
    componentDidMount() {
        if(this.props.list.length==0)
            this.props.getHomeList();
    }
    render() {
        return (
            <Fragment>
                <Helmet>
                  <title>首页标题</title>
                  <meta name="description" content="首页描述"></meta>
                </Helmet>
                  <div className="row">
                    <div className="col-md-12">
                        <ul className="list-group">
                            {
                                this.props.list.map(item => (
                                    <li className="list-group-item" key={item.id}>{item.name}</li>
                                ))
                            }
                        </ul>
                    </div>
                  </div>
            </Fragment>
        )
    }
}
export default connect(
    state => state.home,
    actions
)(Home);
```
### 27.2 src/server/render.js
src/server/render.js
```js
import {Helmet} from 'react-helmet';
let helmet=Helmet.renderStatic();
let cssStr='';
if (context.csses.length>0) {
    cssStr=context.csses.join('\r\n');
}
if (context.action == 'REPLACE') {
    return res.redirect(301,context.url);
} else if (context.notFound) {
    res.status(404);
}
res.send(`
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        ${helmet.title.toString()}
        ${helmet.meta.toString()}
        <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@3.3.7/dist/css/bootstrap.min.css" >
        <style>${cssStr}</style>
        <title>珠峰SSR</title>
    </head>
    <body>
        <div id="root">${content}</div>
        <script>
            window.context = {
                state:${JSON.stringify(store.getState())}
            }
        </script>
        <script src="/index.js"></script>
    </body>
</html>
`);
```