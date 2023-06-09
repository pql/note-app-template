## 1.生成项目并安装模块
- [connected-react-router](https://github.com/supasate/connected-react-router)
```sh
create-react-app zhufeng-connect
cd zhufeng-connect
cnpm i react-router-dom redux  react-redux connected-react-router -S
```
## 2.跑通项目
### 2.1 src\index.js
src\index.js
```js
import React from 'react';
import ReactDOM from 'react-dom';
import {Route,Link} from 'react-router-dom';
import Home from './components/Home';
import Counter from './components/Counter';
import { ConnectedRouter } from 'connected-react-router'
import history from './history';
import store from './store';
import { Provider } from 'react-redux'
ReactDOM.render(
  <Provider store={store}>
    <ConnectedRouter history={history}>
    <>
      <Link  to="/">Home</Link>
      <Link to="/counter">Counter</Link>
      <Route exact={true} path="/" component={Home} />
      <Route path="/counter" component={Counter} />
    </>
    </ConnectedRouter>
  </Provider>
,document.getElementById('root'));
```
### 2.2 index.js
src\store\index.js
```js
import { applyMiddleware, createStore } from 'redux'
import { routerMiddleware } from 'connected-react-router'
import history from '../history';
import reducers from './reducers';
const store = applyMiddleware(routerMiddleware(history))(createStore)(reducers);
window.store = store;
export default  store;
```
### 2.3 history.js
src\history.js
```js
import { createHashHistory } from 'history'
let history = createHashHistory();
export default history;
```
### 2.4 reducers\index.js
src\store\reducers\index.js
```js
import { combineReducers } from 'redux'
import { connectRouter } from 'connected-react-router'
import counter from './counter';
import history from '../../history';
export default  combineReducers({
  router: connectRouter(history),
  counter
})
```
### 2.5 reducers\counter.js
src\store\reducers\counter.js
```js
import * as types from '../action-types';
export default function (state={number:0}, action) {
    switch (action.type) {
        case types.INCREMENT:
            return {number:state.number+1};
        case types.DECREMENT:
        return {number:state.number-1};
        default:
            return state;
    }
}
```
### 2.6 action-types.js
src\store\action-types.js
```js
export const INCREMENT = 'INCREMENT';
export const DECREMENT = 'DECREMENT';
```
### 2.7 actions\counter.js
src\store\actions\counter.js
```js
import  * as types from '../action-types';
import { push } from 'connected-react-router';
export default {
    increment(){
        return {type:types.INCREMENT}
    },
    decrement(){
        return {type:types.DECREMENT}
    },
    go(path){
        return push(path);
    }
}
```
### 2.8 Home.js
src\components\Home.js
```js
import React,{Component} from 'react';
export default class Home extends Component{
    render() {
        return (
            <div>Home</div>
        )
    }
}
```
### 2.9 Counter.js
src\components\Counter.js
```js
import React, { Component } from 'react'
import {connect} from 'react-redux';
import actions from '../store/actions/counter';
class Counter extends Component {
  render() {
    return (
      <>
          <p>{this.props.number}</p>
          <button onClick={this.props.increment}>+</button>    
          <button onClick={this.props.decrement}>-</button>    
          <button onClick={()=>this.props.go('/')}>Home</button>    
      </>
    )
  }
}
export default connect(
    state=>state.counter,
    actions
)(Counter);
```
## 3.实现connected-react-router
### 3.1 index.js
src\connected-react-router\index.js
```js
import push from './push';
import routerMiddleware from './routerMiddleware';
import ConnectedRouter from './ConnectedRouter';
import connectRouter from './connectRouter';
export {push,routerMiddleware,ConnectedRouter,connectRouter};
```
### 3.2 constants.js
src\connected-react-router\constants.js
```js
export const CALL_HISTORY_METHOD = '@@router/CALL_HISTORY_METHOD';
export const LOCATION_CHANGE = '@@router/LOCATION_CHANGE'
```
### 3.3 push.js
src\connected-react-router\push.js
```js
import {CALL_HISTORY_METHOD} from './constants';
export default function (path) {
    return {
        type: CALL_HISTORY_METHOD,
        payload: {
            method: 'push',
            path
        }
    };
}
```
### 3.4 routerMiddleware.js
src\connected-react-router\routerMiddleware.js
```js
import {CALL_HISTORY_METHOD} from './constants';
export default function routerMiddleware(history) {
    return function (store) {
      return function (next) {
        return function (action) {
          if (action.type !== CALL_HISTORY_METHOD) {
            return next(action);
          }
          let {method,path} = action.payload;
          history[method](path);
        };
      };
    };
  };
```
### 3.5 ConnectedRouter.js
src\connected-react-router\ConnectedRouter.js
```js
import React, { Component } from 'react'
import { Router } from 'react-router'
import { LOCATION_CHANGE } from './constants';
import { ReactReduxContext } from 'react-redux';
export default class ConnectedRouter extends Component {
  static contextType = ReactReduxContext
  componentDidMount() {
    this.unlisten = this.props.history.listen((location, action) => {
      this.context.store.dispatch({
        type: LOCATION_CHANGE,
        payload: {
          location,
          action
        }
      });
    });
  }
  componentWillUnmount() {
    this.unlisten();
  }
  render() {
    const { history, children } = this.props
    return (
      <Router history={history}>
        {children}
      </Router>
    )
  }
}
```
### 3.6 connectRouter.js
src\connected-react-router\connectRouter.js
```js
export default function (history) {
    let initState = {location:history.location,action:history.action};
    return function (state=initState, action) {
        if (action.type === LOCATION_CHANGE) {
            return {
                location: action.payload.location,
                action: action.payload.action
            }
        }
        return state;
    }
}
```
## 参考
- [zhufeng_connect](https://gitee.com/zhufengpeixun/zhufeng_connect)