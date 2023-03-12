## 1.dva
- 基于 `redux`、`redux-saga`和`react-router`的轻量级前端框架。（Inspired by elm and choo）
- dva是基于react+redux最佳实践上实现的封装方案，简化了redux和redux-saga使用上的诸多繁琐操作

## 2.数据流向
- 数据的改变发生通常是通过
    - 用户交互行为（用户点击按钮等）
    - 浏览器行为（如路由跳转等）触发的
- 当此类行为会改变数据的时候可以通过 dispatch 发起一个 action，如果是同步行为会直接通过 Reducers 改变 State, 如果是异步行为（副作用）会先触发Effects然后流向Reducers最终改变State
![](/public/images/PPrerEAKbIoDZYr.png)

## 3. 8个概念
### 3.1 State
- State 表示 Model的状态数据，通常表现为一个 javascript 对象（当然它可以是任何值）；
- 操作的时候每次都要当作不可变数据（immutable data）来对待，保证每次都是全新对象，没有引用关系，这样才能保证State的独立性，便于测试和追踪变化。
### 3.2 Action
- Action是一个普通javascript对象，它是改变State的唯一途径。
- 无论是从UI事件、网络回调，还是WebSocket等数据源所获得的数据，最终都会通过dispatch函数调用一个action,从而改变对应的数据。
- action 必须带有type属性指明具体的行为，其他字段可以自定义，
- 如果要发起一个action需要使用dispatch函数；
- 需要注意的是dispatch是在组件connect Models以后，通过 props 传入的。
### 3.3 dispatch
- dispatching function 是一个用于触发 action 的函数
- action 是改变 State 的唯一途径，但是它只描述了一个行为，而dispatch可以看作是触发这个行为的方式，而 Reducer 则是描述如何改变数据的。
- 在dva中，connect Model的组件通过 props 可以访问到 dispatch, 可以调用Model中的Reducer或者Effects,常见的形式如：
```js
dispatch({
    type: 'user/add', // 如果在 model 外调用，需要添加 namespace
    payload: {}, // 需要传递的信息
});
```
### 3.4 Reducer
- Reducer(也称为reducing function) 函数接受两个参数：之前已经累积运算的结果和当前要被累积的值，返回的是一个新的累积结果。该函数把一个集合归并合成一个单值。
- 在 dva 中，reducers 聚合积累的结果是当前 model 的 state 对象。
- 通过 actions 中传入的值，与当前 reducers 中的值进行运算获得新的值（也就是新的state）。
- 需要注意的是 Reducer 必须是纯函数，所以同样的输入必然得到同样的输出，它们不应该产生任何副作用。
- 并且，每一次的计算都应该使用 immutable data,这种特性简单理解就是每次操作都是返回一个全新的数据(独立，纯净)，所以热重载和事件旅行这些功能才能够使用。
### 3.5 Effect
- Effect 被称为副作用，在我们的应用中，最常见的就是异步操作。
- 它来自于函数编程的概念，之所以叫副作用是因为它使得我们的函数变的不纯，同样的输入不一定获得同样的输出。
- dva 为了控制副作用的操作，底层引入了redux-sagas 做异步流程控制，由于采用了 generator的相关概念，所以将异步转成同步写法，从而将effects转为纯函数。
### 3.6 Subscription
- Subscriptions 是一种从源获取数据的方法，它来自于 elm.
- Subscription 语义是订阅，用于订阅一个数据源，然后根据条件 dispatch 需要的 action
- 数据源可以是当前的时间、服务器的websocket连接、keyboard输入、geolocation变化、history路由变化等等。
### 3.7 Router
- 这里的路由通常指的是前端路由
- 由于我们的应用现在在通常是单页应用，所以需要前端代码来控制路由逻辑
- 通过浏览器提供的History API 可以监听浏览器url的变化，从而控制路由相关操作。
### 3.8 Route Components
- 在组件设计方法中，我们提到过Container Components,在 dva 中我们通常将其约束为 Route Components
- 因为在 dva 中我们通常以页面维度来设计 Container Components.
- 所以在 dva 中，通常需要 connect Model的组件都是 Route Components,组织在/routes/目录下,而/components/目录下则是纯组件（Presentational Components）。
## 4.初始化环境
```sh
create-react-app dva-app
cd dva-app
cnpm i dva keymaster -S
```
## 5.文件结构
官方推荐的：
```js
├── /mock/           # 数据mock的接口文件
├── /src/            # 项目源码目录
│ ├── /components/   # 项目组件
│ ├── /routes/       # 路由组件（页面维度）
│ ├── /models/       # 数据模型
│ ├── /services/     # 数据接口
│ ├── /utils/        # 工具函数
│ ├── route.js       # 路由配置
│ ├── index.js       # 入口文件
│ ├── index.less    
│ └── index.html    
├── package.json     # 定义依赖的pkg文件
└── proxy.config.js  # 数据mock配置文件
```
## 6.计数器
| 用法 | 说明 |
| --- | --- |
| app = dva(opts) | 创建应用，返回dva实例 |
| app.use(hooks) | 配置hooks，或者注册插件 |
| app.model(model) | 注册 model |
| app.router(({history, app}) => RouterConfig) | 注册路由表 |
| app.start(selector?) | 启动应用。selector 可选 |
```js
import React from 'react';
import dva, {connect} from 'dva';
import keymaster from 'keymaster';
import { Router, Route } from 'dva/router';
// dva react react-dom redux redux-saga react-router react-router-dom history
const app = dva();
// redux combineReducers reducer都有自己的标签
// combineReducers({counter: counterReducer})
// 总的状态树 state = {counter: 0, counter2: 0}
const delay = (millseconds) => {
    return new Promise(function(resolve, reject) => {
        setTimeout(function(){
            resolve();
        }, millseconds);
    });
}
app.model({
    namespace: 'counter',
    state: {number: 0},
    reducers: {
        // 接收老状态，返回新状态
        add(state) { // dispatch({type: 'add'});
            return {number: state.number+1};
        },
        minus(state) { // dispatch({type: 'minus'})
            return {number: state.number - 1};
        }
    },
    // 延时操作 调用接口 等待
    effects: {
        *async Add(action, {put, call}) { // redux-saga/effects {put,call}
            yield call(delay, 1000);
        }
    },
    subscriptions: {
        keyboard({dispatch}) {
            keymaster('space', () => {
                dispatch({type: 'add'});
            });
        },
        changeTitle({history}) {
            setTimeout(function() {
                history.listen(({pathname}) => {
                    document.title = pathname;
                });
            }, 1000);
        }
    }
});
app.model({
    namespace: 'counter2',
    state: {number: 0},
    reducers: { // 接收老状态，返回新状态
        add(state) { // dispatch({type: 'add'})
            return {number: state.number+1};
        },
        minus(state) { // dispatch({type: 'minus'})
            return {number: state.number-1};
        }
    }
});
const Counter = (props) => {
    return (
        <div>
            <p>{props.number}</p>
            <button onClick={() => props.dispatch({type: 'counter/add'})}>add</button>
            <button onClick={() => props.dispatch({type: 'counter/asyncAdd'})}>asyncAdd</button>
            <button onClick={() => props.dispatch({type: 'counter/minus'})}>-</button>
        </div>
    )
}
const Counter2 = (props)=>{
    return (
        <div>
            <p>{props.number}</p>
            <button onClick={()=>props.dispatch({type:'counter2/add'})}>+</button>
            <button onClick={()=>props.dispatch({type:'counter2/minus'})}>-</button>
        </div>
    )
}
// {counter1: {number: 0}, counter2: {number: 0}}
const ConnectedCounter = connect(
    state => state.counter
)(Counter);
const ConnectedCounter2 = connect(
    state => state.counter2
)(Counter2);
app.router(
    ({app, history}) => (
        <Router history={history}>
            <>
                <Route path="/counter1" component={ConnectedCounter} />
                <Route path="/counter2" component={ConnectedCounter2} />
            </>
        </Router>
    )
);
app.start('#root');
```
- namespace model的命名空间，同时也是他在全局 state 上的属性，只能用字符串
- state 初始值
- reducers 以 key/value 格式定义 reducer。用于处理同步操作，唯一可以修改 state 的地方。由action触发。
- effects 以 key/value 格式定义 effect.用于处理异步操作和业务逻辑，不直接修改 state.由action触发，可以触发action,可以和服务器交互，可以获取全局 state的数据等等。
- subscriptions 以key/value 格式定义 subscription. subscription是订阅，用于订阅一个数据源，然后根据需要 dispatch 相应的 action.在app.start()时被执行，数据源可以是当前的时间、服务器的websocket连接、keyboard输入、geolocation变化、history路由变化等等。
## 7.构建应用
```sh
$ npm run build
```
## 8.参考
- [dvajs](https://dvajs.com/)
- [dva-npm](https://www.npmjs.com/package/dva)
- [dva-github](https://github.com/dvajs/dva)
- [8个概念](https://github.com/dvajs/dva/blob/master/docs/Concepts_zh-CN.md)
- [redux](http://cn.redux.js.org/index.html)
- [redux-saga](https://redux-saga-in-chinese.js.org/)
- [generator](http://www.ruanyifeng.com/blog/2015/04/generator.html)