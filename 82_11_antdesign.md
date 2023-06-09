## 1. 安装 dva-cli
- [dvajs](https://dvajs.com/guide/getting-started.html#connect-%E8%B5%B7%E6%9D%A5)
- [ant.design](https://ant.design/docs/react/getting-started-cn)
```sh
$ npm install dva-cli -g
$ dva -v
```
## 2. 创建新应用
```sh
$ dva new zhufeng-umi-dva
$ cd zhufeng-umi-dva
$ npm start
```
> 在浏览器里打开 [http://localhost:8000](http://localhost:8000/)，你会看到 dva 的欢迎界面。

## 3.使用 antd
- babel-plugin-import 是用来按需加载 antd 的脚本和样式的
- 编辑 .webpackrc，使 babel-plugin-import 插件生效。
```sh
cnpm i antd -S
cnpm i babel-plugin-import -D
```
```js
{
  "extraBabelPlugins": [
    ["import", { "libraryName": "antd", "libraryDirectory": "es", "style": "css" }]
  ]
}
```
## 4. 首页导航
### 4.1 router.js
src/router.js
- 去掉exact属性
```js
+  <Route path="/" component={IndexPage} />
```
### 4.2 IndexPage.js
src\routes\IndexPage.js
```js
import React from 'react';
import { connect } from 'dva';
import styles from './IndexPage.less';
import {Layout} from 'antd';
import NavBar from '../components/NavBar';
const {Content} = Layout;
function IndexPage(props) {
  return (
    <Layout>
      <NavBar {...props}/>
      <Content>内容</Content>
    </Layout>
  );
}
export default connect()(IndexPage);
```
### 4.3 NavBar\index.js
src\components\NavBar\index.js
```js
import styles from './index.less';
import {Link} from 'dva/router';
import React, { Component } from 'react';
import {Layout,Menu} from 'antd';
let logo = require('../../assets/logo.png');
export default class NavBar extends Component {
  render() {
    return (
      <Layout.Header className={styles.header}>
        <img src={logo}/>
        <Menu  className={styles.menu} mode="horizontal" selectedKeys={[this.props.location.pathname]} >
            <Menu.Item key="/home"><Link to="/home">主页</Link></Menu.Item>
            <Menu.Item key="/user"><Link to="/user">用户管理</Link></Menu.Item>
            <Menu.Item key="/profile"><Link to="/profile">个人中心</Link></Menu.Item>
            <Menu.Item key="/login"><Link to="/login">登录</Link></Menu.Item>
            <Menu.Item key="/register"><Link to="/register">注册</Link></Menu.Item>
        </Menu>
      </Layout.Header> 
    )
  }
}
```
### 4.4 NavBar\index.less
src\components\NavBar\index.less
```less
.header{
    height:48px;
    line-height: 48px;
    background-color: #FFF;
    img{
        height:48px;
    }
    .menu{
        display:inline-block;
        height:48px;
    }
}
```
## 5. 实现路由
### 5.1 routes/IndexPage.js
src/routes/IndexPage.js
```js
import React from 'react';
import { connect } from 'dva';
import styles from './IndexPage.less';
import {Layout} from 'antd';
import NavBar from '../components/NavBar';
import {Route,Switch,Redirect} from 'dva/router';
import Home from './Home';
import User from './User';
import Profile from './Profile';
import Register from './Register';
import Login from './Login';
const {Content} = Layout;
function IndexPage(props) {
  return (
    <Layout>
      <NavBar {...props}/>
      <Content>
        <Switch>
          <Route path="/home" component={Home} />
          <Route path="/user" component={User} />
          <Route path="/profile" component={Profile} />
          <Route path="/register" component={Register} />
          <Route path="/login" component={Login} />
          <Redirect  to="/home"/>
        </Switch>
      </Content>
    </Layout>
  );
}
export default connect()(IndexPage);
```
## 6. 配置式路由
### 6.1 src/router.js
src/router.js
```js
import React from 'react';
import { Router, Switch } from 'dva/router';
import {renderRoutes} from './utils/routes';
import routesConfig from './routeConfig';
function RouterConfig({ history }) {
  return (
    <Router history={history}>
      <Switch>
        {renderRoutes(routesConfig)}
      </Switch>
    </Router>
  );
}

export default RouterConfig;
```
### 6.2 routes/IndexPage.js 
src/routes/IndexPage.js
```js
import React from 'react';
import { connect } from 'dva';
import {Layout} from 'antd';
import NavBar from '../components/NavBar';
import {Switch} from 'dva/router';
import NoMatch from '../components/NoMatch';
import {renderRoutes,renderRedirect} from '../utils/routes';
const {Content} = Layout;
function IndexPage(props) {
  return (
    <Layout>
      <NavBar {...props}/>
      <Content>
        <Switch>
          {renderRoutes(props.routes)}
          {/**把/重定向到/home**/}
          {renderRedirect('/',true,props.routes)}
          <NoMatch/>
        </Switch>
      </Content>
    </Layout>
  );
}

export default connect()(IndexPage);
```
- [Redirect](https://reacttraining.com/react-router/web/api/Redirect)
### 6.3 utils/routes.js
src/utils/routes.js
```js
import {Route,Redirect} from 'dva/router';
export function renderRoutes(routesConfig){
    return routesConfig.map(({path,exact=false,component:Component,routes=[]},index)=>(
        <Route path={path} exact={exact} key={index} render={props=><Component {...props} routes={routes}/>}/>
    ))
}
export function renderRedirect(from,exact,routesConfig){
    let {path} = routesConfig.find(route=>route.redirect)||routesConfig[0];
    return <Redirect exact={exact} from={from} to={path}/>
}
```
### 6.4 routeConfig.js
src/routeConfig.js
```js
import IndexPage from './routes/IndexPage';
import Home from './routes/Home';
import User from './routes/User';
import Profile from './routes/Profile';
import Register from './routes/Register';
import Login from './routes/Login';
export default  [
  {
    path:'/',
    component:IndexPage,
    routes:[
      {
        path:'/home',
        redirect:true,
        component:Home
      },
      {
        path:'/user',
        component:User
      },
      {
        path:'/profile',
        component:Profile
      },
      {
        path:'/login',
        component:Login
      }, {
        path:'/register',
        component:Register
      }
    ]
  }
]
```
### 6.5 components\NoMatch\index.js
src\components\NoMatch\index.js
```js
import React from 'react'

export default () => {
  return (
    <div>
      此页面不存在
    </div>
  )
}
```
## 7. 按需加载
### 7.1 src/routeConfig.js
src/routeConfig.js
```js
/* import IndexPage from './routes/IndexPage';
import Home from './routes/Home';
import User from './routes/User';
import Profile from './routes/Profile';
import Register from './routes/Register';
import Login from './routes/Login'; */
export default  [
  {
    path:'/',
    component:()=>import('./routes/IndexPage'),
    routes:[
      {
        path:'/home',
        models:[import('./models/home')],
        component:()=>import('./routes/Home')
      },
      {
        path:'/user',
        component:()=>import('./routes/User')
      },
      {
        path:'/profile',
        component:()=>import('./routes/Profile')
      },
      {
        path:'/login',
        component:()=>import('./routes/Login')
      }, {
        path:'/register',
        component:()=>import('./routes/Register')
      }
    ]
  }
]
```
### 7.2 src/router.js
src/router.js
```js
import React from 'react';
import { Router, Switch } from 'dva/router';
import {renderRoutes} from './utils/routes';
import routesConfig from './routeConfig';
function RouterConfig({ history,app }) {
  return (
    <Router history={history}>
      <Switch>
        {renderRoutes(routesConfig,app)}
      </Switch>
    </Router>
  );
}

export default RouterConfig;
```
### 7.3 routes/Home/index.js
src/routes/Home/index.js
```js
import React, { Component } from 'react'
import styles from './index.less';
import { connect } from 'dva';
class componentName extends Component {
  render() {
    return (
      <div>
        {this.props.title}
      </div>
    )
  }
}
export default connect(state=>state.home)(componentName);
```
### 7.4 routes/IndexPage.js
src/routes/IndexPage.js
```js
import React from 'react';
import { connect } from 'dva';
import {Layout} from 'antd';
import NavBar from '../components/NavBar';
import {Switch} from 'dva/router';
import NoMatch from '../components/NoMatch';
import {renderRoutes,renderRedirect} from '../utils/routes';
const {Content} = Layout;
function IndexPage(props) {
  return (
    <Layout>
      <NavBar {...props}/>
      <Content>
        <Switch>
          {renderRoutes(props.routes,props.app)}
          {renderRedirect('/',true,props.routes)}
          <NoMatch/>
        </Switch>
      </Content>
    </Layout>
  );
}

export default connect()(IndexPage);
```
### 7.5 src/utils/routes.js
src/utils/routes.js
```js
import {Route,Redirect} from 'dva/router';
//import dynamic from 'dva/dynamic';

function dynamic({app,models,component}){
  class Dynamic extends React.Component{
    state={Component: null}
     componentDidMount() {
        Promise.all(
          [
            Promise.all(models()),
            component()
          ]
        ).then(([models,Component])=>{
          models.map(item=>item.default).forEach(model=>app.model(model));
          this.setState({Component});
        });
    }
    render(){
      let Component = this.state.Component;
      return Component&&<Component {...this.props}/>
    }
  }
   return Dynamic;
}
export function renderRoutes(routesConfig,app){
    return routesConfig.map(({path,exact=false,component,routes=[],models=[]},index)=>(
        //<Route path={path} exact={exact} key={index} render={props=><Component {...props} routes={routes}/>}/>
        <Route
         path={path} exact={exact} key={index}
         component={dynamic({
           app,
           models:()=>models,
           component:()=>{
              return component().then(result=>{
                   let Component = result.default || result;
                   return props=><Component {...props} routes={routes}/>;
             })
           }
         })}
        />
    ))
}

export function renderRedirect(from,exact,routesConfig){
    let {path} = routesConfig.find(route=>route.redirect)||routesConfig[0];
    return <Redirect exact={exact} from={from} to={path}/>
}
```
### 7.6 models/home.js
src/models/home.js
```js
export default {
  namespace: 'home',
  state: {
    title:'我是首页'
  }
}
```
## 8. 路由守卫
### 8.1 src/utils/routes.js
```js
export function renderRoutes(routesConfig,app){
+    return routesConfig.map(({path,exact=false,component,routes=[],models=[],authority},index)=>(
        <Route
         path={path} exact={exact} key={index}
         component={dynamic({
           app,
           models:()=>models,
           component:()=>{
+              if(authority && !localStorage.getItem('login')){
+                 return ()=><Redirect to="/login"/>;
+              }
              return component().then(result=>{
                   let Component = result.default || result;
                   return props=><Component {...props} routes={routes} app={app}/>;
              })
           }
         })}
        />
    ))
}
```
### 8.2 routeConfig.js
src/routeConfig.js
```js
{
    path:'/profile',
    component:()=>import('./routes/Profile'),
+        authority:true
},
```