## 1.React
### 1.1 JSX语法
- 什么是JSX
- 什么是React元素
- 什么是JSX表达式
- JSX属性
- 条件属性
- 列表渲染
### 1.2 React组件
- 组件分为函数组件和类组件
- 组件如何渲染
- 组件的属性和类型检查
- 组件的状态和状态更新
    - 状态对象是不可变的
    - 状态的更新可能是异步的
    - 状态可能会被合并
- 单向数据流
### 1.3 React事件
- 什么是合成事件
- 如何在事件上处理函数中绑定this
    - 公共属性（箭头函数）
    - 匿名函数
    - bind进行绑定
- 如何向事件中传递参数
### 1.4 组件生命周期
![](/public/images/react15.jpg)
![](/public/images/react16.jpg)
### 1.5 React高级
- 受控组件和非受控组件
- ref
    - ref值是字符串
    - ref值是函数
    - refs值是对象
    - 给类组件添加ref
    - 通过forwardRef给函数组件添加ref
- Portal
- context
- 异步组件和懒加载
    - Lazy Suspense
    - import
- shouldComponentUpdate
- immutable.js
- 展示组件和容器组件
- 高阶组件HOC
- render props
- Fragment
- Error Boundary
- React Hooks
    - useState
    - useReducer
    - useContext
    - useEffect
    - 自定义 Hooks
## 2.redux
### 2.1 redux核心
- reducer
- action
- store
- subscribe
- middleware
### 2.2 redux生态
- redux中间件
    - redux-thunk
    - redux-promise
    - redux-logger
    - redux-saga
    - redux-undo
    - redux-persist
    - redux-actions
    - reselect
- react-redux
    - Provider
    - connect
    - mapStateToProps
    - mapDispatchToProps
- react-router
    - react-router-dom
        - HashRouter VS BrowserRouter
        - history
        - Route
        - Link
        - Redirect
        - Switch
        - withRouter
        - Prompt
    - connect-react-router
## 4.react周边
- roadhog
- dva
- umi
- AntDesign
- AntDesignPro
## 5.原理
- 1. JSX的本质是什么?
- 2. 不同的类型的组件是如何渲染的？
- 3. setState是如何工作的
- 4. 虚拟DOM和DOM-DIFF算法
- 5. 什么是合成事件？
- 6. React Fiber是什么？它解决了什么问题？它是如何工作的？