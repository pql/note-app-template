## 1.生成项目
```sh
$ create-react-app jwt-frontend
cd jwt-frontend
cnpm i redux react-redux redux-logger redux-saga react-router-dom react-router-redux@next history axios jsonwebtoken -S
```
## 2.跑通路由和仓库
### 2.1 history.js
```js
import createHistory from 'history/createBrowserHistory';
let history = createHistory();
export default history;
```
### 2.2 src\index.js
```js
+ import {Provider} from 'react-redux';
+ import store from './store';
+ import App from './containers/App';
+ import {ConnectedRouter} from 'react-router-redux';
+ import history from './history';
+ import 'bootstrap/dist/css/bootstrap.css'
+ ReactDOM.render(<Provider store={store}>
+     <ConnectedRouter history={history}>
+         <App />
+     </ConnectedRouter>
+ </Provider>, document.getElementById('root'));
```
### 2.3 src\containers\App.js
```js
import React, {Component, Fragment} from 'react';
import {Link} from 'react-router-dom';
class App extends Component{
    render() {
        return (
            <Fragment>
                <nav className="navbar navbar-default">
                    <div className="container-fluid">
                        <div className="navbar-header">
                            <a className="navbar-brand" href="#">珠峰博客</a>
                        </div>
                        <div>
                            <ul className="nav navbar-nav">
                                <li><Link to="/">首页</Link></li>
                                <li><Link to="/users/signup">注册</Link></li>
                                <li><Link to="/users/signin">登录</Link></li>
                                <li><Link to="/articles/add">发表文章</Link></li>
                            </ul>
                        </div>
                    </div>
                </nav>
            </Fragment>
        )
    }
}
export default App;
```
### 2.4 store/index.js
```js
import {createStore} from 'redux';
import reducers from './reducers';
let store = createStore(reducers);
export default store;
```
### 2.5 store/reducers/index.js
```js
import {combineReducers} from 'redux';
import user from './user';
export default combineReducers({
    user
});
```
### 2.6 store/reducers/user.js
```js
let initState = {};
export default function(state=initState, action) {
    return state;
}
```
## 3.连接组件和仓库
### 3.1 NavHeader.js
src/components/NavHeader.js
```js
import React, {Component} from 'react';
import {Link} from 'react-router-dom';
import {connect} from 'react-redux';
class NavHeader extends Component {
    render() {
        return (
            <nav className="navbar navbar-default">
                <div className="container-fluid">
                    <div className="navbar-header">
                        <a className="navbar-brand" href="#">珠峰博客</a>
                    </div>
                    <div>
                        <ul className="nav navbar-nav">
                            <li><Link to="/">首页</Link></li>
                            <li><Link to="/users/signup">注册</Link></li>
                            <li><Link to="/users/signin">登录</Link></li>
                            <li><Link to="/articles/add">发表文章</Link></li>
                        </ul>
                        <ul className="nav navbar-nav navbar-right">
                            <li><Link to="/users/signout">欢迎:{this.props.username}</Link></li>
                            <li><Link to="/users/signout">退出</Link></li>
                        </ul>
                    </div>
                </div>
            </nav>
        )
    }
}
export default connect(state => state.user)(NavHeader);
```
### 3.2 Home.js
src/containers/Home.js
```js
import React, {Component} from 'react';
export default class Home extends Component {
    render() {
        return (
            <div>Home</div>
        )
    }
}
```
### 3.3 Signin.js
src/containers/Signin.js
```js
import React, {Component} from 'react';
export default class Signin extends Component {
    render() {
        return (
            <div>Signin</div>
        )
    }
}
```
### 3.4 Signup.js
src/containers/Signup.js
```js
import React,{Component} from 'react';
export default class Signup extends Component {
    render() {
        return (
            <div>Signup</div>
        )
    }
}
```
### 3.5 article/Add.js
src/containers/article/Add.js
```js
import React,{Component} from 'react';
export default class Add extends Component {
    render() {
        return (
            <div>Article Add</div>
        )
    }
}
```
### 3.6 reducers/user.js
src/store/reducers/user.js
```js
-let initState={};
+let initState={username:'张三'};
```
## 4.用户登录
### 4.1 NavHeader.js
src/components/NavHeader.js
```js
import React,{Component} from 'react';
import {Link} from 'react-router-dom';
import {connect} from 'react-redux';
import actions from '../store/actions/user';
class NavHeader extends Component {
    componentDidMount() {
        this.props.loadUser();
    }
    render() {
        return (
            <nav className="navbar navbar-default">
                <div className="container-fluid">
                    <div className="navbar-header">
-                       <a className="navbar-brand" href="#">珠峰博客</a>
+                       <a className="navbar-brand" href="/">珠峰博客</a>
                    </div>
                    <div>
                        <ul className="nav navbar-nav">
-                           <li><Link to="/">首页</Link></li>
-                           <li><Link to="/users/signup">注册</Link></li>
-                           <li><Link to="/users/signin">登录</Link></li>
-                           <li><Link to="/articles/add">发表文章</Link></li>
-                       </ul>
-                       <ul className="nav navbar-nav navbar-right">
-                           <li>
-                               <Link to="/users/signout">欢迎:{this.props.username}</Link>
-                            </li>
+                            <li><Link to="/">首页</Link></li>
+                            {
+                                !this.props.user&&(<li><Link to="/users/signup">注册</Link></li>
+                                )
+                            }
+                            {
+                                !this.props.user&&(<li><Link to="/users/signin">登录</Link></li>)
+                            }
+                            {
+                                this.props.user&&this.props.user.admin&&<li><Link to="/articles/add">发表文章</Link></li>
+                            }
+                       </ul>
+                        {
+                            this.props.user&&<ul className="nav navbar-nav navbar-right">
                             <li>
-                                <Link to="/users/signout">退出</Link>
-                            </li>
-                            </ul>
+                                <Link to="/users/signout">欢迎:{this.props.user.username}</Link>
+                             </li>
+                             <li>
+                                    <a onClick={this.props.logout}>退出</a>
+                             </li>
+                             </ul>
+                        }
                    </div>
                </div>
            </nav>
        )
    }
}
-export default connect(state=>state.user)(NavHeader);
+export default connect(state=>state.user,actions)(NavHeader);
```
### 4.2 App.js
src/containers/App.js
```js
-import {Link,Route} from 'react-router-dom';
+import {Route} from 'react-router-dom';
```
### 4.3 Signin.js
src/containers/Signin.js
```js
 import React,{Component} from 'react'
-export default class Signin extends Component{
+import actions from '../store/actions/user';
+import {connect} from 'react-redux';
+class Signin extends Component{
+    handleSubmit=(event) => {
+        event.preventDefault();
+        let username=this.username.value;
+        let password=this.password.value;
+        let user={username,password};
+        this.props.login(user);
+    }
     render() {
         return (
-            <div>Signin</div>
+            <form onSubmit={this.handleSubmit}>
+                <div className="form-group">
+                    <label htmlFor="username">用户名</label>
+                    <input className="form-control" ref={input=>this.username=input}/>
+                </div>
+                <div className="form-group">
+                    <label htmlFor="password">密码</label>
+                    <input className="form-control" ref={input=>this.password=input}/>
+                </div>
+                <div className="form-group">
+                    <input type="submit" className="btn btn-primary"/>
+                </div>
+                {
+                    this.props.error&&(
+                        <div className="form-group">
+                                <div className="alert alert-danger">
+                                    {this.props.error.toString()}
+                                </div>
+                        </div>
+                    )
+                }
+                
+            </form>
         )
     }
-}
+}
+export default connect(
+    state => state.user,
+    actions
+)(Signin);
```
### 4.4 action-types.js
src/store/action-types.js
```js
export const LOGIN = 'LOGIN';
export const LOGOUT = 'LOGOUT';
export const LOGIN_SUCCESS = 'LOGIN_SUCCESS';
export const LOGIN_ERROR = 'LOGIN_ERROR';
export const LOGIN_OUT_SUCCESS = 'LOGIN_OUT_SUCCESS';
export const LOAD_USER = 'LOAD_USER';
```
### 4.5 index.js
src/store.index.js
```js
-import {createStore} from 'redux';
+import {createStore,applyMiddleware} from 'redux';
 import reducers from './reducers';
-let store=createStore(reducers);
+import createSagaMiddelware from 'redux-saga';
+import logger from 'redux-logger';
+import rootSaga from './saga';
+import {routerMiddleware} from 'react-router-redux';
+import history from '../history';
+let router=routerMiddleware(history);
+let sagaMiddleware=createSagaMiddelware();
+let store=createStore(reducers,applyMiddleware(sagaMiddleware,router,logger));
+sagaMiddleware.run(rootSaga);
 export default store;
```
### 4.6 saga.js
src/store/saga.js
```js
import * as types from './action-types';
import {put,call,take,all,takeEvery} from 'redux-saga/effects';
import {push} from 'react-router-redux';
import userApi from './api/user';
import {decode} from '../utils/jwt';
function* login(action) {
    try {
        const response=yield call(userApi.login,action.user);
        let {data: {data: {token}}}=response;
        window.localStorage.setItem('token',token);
        const user=decode(token);
        yield put({type: types.LOGIN_SUCCESS,user});
        yield put(push('/'));
    } catch (error) {
        yield put({type: types.LOGIN_ERROR,error});
    }
}
function* logout() {
    window.localStorage.removeItem('token');
    yield put({type: types.LOGIN_OUT_SUCCESS});
    yield put(push('/'));
}
function* loginFlow() {
    yield takeEvery(types.LOGIN,login);
    yield takeEvery(types.LOGOUT,logout);
}
function* loadUser() {
    let token = window.localStorage.getItem('token');
    if (token) {
        const user=decode(token);
        if (user) {
            yield put({type: types.LOGIN_SUCCESS,user});
            yield put(push('/'));
        } else {
            yield put(push('/users/signin'));
        }
    }
}
function* watchLoadUser() {
    yield takeEvery(types.LOAD_USER,loadUser);
}

export default function* rootSaga() {
    yield all([loginFlow(),watchLoadUser()]);
}
```
### 4.7 user.js
src/store/actions/user.js
```js
import * as types from '../action-types';
export default {
    login(user) {
        return {type: types.LOGIN, user}
    },
    logout() {
        return {type: types.LOGOUT};
    },
    loadUser() {
        return {type: types.LOAD_USER};
    }
}
```
### 4.8 index.js
src/store/api/index.js
```js
import axios from 'axios';
const BASE_URL='http://localhost:8080';
axios.interceptors.request.use(config => {
    let token=localStorage.getItem('token');
    if (token) {
        config.headers.Authorization=token;
    }
    return config;
});
export function post(url,body) {
    return axios.post(BASE_URL+'/users/signin',body);
}
```
### 4.9 user.js
src/store/api/user.js
```js
import {post} from './index';
async function login(body) {
    return post('/users/signin', body);
}
export default {
    login
}
```
### 4.10 index.js
src/store/reducers/index.js
```js
 import {combineReducers} from 'redux';
 import user from './user';
+import {routerReducer} from 'react-router-redux';
 export default combineReducers({
+  router:routerReducer
 });
```
### 4.11 user.js
src/store/reducers/user.js
```js
-let initState={username:'张三'};
+import * as types from '../action-types';
+let initState={};
 export default function (state=initState,action) {
-    return state;
+    switch (action.type) {
+        case types.LOGIN_SUCCESS:
+            return {...state,user:action.user,error: null};
+        case types.LOGIN_ERROR:
+            return {...state,user: null,error: action.error};
+            case types.LOGIN_OUT_SUCCESS:
+            return {...state,user: null,error: null};
+        default:
+            return state;
+    }
}
```
### 4.12 jwt.js
src/utils/jwt.js
```js
import jwt from 'jsonwebtoken';
export function decode(token) {
    return jwt.decode(token);;
}
```
## 5.发表文章权限控制
### 5.1 article/Add.js
src/containers/article/Add.js
```js
 import React,{Component} from 'react'
-export default class Add extends Component{
+import actions from '../../store/actions/article';
+import {connect} from 'react-redux';
+class Signin extends Component{
+    handleSubmit=(event) => {
+        event.preventDefault();
+        let title=this.title.value;
+        let content=this.content.value;
+        let article={title,content};
+        this.props.add(article);
+    }
     render() {
         return (
-            <div>Article Add</div>
+            <form onSubmit={this.handleSubmit}>
+                <div className="form-group">
+                    <label htmlFor="title">标题</label>
+                    <input className="form-control" ref={input=>this.title=input}/>
+                </div>
+                <div className="form-group">
+                    <label htmlFor="content">内容</label>
+                    <textarea className="form-control" ref={input=>this.content=input}/>
+                </div>
+                <div className="form-group">
+                    <input type="submit" className="btn btn-primary"/>
+                </div>
+                {
+                    this.props.error&&(
+                        <div className="form-group">
+                                <div className="alert alert-danger">
+                                    {this.props.error.toString()}
+                                </div>
+                        </div>
+                    )
+                }
+                
+            </form>
         )
     }
-}
+}
+export default connect(
+    state => state.article,
+    actions
+)(Signin);
```
### 5.2 action-types.js
src/store/action-types.js
```js
-export const LOAD_USER='LOAD_USER';
+export const LOAD_USER='LOAD_USER';
+export const ADD_ARTICLE='ADD_ARTICLE';
+export const ADD_ARTICLE_SUCCESS='ADD_ARTICLE_SUCCESS';
+export const ADD_ARTICLE_ERROR='ADD_ARTICLE_ERROR';
```
### 5.3 saga.js
src/store/saga.js
```js
+function* addArticle(action) {
+    try {
+        debugger;
+        yield call(articleApi.add,action.article);
+        yield put(push('/'));
+    } catch (error) {
+        yield put({type: types.ADD_ARTICLE_ERROR,error});
+    }
+}
+
+function* watchAddArticle() {
+    yield takeEvery(types.ADD_ARTICLE,addArticle);
+}
+
 export default function* rootSaga() {
-    yield all([loginFlow(),watchLoadUser()]);
+    yield all([loginFlow(),watchLoadUser(),watchAddArticle()]);
}
```
### 5.4 actions/article.js
src/store/actions/article.js
```js
import * as types from '../action-types';
export default {
    add(article) {
        return {type: types.ADD_ARTICLE, article}
    }
}
```
### 5.5 article.js
src/store/api/article.js
```js
import {post} from '.';
async function add(body) {
    return post('/articles/add',body);    
}
export default {
    add
}
```
### 5.6 api/index.js
src/store/api/index.js
```js
-    return axios.post(BASE_URL+'/users/signin',body);
+    return axios.post(BASE_URL+url,body);
```
### 5.7 api/user.js
src/store/api/user.js
```js
-import {post} from './index';
+import {post} from '.';
 async function login(body) {
     return post('/users/signin',body);    
 }
```
### 5.8 reducers/article.js
src/store/reducers/article.js
```js
import * as types from '../action-types';
let initState = {};
export default function(state=initState, action) {
    switch(action.type) {
        case types.ADD_ARTICLE_ERROR:
            return {...state, user:null, error: action.error};
        default:
            return state;
    }
}
```
### 5.9 reducers/index.js
src/store/reducers/index.js
```js
+ import {routerReducer} from 'react-router-redux';
export default combineReducers({
  user,
+  article,
  router:routerReducer
});
```
## 参考仓库
-[react-router-redux](https://github.com/ReactTraining/react-router/tree/master/packages/react-router-redux)
-[jwt-frontend](https://gitee.com/zhufengpeixun/jwt-frontend)