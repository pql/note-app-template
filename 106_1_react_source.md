## 1. 什么是React?
- React 是一个用于构建用户界面的JavaScript库,核心专注于视图,目的实现组件化开发
## 2.搭建React开发环境
```sh
cnpm i create-react-app -g
create-react-app zhufeng2020react
cd zhufeng2020react
npm start
```
package.json
```json
  "scripts": {
+    "start": "cross-env DISABLE_NEW_JSX_TRANSFORM=true&&react-scripts start",
+    "build": "cross-env DISABLE_NEW_JSX_TRANSFORM=true&&react-scripts build",
+    "test": "cross-env DISABLE_NEW_JSX_TRANSFORM=true&&react-scripts test",
+    "eject": "cross-env DISABLE_NEW_JSX_TRANSFORM=true&&react-scripts eject"
  },
```
## 3.JSX
### 3.1 什么是JSX
- 是一种JS和HTML混合的语法,将组件的结构、数据甚至样式都聚合在一起定义组件
![](/public/images/1609297534961.png)
```js
let element = <h1>Hello</h1>;
console.log(element);
```
### 3.2 JSX属性
- 需要注意的是JSX并不是HTML,更像JavaScript
- 在JSX中属性不能包含关键字，像class需要写成className,for需要写成htmlFor,并且属性名需要采用驼峰命名法
```js
import React from 'react';
let element = <h1 className="title" style={{backGroundColor:'green', color: 'red' }}>hello<span>world</span></h1>;
console.log(element);
```
### 3.3 JSX实现
![](/public/images/1609321942740.png)
#### 3.3.1 src\constants.js
src\constants.js
```js
export let REACT_TEXT = 'REACT_TEXT';
```
#### 3.3.2 src\utils.js
src\utils.js
```js
import {REACT_TEXT} from './constants';

export function isText(obj){
    return typeof obj === 'string'||typeof obj === 'number';
}
export function wrapToVdom(obj){
    return isText(obj)?{type:REACT_TEXT,props:{content:obj}}:obj;
}
```
#### 3.3.3 src\react.js
src\react.js
```js
import {wrapToVdom} from './utils';
function createElement(type, config, children) {
    if (config) {
        delete config._owner;
        delete config._store;
        delete config.__self;
        delete config.__source;
        delete config.ref;
    }
    let props = { ...config };
    if (arguments.length > 3) {
        props.children = Array.prototype.slice.call(arguments,2).map(wrapToVdom);
    }else{
        props.children = wrapToVdom(children);
    }
    return {
        type,
        props
    };
}
const React = {
    createElement
};
export default React;
```
## 4.JSX渲染
### 4.1 src\index.js
```js
import React from './react';
import ReactDOM from './react-dom';
let element = <h1 className="title" style={{backgroundColor:'green', color: 'red' }}>hello<span>world</span></h1>;
ReactDOM.render(element,document.getElementById('root'));
```
### 4.2 react-dom.js
src\react-dom.js
```js
import { REACT_TEXT } from './constants';
function render(vdom,container){
   mount(vdom,container);
}
function  mount(vdom,container){
    if(!vdom)return;
    const dom = createDOM(vdom);
    container.appendChild(dom);
}
export function createDOM(vdom){
    let {type,props} = vdom;
    let dom;
    if (type === REACT_TEXT) {
        dom = document.createTextNode(props.content);
    }else{
        dom = document.createElement(type);//span div
    }
    if(props){
        updateProps(dom,{},props);//更新属性
        if(typeof props.children=='object' && props.children.type){
            render(props.children,dom);
        }else if(Array.isArray(props.children)){//是数组的话
            reconcileChildren(props.children,dom);
        }
    }
    vdom.dom=dom;
    return dom;
}
function updateProps(dom,oldProps={},newProps={}){
    for(let key in newProps){
        if(key === 'children'){continue;}
        if(key === 'style'){
            let style = newProps[key];
            for(let attr in style){
                dom.style[attr] = style[attr]
            }
        }else{
            dom[key]=newProps[key];
        }
    }
     if (oldProps) {
        for (let key in oldProps) {
            if (!newProps.hasOwnProperty(key)) {
                dom[key]='';
            }
        }
    }
}
function reconcileChildren(childrenVdom,parentDOM){
    childrenVdom.forEach((childVdom)=>render(childVdom,parentDOM));
}
const ReactDOM =  {
    render
};
export default ReactDOM;
```
## 5.函数式组件
- React元素不但可以是DOM标签，还可以是用户自定义的组件
- 当 React 元素为用户自定义组件时，它会将 JSX 所接收的属性（attributes）转换为单个对象传递给组件，这个对象被称之为 props
- 组件名称必须以大写字母开头
- 组件必须在使用的时候定义或引用它
- 组件的返回值只能有一个根元素
- 组件从概念上类似于 JavaScript 函数。它接受任意的入参(即 props)，并返回用于描述页面展示内容的 React 元素

![](/public/images/1609002319803.png)
### 5.1 src\index.js
```js
import React from './react';
import ReactDOM from './react-dom';
+function FunctionComponent(props){
+  return <div className="title" style={{backgroundColor:'green', color: 'red' }}><span>{props.name}</span>{props.children}</div>;
+}
+let element = <FunctionComponent name="hello">world</FunctionComponent>;
ReactDOM.render(element, document.getElementById('root'));
```
### 5.2 react-dom.js
src\react-dom.js
```js
import { REACT_TEXT } from './constants';
function render(vdom,container){
   mount(vdom,container);
}
function  mount(vdom,container){
    if(!vdom)return;
    const dom = createDOM(vdom);
    container.appendChild(dom);
}
export function createDOM(vdom){
    let {type,props} = vdom;
    let dom;
    if (type === REACT_TEXT) {
        dom = document.createTextNode(props.content);
+   }else if (typeof type === 'function') {
+       return mountFunctionComponent(vdom);
+   } else{
        dom = document.createElement(type);//span div
    }
    if(props){
        updateProps(dom,{},props);//更新属性
        if(typeof props.children=='object' && props.children.type){
            render(props.children,dom);
        }else if(Array.isArray(props.children)){//是数组的话
            reconcileChildren(props.children,dom);
        }
    }
    vdom.dom=dom;
    return dom;
}
+function mountFunctionComponent(vdom) {
+    const {type, props} = vdom;
+    const renderVdom = type(props);
+    vdom.oldRenderVdom=renderVdom;//这个在函数组件新的时候用
+    return createDOM(renderVdom);
+}
function updateProps(dom,oldProps={},newProps={}){
    for(let key in newProps){
        if(key === 'children'){continue;}
        if(key === 'style'){
            let style = newProps[key];
            for(let attr in style){
                dom.style[attr] = style[attr]
            }
        }else{
            dom[key]=newProps[key];
        }
    }
     if (oldProps) {
        for (let key in oldProps) {
            if (!newProps.hasOwnProperty(key)) {
                dom[key]='';
            }
        }
    }
}
function reconcileChildren(childrenVdom,parentDOM){
    childrenVdom.forEach((childVdom)=>render(childVdom,parentDOM));
}
const ReactDOM =  {
    render
};
export default ReactDOM;
```
## 6.类组件渲染
![](/public/images/1609002426742.png)
### 6.1 src\index.js
src\index.js
```js
import React from './react';
import ReactDOM from './react-dom';
+class ClassComponent extends React.Component{
+    render(){
+        return <div className="title" style={{backgroundColor:'green', color: 'red' }}><span>{this.props.name}</span>{this.props.children}</div>;
+    }
+}
+let element = <ClassComponent name="hello">world</ClassComponent>;
ReactDOM.render(element, document.getElementById('root'));
```
### 6.2 src\Component.js
src\Component.js
```js
class Component{
    static isReactComponent=true
    constructor(props){
        this.props = props;
    }
}
export default Component;
```
### 6.3 src\react.js
src\react.js
```js
import {wrapToVdom} from './utils';
+import Component from './Component';
function createElement(type, config, children) {
    if (config) {
        delete config._owner;
        delete config._store;
        delete config.__self;
        delete config.__source;
        delete config.ref;
    }
    let props = { ...config };
    if (arguments.length > 3) {
        props.children = Array.prototype.slice.call(arguments,2).map(wrapToVdom);
    }else{
        props.children = wrapToVdom(children);
    }
    return {
        type,
        props
    };
}
const React = {
    createElement,
+   Component
};
export default React;
```
### 6.4 src\react-dom.js
src\react-dom.js
```js
import { REACT_TEXT } from './constants';
function render(vdom,container){
   mount(vdom,container);
}
function  mount(vdom,container){
    if(!vdom)return;
    const dom = createDOM(vdom);
    container.appendChild(dom);
}
export function createDOM(vdom){
    let {type,props} = vdom;
    let dom;
    if (type === REACT_TEXT) {
        dom = document.createTextNode(props.content);
    }else if (typeof type === 'function') {
+       if (type.isReactComponent) {//说明这个type是一个类组件的虚拟DOM元素
+           return mountClassComponent(vdom);
+         } else {
+           return mountFunctionComponent(vdom);
+       }
    } else{
        dom = document.createElement(type);//span div
    }
    if(props){
        updateProps(dom,{},props);//更新属性
        if(typeof props.children=='object' && props.children.type){
            render(props.children,dom);
        }else if(Array.isArray(props.children)){//是数组的话
            reconcileChildren(props.children,dom);
        }
    }
    vdom.dom=dom;
    return dom;
}
+function mountClassComponent(vdom){
+    const {type, props} = vdom;
+    const classInstance = new type(props);
+    vdom.classInstance=classInstance;
+    const renderVdom = classInstance.render();
+    //classInstance在updateComponent时候使用，vdom在findDOM的时候使用
+    classInstance.oldRenderVdom=vdom.oldRenderVdom=renderVdom;
+    const dom = createDOM(renderVdom);
+    return dom;
+}

function mountFunctionComponent(vdom) {
    const {type, props} = vdom;
    const renderVdom = type(props);
    vdom.oldRenderVdom=renderVdom;
    return createDOM(renderVdom);
}
function updateProps(dom,oldProps={},newProps={}){
    for(let key in newProps){
        if(key === 'children'){continue;}
        if(key === 'style'){
            let style = newProps[key];
            for(let attr in style){
                dom.style[attr] = style[attr]
            }
        }else{
            dom[key]=newProps[key];
        }
    }
     if (oldProps) {
        for (let key in oldProps) {
            if (!newProps.hasOwnProperty(key)) {
                dom[key]='';
            }
        }
    }
}
function reconcileChildren(childrenVdom,parentDOM){
    childrenVdom.forEach((childVdom)=>render(childVdom,parentDOM));
}
const ReactDOM =  {
    render
};
export default ReactDOM;
```
## 7.组件更新
- 组件的数据来源有两个地方，分别是属性对象和状态对象
- 属性是父组件传递过来的
- 状态是自己内部的,改变状态唯一的方式就是setState
- 属性和状态的变化都会影响视图更新
![](/public/images/1609002644954.png)
### 7.1 src\index.js
```js
import React from './react';
import ReactDOM from './react-dom';
+class Counter extends React.Component {
+    constructor(props) {
+        super(props);
+        this.state = { number: 0 };
+    }
+    handleClick = () => {
+        this.setState({ number: this.state.number + 1 });
+        console.log(this.state);
+
+    }
+    render() {
+        return (
+            <div>
+                <p>number:{this.state.number}</p>
+                <button onClick={this.handleClick}>+</button>
+            </div>
+        )
+    }
+}
+ReactDOM.render(<Counter title="计数器" />, document.getElementById('root'));
```
### 7.2 src\Component.js
src\Component.js
```js
+import {createDOM,findDOM} from './react-dom';
class Component{
    static isReactComponent=true
    constructor(props){
        this.props = props;
+       this.state = {};
   }
+   setState(partialState) {
+       let {state} = this;
+       this.state = {...state,...partialState};
+       this.updateComponent();
+   }
+   updateComponent(newRenderVdom){
+    let newRenderVdom= this.render();
+    let oldDOM = findDOM(this.oldRenderVdom);
+    let newDOM = createDOM(newRenderVdom);
+    oldDOM.parentNode.replaceChild(newDOM, oldDOM);
+    this.oldRenderVdom=newRenderVdom;
+   }
}
export default Component;
```
### 7.3 src\react-dom.js
src\react-dom.js
```js
import { REACT_TEXT } from './constants';
function render(vdom,container){
   mount(vdom,container);
}
function  mount(vdom,container){
    if(!vdom)return;
    const dom = createDOM(vdom);
    container.appendChild(dom);
}
export function createDOM(vdom){
    let {type,props} = vdom;
    let dom;
    if (type === REACT_TEXT) {
        dom = document.createTextNode(props.content);
    }else if (typeof type === 'function') {
        if (type.isReactComponent) {//说明这个type是一个类组件的虚拟DOM元素
            return mountClassComponent(vdom);
          } else {
            return mountFunctionComponent(vdom);
        }
    } else{
        dom = document.createElement(type);//span div
    }
    if(props){
        updateProps(dom,{},props);//更新属性
        if(typeof props.children=='object' && props.children.type){
            render(props.children,dom);
        }else if(Array.isArray(props.children)){//是数组的话
            reconcileChildren(props.children,dom);
        }
    }
    vdom.dom=dom;
    return dom;
}
function mountClassComponent(vdom){
    const {type, props} = vdom;
    const classInstance = new type(props);
    vdom.classInstance=classInstance;
    const renderVdom = classInstance.render();
    classInstance.oldRenderVdom=vdom.oldRenderVdom=renderVdom;
    const dom = createDOM(renderVdom);
    return dom;
}

function mountFunctionComponent(vdom) {
    const {type, props} = vdom;
    const renderVdom = type(props);
    vdom.oldRenderVdom=renderVdom;
    return createDOM(renderVdom);
}
function updateProps(dom,oldProps={},newProps={}){
    for(let key in newProps){
        if(key === 'children'){continue;}
        if(key === 'style'){
            let style = newProps[key];
            for(let attr in style){
                dom.style[attr] = style[attr]
            }
+       }else if(key.startsWith('on')){
+            dom[key.toLocaleLowerCase()]=newProps[key];
        }else{
            dom[key]=newProps[key];
        }
    }
     if (oldProps) {
        for (let key in oldProps) {
            if (!newProps.hasOwnProperty(key)) {
                dom[key]='';
            }
        }
    }
}
function reconcileChildren(childrenVdom,parentDOM){
    childrenVdom.forEach((childVdom)=>render(childVdom,parentDOM));
}
+export function findDOM(vdom){
+    let {type}= vdom;
+    let dom;
+    if(typeof type === 'function'){//如果是组件的话
+        dom=findDOM(vdom.oldRenderVdom);
+    }else{///普通的字符串，那说明它是一个原生组件。dom指向真实DOM
+        dom=vdom.dom;
+    }
+    return dom;
+}
const ReactDOM =  {
    render
};
export default ReactDOM;
```
## 8.合成事件和批量更新
![](/public/images/1609302814714.png)
### 8.1 src\index.js
```js
import React from './react';
import ReactDOM from './react-dom';
class Counter extends React.Component {
    constructor(props) {
        super(props);
        this.state = { number: 0 };
    }
+   handleClick = () => {
+        this.setState({ number: this.state.number + 1 });
+        console.log(this.state);
+        this.setState({ number: this.state.number + 1 });
+        console.log(this.state);
+        setTimeout(()=>{
+            this.setState({ number: this.state.number + 1 });
+            console.log(this.state);
+        });
+   }
    render() {
        return (
            <div>
                <p>number:{this.state.number}</p>
                <button onClick={this.handleClick}>+</button>
            </div>
        )
    }
}
ReactDOM.render(<Counter title="计数器" />, document.getElementById('root'));
```
### 8.2 src\utils.js
src\utils.js
```js
import {REACT_TEXT} from './constants';

export function isText(obj){
    return typeof obj === 'string'||typeof obj === 'number';
}
export function wrapToVdom(obj){
    return isText(obj)?{type:REACT_TEXT,props:{content:obj}}:obj;
}
+export function isFunction(obj) {
+    return typeof obj === 'function';
+}
```
### 8.3 src\Component.js
src\Component.js
```js
import {createDOM,findDOM} from './react-dom';
+import {isFunction} from './utils';
+export let updateQueue = {
+    isBatchingUpdate: false,
+    updaters: new Set(),
+    add(updater){
+        updateQueue.updaters.add(updater);
+    },
+    batchUpdate() {
+        updateQueue.isBatchingUpdate = false;
+        for(let updater of updateQueue.updaters){
+            updater.emitUpdate();
+        }
+        updateQueue.updaters.clear();  
+    }
+};
+class Updater {
+    constructor(classInstance) {
+        this.classInstance = classInstance;
+        this.pendingStates = [];
+    }
+    addState(partialState) {
+        this.pendingStates.push(partialState);
+        updateQueue.isBatchingUpdate ? updateQueue.add(this) : this.emitUpdate();
+    }
+    emitUpdate(){
+        let { classInstance, pendingStates } = this;
+        if (pendingStates.length > 0) {
+            let nextState = this.getState();
+            classInstance.state = nextState;
+            classInstance.updateComponent();
+        }
+    }
+    getState() {
+        let { classInstance, pendingStates } = this;
+        let { state } = classInstance;
+        if (pendingStates.length) {
+            pendingStates.forEach(nextState => {
+                if (isFunction(nextState)) {
+                    nextState = nextState.call(classInstance, state);
+                }
+                state = { ...state, ...nextState };
+            });
+            pendingStates.length = 0;
+        }
+        return state;
+    }
+}
class Component{
    static isReactComponent=true
    constructor(props){
      this.props = props;
      this.state = {};
+     this.updater = new Updater(this);
   }
   setState(partialState) {
+    this.updater.addState(partialState);
   }
   updateComponent(){
    let newRenderVdom= this.render();
    let oldDOM = findDOM(this.oldRenderVdom);
    let newDOM = createDOM(newRenderVdom);
    oldDOM.parentNode.replaceChild(newDOM, oldDOM);
    this.oldRenderVdom=newRenderVdom;
   }
}
export default Component;
```
### 8.4 src\event.js
src\event.js
```js
import {updateQueue} from './Component';
/**
 * 给真实DOM添加事件处理函数 
 * 为什么要这么做?合成事件？为什么要做事件委托或者 事件代理
 * 1.做兼容处理 兼容不同的浏览器 不同的浏览器event是不一样的。处理浏览器的兼容 性
 * 2.可以在你写的事件处理函数之前和之后做一些事情，比如修改
 *   之前 updateQueue.isBatchingUpdate = true;
 *   之后 updateQueue.batchUpdate()
 * @param {*} dom 真实DOM
 * @param {*} eventType  事件类型
 * @param {*} listener  监听函数
 */
export function addEvent(dom,eventType,handleClick){
    //我要给dom绑定一个onclick事件回调函数,handleClick
    let store;
    if(dom.store){
        store = dom.store;
    }else{
        dom.store={};
        store=dom.store;
    }
   //let store = dom.store || (dom.store = {});
   store[eventType]=handleClick;//store.onclick = handleClick
   if(!document[eventType]){
    //事件委托，不管你给哪个DOM元素上绑事件，最后都统一代理到document上去了
    document[eventType]=dispatchEvent;//document.onclick=dispatchEvent;
   }
}
let syntheticEvent = {};
//原生的DOM
function dispatchEvent(event){
    let {target,type}= event;//事件源=button那个DOM元素 类型type=click
    let eventType = `on${type}`;//onclick
    updateQueue.isBatchingUpdate = true;//把队列设置为批量更新模式
    createSyntheticEvent(event);
    while(target){
        let {store}=target;
        let handleClick = store&&store[eventType];
        handleClick&&handleClick.call(target,syntheticEvent);
        target=target.parentNode;
    }
    for(let key in syntheticEvent){
        syntheticEvent[key]=null;
    }
    updateQueue.batchUpdate();//批量更新一下
}
function createSyntheticEvent(nativeEvent){
    for(let key in nativeEvent){
        syntheticEvent[key]=nativeEvent[key];
    }
}
```
### 8.5 src\react-dom.js
src\react-dom.js
```js
import { REACT_TEXT } from './constants';
+import { addEvent } from './event';
function render(vdom,container){
   mount(vdom,container);
}
function  mount(vdom,container){
    if(!vdom)return;
    const dom = createDOM(vdom);
    container.appendChild(dom);
}
export function createDOM(vdom){
    let {type,props} = vdom;
    let dom;
    if (type === REACT_TEXT) {
        dom = document.createTextNode(props.content);
    }else if (typeof type === 'function') {
        if (type.isReactComponent) {//说明这个type是一个类组件的虚拟DOM元素
            return mountClassComponent(vdom);
          } else {
            return mountFunctionComponent(vdom);
        }
    } else{
        dom = document.createElement(type);//span div
    }
    if(props){
        updateProps(dom,{},props);//更新属性
        if(typeof props.children=='object' && props.children.type){
            render(props.children,dom);
        }else if(Array.isArray(props.children)){//是数组的话
            reconcileChildren(props.children,dom);
        }
    }
    vdom.dom=dom;
    return dom;
}
function mountClassComponent(vdom){
    const {type, props} = vdom;
    const classInstance = new type(props);
    //这个在类组组件更新的时候用
    vdom.classInstance=classInstance;
    const renderVdom = classInstance.render();
    //classInstance在类组件更新的时候使用，vdom.在获取对应的真实的DOM的时候使用
    classInstance.oldRenderVdom=vdom.oldRenderVdom=renderVdom;
    const dom = createDOM(renderVdom);
    return dom;
}

function mountFunctionComponent(vdom) {
    const {type, props} = vdom;
    const renderVdom = type(props);
    vdom.oldRenderVdom=renderVdom;
    return createDOM(renderVdom);
}
function updateProps(dom,oldProps={},newProps={}){
    for(let key in newProps){
        if(key === 'children'){continue;}
        if(key === 'style'){
            let style = newProps[key];
            for(let attr in style){
                dom.style[attr] = style[attr]
            }
        }else if(key.startsWith('on')){
+           addEvent(dom,key.toLocaleLowerCase(),newProps[key]);
        }else{
            dom[key]=newProps[key];
        }
    }
     if (oldProps) {
        for (let key in oldProps) {
            if (!newProps.hasOwnProperty(key)) {
                dom[key]='';
            }
        }
    }
}
function reconcileChildren(childrenVdom,parentDOM){
    childrenVdom.forEach((childVdom)=>render(childVdom,parentDOM));
}
export function findDOM(vdom){
    let {type}= vdom;
    let dom;
    if(typeof type === 'function'){//如果是组件的话
        dom=findDOM(vdom.oldRenderVdom);
    }else{///普通的字符串，那说明它是一个原生组件。dom指向真实DOM
        dom=vdom.dom;
    }
    return dom;
}
const ReactDOM =  {
    render
};
export default ReactDOM;
```
## 9.基本生命周期
![](/public/images/react15.jpg)
![](/public/images/1609305099285.png)
### 9.1 src\index.js
```js
import React from './react';
import ReactDOM from './react-dom';
+class Counter extends React.Component{ // 他会比较两个状态相等就不会刷新视图 PureComponent是浅比较
+    static defaultProps = {
+        name: '珠峰架构'
+    };
+    constructor(props) {
+        super(props);
+        this.state = { number: 0 }
+        console.log('Counter 1.constructor')
+    }
+    componentWillMount() { // 取本地的数据 同步的方式：采用渲染之前获取数据，只渲染一次
+        console.log('Counter 2.componentWillMount');
+    }
+    componentDidMount() {
+        console.log('Counter 4.componentDidMount');
+    }
+    handleClick = () => {
+        this.setState({ number: this.state.number + 1 });
+    };
+    // react可以shouldComponentUpdate方法中优化 PureComponent 可以帮我们做这件事
+    shouldComponentUpdate(nextProps, nextState) { // 代表的是下一次的属性 和 下一次的状态
+        console.log('Counter 5.shouldComponentUpdate');
+        return nextState.number % 2 === 0;
+        // return nextState.number!==this.state.number; //如果此函数种返回了false 就不会调用render方+法了
+    } //不要随便用setState 可能会死循环
+    componentWillUpdate() {
+        console.log('Counter 6.componentWillUpdate');
+    }
+    componentDidUpdate() {
+        console.log('Counter 7.componentDidUpdate');
+    }
+    render() {
+        console.log('Counter 3.render');
+        return (
+            <div>
+                <p>{this.state.number}</p>
+                <button onClick={this.handleClick}>+</button>
+            </div>
+        )
+    }
+}
ReactDOM.render(<Counter />, document.getElementById('root'));
```
```sh
Counter 1.constructor
Counter 2.componentWillMount
Counter 3.render
Counter 4.componentDidMount
Counter 5.shouldComponentUpdate
Counter 5.shouldComponentUpdate
Counter 6.componentWillUpdate
Counter 3.render
Counter 7.componentDidUpdate
```
### 9.2 src\Component.js
src\Component.js
```js
import {createDOM,findDOM} from './react-dom';
import {isFunction} from './utils';
export let updateQueue = {
    isBatchingUpdate: false,
    updaters: new Set(),
    add(updater){
        updateQueue.updaters.add(updater);
    },
    batchUpdate() {
        updateQueue.isBatchingUpdate = false;
        for(let updater of updateQueue.updaters){
            updater.emitUpdate();
        }
        updateQueue.updaters.clear();  
    }
};
class Updater {
    constructor(classInstance) {
        this.classInstance = classInstance;
        this.pendingStates = [];
    }
    addState(partialState) {
        this.pendingStates.push(partialState);
        updateQueue.isBatchingUpdate ? updateQueue.add(this) : this.emitUpdate();
    }
+   emitUpdate(nextProps){
        let { classInstance, pendingStates } = this;
+       if (nextProps || pendingStates.length > 0) {
+            shouldUpdate(classInstance,nextProps,this.getState());
+        }
    }
    getState() {
        let { classInstance, pendingStates } = this;
        let { state } = classInstance;
        if (pendingStates.length) {
            pendingStates.forEach(nextState => {
                if (isFunction(nextState)) {
                    nextState = nextState.call(classInstance, state);
                }
                state = { ...state, ...nextState };
            });
            pendingStates.length = 0;
        }
        return state;
    }
}
+function shouldUpdate(classInstance,nextProps,nextState){
+    let willUpdate = true;
+    if(classInstance.shouldComponentUpdate
+        &&!classInstance.shouldComponentUpdate(nextProps,nextState)){
+            willUpdate = false;
+    }
+    if(willUpdate && classInstance.componentWillUpdate){
+        classInstance.componentWillUpdate();
+    }
+    if(nextProps){
+        classInstance.props = nextProps;
+    }
+    classInstance.state = nextState;
+    if(willUpdate)
+        classInstance.updateComponent();
+}
class Component{
    static isReactComponent=true
    constructor(props){
      this.props = props;
      this.state = {};
      this.updater = new Updater(this);
   }
   setState(partialState) {
    this.updater.addState(partialState);
   }
   updateComponent(){
    let newRenderVdom= this.render();
    let oldDOM = findDOM(this.oldRenderVdom);
    let newDOM = createDOM(newRenderVdom);
    oldDOM.parentNode.replaceChild(newDOM, oldDOM);
    this.oldRenderVdom=newRenderVdom;
+   if(this.componentDidUpdate)
+      this.componentDidUpdate(this.props,this.state);
+  }
}
export default Component;
```
### 9.3 src\react-dom.js
src\react-dom.js
```js
import { REACT_TEXT } from './constants';
import { addEvent } from './event';
function render(vdom,container){
   mount(vdom,container);
}
function  mount(vdom,container){
    if(!vdom)return;
    const dom = createDOM(vdom);
    container.appendChild(dom);
+   dom.componentDidMount&&dom.componentDidMount();
}
export function createDOM(vdom){
    let {type,props} = vdom;
    let dom;
    if (type === REACT_TEXT) {
        dom = document.createTextNode(props.content);
    }else if (typeof type === 'function') {
        if (type.isReactComponent) {//说明这个type是一个类组件的虚拟DOM元素
            return mountClassComponent(vdom);
          } else {
            return mountFunctionComponent(vdom);
        }
    } else{
        dom = document.createElement(type);//span div
    }
    if(props){
        updateProps(dom,{},props);//更新属性
        if(typeof props.children=='object' && props.children.type){
            render(props.children,dom);
        }else if(Array.isArray(props.children)){//是数组的话
            reconcileChildren(props.children,dom);
        }
    }
    vdom.dom=dom;
    return dom;
}
function mountClassComponent(vdom){
    const {type, props} = vdom;
    const classInstance = new type(props);
    vdom.classInstance=classInstance;
+   if(classInstance.componentWillMount)
+     classInstance.componentWillMount();
    const renderVdom = classInstance.render();
    classInstance.oldRenderVdom=vdom.oldRenderVdom=renderVdom;
    const dom = createDOM(renderVdom);
+   if(classInstance.componentDidMount)
+     dom.componentDidMount=classInstance.componentDidMount.bind(classInstance);
    return dom;
}

function mountFunctionComponent(vdom) {
    const {type, props} = vdom;
    const renderVdom = type(props);
    vdom.oldRenderVdom=renderVdom;
    return createDOM(renderVdom);
}
function updateProps(dom,oldProps={},newProps={}){
    for(let key in newProps){
        if(key === 'children'){continue;}
        if(key === 'style'){
            let style = newProps[key];
            for(let attr in style){
                dom.style[attr] = style[attr]
            }
        }else if(key.startsWith('on')){
            addEvent(dom,key.toLocaleLowerCase(),newProps[key]);
        }else{
            dom[key]=newProps[key];
        }
    }
     if (oldProps) {
        for (let key in oldProps) {
            if (!newProps.hasOwnProperty(key)) {
                dom[key]='';
            }
        }
    }
}
function reconcileChildren(childrenVdom,parentDOM){
    childrenVdom.forEach((childVdom)=>render(childVdom,parentDOM));
}
export function findDOM(vdom){
    let {type}= vdom;
    let dom;
    if(typeof type === 'function'){//如果是组件的话
        dom=findDOM(vdom.oldRenderVdom);
    }else{///普通的字符串，那说明它是一个原生组件。dom指向真实DOM
        dom=vdom.dom;
    }
    return dom;
}
const ReactDOM =  {
    render
};
export default ReactDOM;
```
## 10.完整生命周期
### 10.1 src\index.js
```js
import React from 'react';
import ReactDOM from 'react-dom';
class Counter extends React.Component{ // 他会比较两个状态相等就不会刷新视图 PureComponent是浅比较
    static defaultProps = {
        name: '珠峰架构'
    };
    constructor(props) {
        super(props);
        this.state = { number: 0 }
        console.log('Counter 1.constructor')
    }
    componentWillMount() {
        console.log('Counter 2.componentWillMount');
    }
    componentDidMount() {
        console.log('Counter 4.componentDidMount');
    }
    handleClick = () => {
        this.setState({ number: this.state.number + 1 });
    };
    shouldComponentUpdate(nextProps, nextState) {
        console.log('Counter 5.shouldComponentUpdate');
        return nextState.number % 2 === 0;
    }
    componentWillUpdate() {
        console.log('Counter 6.componentWillUpdate');
    }
    componentDidUpdate() {
        console.log('Counter 7.componentDidUpdate');
    }
    render() {
        console.log('Counter 3.render');
        return (
            <div id={`Counter${this.state.number}`}>
                <p>Counter:{this.state.number}</p>
                {this.state.number === 4 ? null : <ChildCounter count={this.state.number} />}
                <button onClick={this.handleClick}>+</button>
                <FunctionCounter count={this.state.number}/>
            </div>
        )
    }
}
class ChildCounter extends React.Component {
    componentWillUnmount() {
        console.log(' ChildCounter 5.componentWillUnmount')
    }
    componentWillMount() {
        console.log('ChildCounter 1.componentWillMount')
    }
    render() {
        console.log('ChildCounter 2.render');
        return (
           <div  id={`ChildCounter${this.props.count}`}>
              ChildCounter:{this.props.count}
           </div>
        )
    }
    componentDidMount() {
        console.log('ChildCounter 3.componentDidMount')
    }
    componentWillReceiveProps(newProps) {
        console.log('ChildCounter 4.componentWillReceiveProps')
    }
    componentWillUpdate() {
        console.log('ChildCounter 6.componentWillUpdate');
    }
    componentDidUpdate() {
        console.log('ChildCounter 7.componentDidUpdate');
    }
    shouldComponentUpdate(nextProps, nextState) {
        return nextProps.count % 3 === 0;
    }
}
let  FunctionCounter = (props)=><div id={`FunctionCounter${props.count}`}>FunctionCounter:{props.count}</div>;
ReactDOM.render(<Counter />, document.getElementById('root'));
```
```sh
Counter 1.constructor
Counter 2.componentWillMount
Counter 3.render
ChildCounter 1.componentWillMount
ChildCounter 2.render
ChildCounter 3.componentDidMount
Counter 4.componentDidMount
Counter 5.shouldComponentUpdate
Counter 5.shouldComponentUpdate
Counter 6.componentWillUpdate
Counter 3.render
ChildCounter 4.componentWillReceiveProps
Counter 7.componentDidUpdate
Counter 5.shouldComponentUpdate
Counter 5.shouldComponentUpdate
Counter 6.componentWillUpdate
Counter 3.render
 ChildCounter 5.componentWillUnmount
Counter 7.componentDidUpdate
Counter 5.shouldComponentUpdate
Counter 5.shouldComponentUpdate
Counter 6.componentWillUpdate
Counter 3.render
ChildCounter 1.componentWillMount
ChildCounter 2.render
ChildCounter 3.componentDidMount
Counter 7.componentDidUpdate
Counter 5.shouldComponentUpdate
Counter 5.shouldComponentUpdate
Counter 6.componentWillUpdate
Counter 3.render
ChildCounter 4.componentWillReceiveProps
Counter 7.componentDidUpdate
```
### 10.2 src\Component.js
src\Component.js
```js
+import {createDOM,findDOM,compareTwoVdom} from './react-dom';
import {isFunction} from './utils';
export let updateQueue = {
    isBatchingUpdate: false,
    updaters: new Set(),
    add(updater){
        updateQueue.updaters.add(updater);
    },
    batchUpdate() {
        updateQueue.isBatchingUpdate = false;
        for(let updater of updateQueue.updaters){
            updater.emitUpdate();
        }
        updateQueue.updaters.clear();  
    }
};
class Updater {
    constructor(classInstance) {
        this.classInstance = classInstance;
        this.pendingStates = [];
    }
    addState(partialState) {
        this.pendingStates.push(partialState);
        updateQueue.isBatchingUpdate ? updateQueue.add(this) : this.emitUpdate();
    }
    emitUpdate(nextProps){
        let { classInstance, pendingStates } = this;
        if (nextProps || pendingStates.length > 0) {
            shouldUpdate(classInstance,nextProps,this.getState());
        }
    }
    getState() {
        let { classInstance, pendingStates } = this;
        let { state } = classInstance;
        if (pendingStates.length) {
            pendingStates.forEach(nextState => {
                if (isFunction(nextState)) {
                    nextState = nextState.call(classInstance, state);
                }
                state = { ...state, ...nextState };
            });
            pendingStates.length = 0;
        }
        return state;
    }
}
function shouldUpdate(classInstance,nextProps,nextState){
    let willUpdate = true;
    if(classInstance.shouldComponentUpdate
        &&!classInstance.shouldComponentUpdate(nextProps,nextState)){
            willUpdate = false;
    }
    if(willUpdate && classInstance.componentWillUpdate){
        classInstance.componentWillUpdate();
    }
    if(nextProps){
        classInstance.props = nextProps;
    }
    classInstance.state = nextState;
    if(willUpdate)
        classInstance.updateComponent();
}
class Component{
    static isReactComponent=true
    constructor(props){
      this.props = props;
      this.state = {};
      this.updater = new Updater(this);
   }
   setState(partialState) {
    this.updater.addState(partialState);
   }
   updateComponent(){
    let newRenderVdom= this.render();
    let oldDOM = findDOM(this.oldRenderVdom);
+   compareTwoVdom(oldDOM.parentNode,this.oldRenderVdom,newRenderVdom);
    this.oldRenderVdom=newRenderVdom;
    if(this.componentDidUpdate)
       this.componentDidUpdate(this.props,this.state);
   }
}
export default Component;
```
### 10.3 src\react-dom.js
src\react-dom.js
```js
import { REACT_TEXT } from './constants';
import { addEvent } from './event';
function render(vdom,container){
   mount(vdom,container);
}
function  mount(vdom,container){
    if(!vdom)return;
    const dom = createDOM(vdom);
    container.appendChild(dom);
    dom.componentDidMount&&dom.componentDidMount();
}
export function createDOM(vdom){
    let {type,props} = vdom;
    let dom;
    if (type === REACT_TEXT) {
        dom = document.createTextNode(props.content);
    }else if (typeof type === 'function') {
        if (type.isReactComponent) {//说明这个type是一个类组件的虚拟DOM元素
            return mountClassComponent(vdom);
          } else {
            return mountFunctionComponent(vdom);
        }
    } else{
        dom = document.createElement(type);//span div
    }
    if(props){
        updateProps(dom,{},props);//更新属性
        if(typeof props.children=='object' && props.children.type){
            render(props.children,dom);
        }else if(Array.isArray(props.children)){//是数组的话
            reconcileChildren(props.children,dom);
        }
    }
    vdom.dom=dom;
    return dom;
}
function mountClassComponent(vdom){
    const {type, props} = vdom;
    const classInstance = new type(props);
    vdom.classInstance=classInstance;
    if(classInstance.componentWillMount)
       classInstance.componentWillMount();
    const renderVdom = classInstance.render();
    //classInstance在类组件更新的时候使用，vdom.在获取对应的真实的DOM的时候使用
    classInstance.oldRenderVdom=vdom.oldRenderVdom=renderVdom;
    const dom = createDOM(renderVdom);
    if(classInstance.componentDidMount)
      dom.componentDidMount=classInstance.componentDidMount.bind(classInstance);
    return dom;
}

function mountFunctionComponent(vdom) {
    const {type, props} = vdom;
    const renderVdom = type(props);
    vdom.oldRenderVdom=renderVdom;
    return createDOM(renderVdom);
}
function updateProps(dom,oldProps={},newProps={}){
    for(let key in newProps){
        if(key === 'children'){continue;}
        if(key === 'style'){
            let style = newProps[key];
            for(let attr in style){
                dom.style[attr] = style[attr]
            }
        }else if(key.startsWith('on')){
            addEvent(dom,key.toLocaleLowerCase(),newProps[key]);
        }else{
            dom[key]=newProps[key];
        }
    }
     if (oldProps) {
        for (let key in oldProps) {
            if (!newProps.hasOwnProperty(key)) {
                dom[key]='';
            }
        }
    }
}
function reconcileChildren(childrenVdom,parentDOM){
    childrenVdom.forEach((childVdom)=>render(childVdom,parentDOM));
}
export function findDOM(vdom){
    let {type}= vdom;
    let dom;
    if(typeof type === 'function'){//如果是组件的话
        dom=findDOM(vdom.oldRenderVdom);
    }else{///普通的字符串，那说明它是一个原生组件。dom指向真实DOM
        dom=vdom.dom;
    }
    return dom;
}
+export function compareTwoVdom(parentDOM, oldVdom, newVdom, nextDOM) {
+    //如果老的是null新的也是null
+    if (!oldVdom && !newVdom) {
+        return;
+        //如果老有,新没有,意味着此节点被删除了  
+    } else if (oldVdom && !newVdom) {
+        let currentDOM = findDOM(oldVdom);
+        if (currentDOM)
+            parentDOM.removeChild(currentDOM);
+        if (oldVdom.classInstance && oldVdom.classInstance.componentWillUnmount) {
+            oldVdom.classInstance.componentWillUnmount();
+        }
+        return;
+        //如果说老没有,新的有,新建DOM节点  
+    } else if (!oldVdom && newVdom) {
+        let newDOM = createDOM(newVdom);//创建一个新的真实DOM并且挂载到父节点DOM上
+        if (nextDOM) {//如果有下一个弟弟DOM的话,插到弟弟前面 p child-counter button
+            parentDOM.insertBefore(newDOM, nextDOM);
+        } else {
+            parentDOM.appendChild(newDOM);
+        }
+        if(newDOM.componentDidMount){
+            newDOM.componentDidMount();
+        }
+        return;
+        //如果类型不同，也不能复用了，也需要把老的替换新的
+    } else if (oldVdom && newVdom && (oldVdom.type !== newVdom.type)) {
+        let oldDOM = oldVdom.dom;
+        let newDOM = createDOM(newVdom);
+        oldDOM.parentNode.replaceChild(newDOM, oldDOM);
+        if (oldVdom.classInstance && oldVdom.classInstance.componentWillUnmount) {
+            oldVdom.classInstance.componentWillUnmount();
+        }
+        if(newDOM.componentDidMount){
+            newDOM.componentDidMount();
+        }
+        return;
+    } else {//新节点和老节点都有值
+        deepCompare(oldVdom, newVdom);
+        return;
+    }
+}
+/**
+ * 深度比较这二个虚拟DOM
+ * @param {*} oldVdom 老的虚拟DOM
+ * @param {*} newVdom 新的虚拟DOM
+ */
+function deepCompare(oldVdom,newVdom){
+    if(oldVdom.type === REACT_TEXT){//文件节点
+        let currentDOM = newVdom.dom = oldVdom.dom;//复用老的真实DOM节点
+        currentDOM.textContent = newVdom.props.content;//直接修改老的DOM节点的文件就可以了
+    }else if(typeof oldVdom.type === 'string'){//说明是个原生组件 div
+        let currentDOM = newVdom.dom = oldVdom.dom;//复用老的DIV的真实DOM div#counter
+        updateProps(currentDOM,oldVdom.props,newVdom.props);//更新自己的属性
+        //更新儿子们 只有原生的组件 div span才会去深度对比
+        updateChildren(currentDOM,oldVdom.props.children,newVdom.props.children);
+    }else if(typeof oldVdom.type === 'function'){
+        if(oldVdom.type.isReactComponent){
+            updateClassComponent(oldVdom,newVdom);//老的和新的都是类组件，进行类组件更新
+        }else{
+            updateFunctionComponent(oldVdom,newVdom);//老的和新的都是函数组件，进行函数数组更新
+        }
+    }
+}
+function updateFunctionComponent(oldVdom,newVdom){
+    let parentDOM=findDOM(oldVdom).parentNode;//div#counter
+    let {type,props}= newVdom;//FunctionCounter {count:2,children:[div]}
+    let oldRenderVdom=oldVdom.oldRenderVdom;//老的渲染出来的vdom div#counter-function>0
+    let newRenderVdom = type(props);//新的vdom div#counter-function>2
+    compareTwoVdom(parentDOM,oldRenderVdom,newRenderVdom);
+    newVdom.oldRenderVdom = newRenderVdom;
+}
+/**
+ * 如果老的虚拟DOM节点和新的虚拟DOM节点都是类组件的话，走这个更新逻辑
+ * @param {*} oldVdom 老的虚拟DOM节点
+ * @param {*} newVdom 新的虚拟DOM节点
+ */
+function updateClassComponent(oldVdom,newVdom){
+    let classInstance = newVdom.classInstance = oldVdom.classInstance;//类的实例需要复用。类的实例不管+更新多少只有一个
+    newVdom.oldRenderVdom=oldVdom.oldRenderVdom;//上一次的这个类组件的渲染出来的虚拟DOM
+    if(classInstance.componentWillReceiveProps){//组件将要接收到新的属性
+        classInstance.componentWillReceiveProps();
+    }
+    //触发组件的更新，要把新的属性传过来
+    classInstance.updater.emitUpdate(newVdom.props);
+}
+/**
+ * 深度比较它的儿子们
+ * @param {*} parentDOM 父DOM点
+ * @param {*} oldVChildren 老的儿子们 p2 ChildCounter button+
+ * @param {*} newVChildren 新的儿子们 p4 null button+
+ */
+function updateChildren(parentDOM,oldVChildren,newVChildren){
+    //因为children可能是对象，也可能是数组,为了方便按索引比较，全部格式化为数组
+    oldVChildren = Array.isArray(oldVChildren)?oldVChildren:[oldVChildren];
+    newVChildren = Array.isArray(newVChildren)?newVChildren:[newVChildren];
+    let maxLength = Math.max(oldVChildren.length,newVChildren.length);
+    for(let i=0;i<maxLength;i++){
+        //在儿子们里查找，找索引是大于当前索引的
+        let nextDOM = oldVChildren.find((item,index)=>index>i&&item&&item.dom);
+        compareTwoVdom(parentDOM,oldVChildren[i],newVChildren[i],nextDOM&&nextDOM.dom);
+    }
+}
const ReactDOM =  {
    render
};
export default ReactDOM;
```
## 11.新的生命周期
![](/public/images/react16.jpg)
### 11.1 getDerivedStateFromProps
#### 11.1.1 src\index.js
```js
import React from './react';
import ReactDOM from './react-dom';
class Counter extends React.Component{ // 他会比较两个状态相等就不会刷新视图 PureComponent是浅比较
    static defaultProps = {
        name: '珠峰架构'
    };
    constructor(props) {
        super(props);
        this.state = { number: 0 }
        console.log('Counter 1.constructor')
    }
    componentWillMount() {
        console.log('Counter 2.componentWillMount');
    }
    componentDidMount() {
        console.log('Counter 4.componentDidMount');
    }
    handleClick = () => {
        this.setState({ number: this.state.number + 1 });
    };
    shouldComponentUpdate(nextProps, nextState) {
        console.log('Counter 5.shouldComponentUpdate');
        return nextState.number % 2 === 0;
    }
    componentWillUpdate() {
        console.log('Counter 6.componentWillUpdate');
    }
    componentDidUpdate() {
        console.log('Counter 7.componentDidUpdate');
    }
    render() {
        console.log('Counter 3.render');
        return (
            <div id={`Counter${this.state.number}`}>
                <p>Counter:{this.state.number}</p>
                <ChildCounter count={this.state.number} />
                <button onClick={this.handleClick}>+</button>
            </div>
        )
    }
}
class ChildCounter extends React.Component {
    constructor(props) {
        super(props);
        this.state = { number: 0 };
    }
+   static getDerivedStateFromProps(nextProps, prevState) {
+       console.log('ChildCounter getDerivedStateFromProps');
+       const { count } = nextProps;
+       // 当传入的type发生变化的时候，更新state
+       if (count % 2 === 0) {
+           return { number: count * 2 };
+       } else {
+           return { number: count * 3 };
+       }
+   }
    render() {
        console.log('ChildCounter render', this.state)
        return (<div  id={`ChildCounter${this.props.count}`}>
            {this.state.number}
        </div>)
    }
}
ReactDOM.render(<Counter />, document.getElementById('root'));
```
#### 11.1.2 src\Component.js
src\Component.js
```js
import {createDOM,findDOM,compareTwoVdom} from './react-dom';
import {isFunction} from './utils';
export let updateQueue = {
    isBatchingUpdate: false,
    updaters: new Set(),
    add(updater){
        updateQueue.updaters.add(updater);
    },
    batchUpdate() {
        updateQueue.isBatchingUpdate = false;
        for(let updater of updateQueue.updaters){
            updater.emitUpdate();
        }
        updateQueue.updaters.clear();  
    }
};
class Updater {
    constructor(classInstance) {
        this.classInstance = classInstance;
        this.pendingStates = [];
    }
    addState(partialState) {
        this.pendingStates.push(partialState);
        updateQueue.isBatchingUpdate ? updateQueue.add(this) : this.emitUpdate();
    }
    emitUpdate(nextProps){
        let { classInstance, pendingStates } = this;
        if (nextProps || pendingStates.length > 0) {
+          shouldUpdate(classInstance,nextProps,this.getState(nextProps));
        }
    }
+   getState(nextProps) {
        let { classInstance, pendingStates } = this;
        let { state } = classInstance;
        if (pendingStates.length) {
            pendingStates.forEach(nextState => {
                if (isFunction(nextState)) {
                    nextState = nextState.call(classInstance, state);
                }
                state = { ...state, ...nextState };
            });
            pendingStates.length = 0;
        }
+       state = getDerivedStateFromProps(classInstance,nextProps,state);
        return state;
    }
}
+export function getDerivedStateFromProps(classInstance,nextProps,state){ 
+    if(classInstance.constructor.getDerivedStateFromProps){
+        let partialState = classInstance.constructor.getDerivedStateFromProps(nextProps,classInstance.state );
+        if(partialState){
+            state={...state,...partialState};
+        }
+    }
+    return state;
+}
function shouldUpdate(classInstance,nextProps,nextState){
    let willUpdate = true;
    if(classInstance.shouldComponentUpdate
        &&!classInstance.shouldComponentUpdate(nextProps,nextState)){
            willUpdate = false;
    }
    if(willUpdate && classInstance.componentWillUpdate){
        classInstance.componentWillUpdate();
    }
    if(nextProps){
        classInstance.props = nextProps;
    }
    classInstance.state = nextState;
    if(willUpdate)
        classInstance.updateComponent();
}
class Component{
    static isReactComponent=true
    constructor(props){
      this.props = props;
      this.state = {};
      this.updater = new Updater(this);
   }
   setState(partialState) {
    this.updater.addState(partialState);
   }
+  forceUpdate(){
+   this.state =  getDerivedStateFromProps(this,this.props,this.state);
+   this.updateComponent();
+  }
   updateComponent(){
    let newRenderVdom= this.render();
    let oldDOM = findDOM(this.oldRenderVdom);
    compareTwoVdom(oldDOM.parentNode,this.oldRenderVdom,newRenderVdom);
    this.oldRenderVdom=newRenderVdom;
    if(this.componentDidUpdate)
       this.componentDidUpdate(this.props,this.state);
   }
}
export default Component;
```
#### 11.1.3 src\react-dom.js
src\react-dom.js
```js
import { REACT_TEXT } from './constants';
import { addEvent } from './event';
+import {getDerivedStateFromProps} from './Component';
function mountClassComponent(vdom){
    const {type, props} = vdom;
    const classInstance = new type(props);
    vdom.classInstance=classInstance;
    if(classInstance.componentWillMount)
       classInstance.componentWillMount();
+    classInstance.state = getDerivedStateFromProps(classInstance,classInstance.props,classInstance.state)   
    const renderVdom = classInstance.render();
    classInstance.oldRenderVdom=vdom.oldRenderVdom=renderVdom;
    const dom = createDOM(renderVdom);
    if(classInstance.componentDidMount)
      dom.componentDidMount=classInstance.componentDidMount.bind(classInstance);
    return dom;
}
```
### 11.2 getSnapshotBeforeUpdate
- getSnapshotBeforeUpdate() 被调用于render之后，可以读取但无法使用DOM的时候,它使您的组件可以在可能更改之前从DOM捕获一些信息（例如滚动位置）
- 此生命周期返回的任何值都将作为参数传递给componentDidUpdate()
#### 11.2.1 src\index.js
```js
import React from 'react';
import ReactDOM from 'react-dom';
class Counter extends React.Component {
    ulRef = React.createRef()
    state = { list: [] }
    getSnapshotBeforeUpdate() {
        return this.ulRef.current.scrollHeight;
    }
    componentDidUpdate(prevProps, prevState, scrollHeight) {
        console.log('ul高度本次添加了', (this.ulRef.current.scrollHeight - scrollHeight) + 'px');
    }
    componentDidMount(){
        setInterval(() => {
            let list = this.state.list;
            this.setState({ list:[...list,list.length] });
        }, 1000);
    }
    render() {
        return (
            <div>
                <ul ref={this.ulRef}>
                    {
                        this.state.list.map((item, index) => <li key={index}>{item}</li>)
                    }
                </ul>
            </div>
        )
    }
}
ReactDOM.render(<Counter />, document.getElementById('root'));
```
#### 11.2.2 src\react.js
src\react.js
```js
import {wrapToVdom} from './utils';
import Component from './Component';
function createElement(type, config, children) {
+   let ref;
    if (config) {
        delete config._owner;
        delete config._store;
        delete config.__self;
        delete config.__source;
+       ref=config.ref;
        delete config.ref;
    }
    let props = { ...config };
    if (arguments.length > 3) {
        props.children = Array.prototype.slice.call(arguments,2).map(wrapToVdom);
    }else{
        props.children = wrapToVdom(children);
    }
    return {
        type,
+       ref,
        props
    };
}
+function createRef() {
+    return { current: null };
+}
const React = {
    createElement,
    Component,
+   createRef
};
export default React;
```
#### 11.2.3 src\Component.js
src\Component.js
```js
class Component{
   ...
   updateComponent(){
    let newRenderVdom= this.render();
    let oldDOM = findDOM(this.oldRenderVdom);
+   let extraArgs = this.getSnapshotBeforeUpdate && this.getSnapshotBeforeUpdate();
    compareTwoVdom(oldDOM.parentNode,this.oldRenderVdom,newRenderVdom);
    this.oldRenderVdom=newRenderVdom;
    if(this.componentDidUpdate)
+      this.componentDidUpdate(this.props,this.state,extraArgs);
   }
}
export default Component;
```
#### 11.2.4 src\react-dom.js
src\react-dom.js
```js
import { REACT_TEXT } from './constants';
import { addEvent } from './event';
import {getDerivedStateFromProps} from './Component';
function render(vdom,container){
   mount(vdom,container);
}
function  mount(vdom,container){
    if(!vdom)return;
    const dom = createDOM(vdom);
    container.appendChild(dom);
    dom.componentDidMount&&dom.componentDidMount();
}
export function createDOM(vdom){
+   let {type,props,ref} = vdom;
    let dom;
    if (type === REACT_TEXT) {
        dom = document.createTextNode(props.content);
    }else if (typeof type === 'function') {
        if (type.isReactComponent) {//说明这个type是一个类组件的虚拟DOM元素
            return mountClassComponent(vdom);
          } else {
            return mountFunctionComponent(vdom);
        }
    } else{
        dom = document.createElement(type);//span div
    }
    if(props){
        updateProps(dom,{},props);//更新属性
        if(typeof props.children=='object' && props.children.type){
            render(props.children,dom);
        }else if(Array.isArray(props.children)){//是数组的话
            reconcileChildren(props.children,dom);
        }
    }
    vdom.dom=dom;
+   if(ref)
+     ref.current = dom;
    return dom;
}
```
## 12.Context(上下文)
- 在某些场景下，你想在整个组件树中传递数据，但却不想手动地在每一层传递属性。你可以直接在 React 中使用强大的contextAPI解决上述问题
- 在一个典型的 React 应用中，数据是通过 props 属性自上而下（由父及子）进行传递的，但这种做法对于某些类型的属性而言是极其繁琐的（例如：地区偏好，UI 主题），这些属性是应用程序中许多组件都需要的。Context 提供了一种在组件之间共享此类值的方式，而不必显式地通过组件树的逐层传递 props

![](/public/images/contextapi.gif)

### 12.1 使用
```js
import React from './react';
import ReactDOM from './react-dom';
let PersonContext = React.createContext();
function getStyle(color){
  return {border:`5px solid ${color}`,padding:'5px',margin:'5px'}
}
class Person extends React.Component{
  state = {color:'red'}
  changeColor=(color)=>this.setState({color})
  render(){
    let contextValue = {name:'Person',color:this.state.color,changeColor:this.changeColor};
    return (
      <PersonContext.Provider value={contextValue}>
        <div style={{...getStyle(this.state.color),width:'200px'}}>
          人
          <Head/>
          <Body/>
        </div>
      </PersonContext.Provider>
    )
  }
}
class Head extends React.Component{
  static contextType = PersonContext;
  render(){
    return (
      <div  style={getStyle(this.context.color)}>
      头
      <Hair/>
    </div>

    )
  }
}
class Hair extends React.Component{
  static contextType = PersonContext;
  render(){
    console.log('Eye',this.context);
    return (
      <div  style={getStyle(this.context.color)}>
          头发
      </div>
    )
  }
}
class Body extends React.Component{
  static contextType = PersonContext;
  render(){
    return (
      <div  style={getStyle(this.context.color)}>
        四肢
        <Hand/>
      </div>
    )
  }
}
function Hand(){
  return (
    <PersonContext.Consumer>
      {
        contextValue=>(
          <div  style={getStyle(contextValue.color)}>
            手
            <button style={{color:'red'}} onClick={()=>contextValue.changeColor('red')}>变红</button>
            <button style={{color:'green'}} onClick={()=>contextValue.changeColor('green')}>变绿</button>
          </div>
        )
      }
    </PersonContext.Consumer>
  )
}  
ReactDOM.render(<Person/>,document.getElementById('root'));
```
### 12.2 src\react.js
src\react.js
```js
+function createContext(initialValue={}){
+    let context = {Provider,Consumer};
+    function Provider(props){
+      context._currentValue=context._currentValue||initialValue;
+      Object.assign(context._currentValue,props.value);
+      return props.children;
+    }
+    function Consumer(props){
+      return props.children(context._currentValue);
+    }
+    return context;
+}
const React = {
    createElement,
    Component,
    createRef,
+   createContext
};
export default React;
```
### 12.3 src\react-dom.js
src\react-dom.js
```js
function mountClassComponent(vdom){
    const {type, props} = vdom;
    const classInstance = new type(props);
    vdom.classInstance=classInstance;
+   if(type.contextType){
+       classInstance.context = type.contextType.Provider._value;
+   }
    if(classInstance.componentWillMount)
       classInstance.componentWillMount();
    classInstance.state = getDerivedStateFromProps(classInstance,classInstance.props,classInstance.state)   
    const renderVdom = classInstance.render();
    classInstance.oldRenderVdom=vdom.oldRenderVdom=renderVdom;
    const dom = createDOM(renderVdom);
    if(classInstance.componentDidMount)
      dom.componentDidMount=classInstance.componentDidMount.bind(classInstance);
    return dom;
}
```
## 13. 高阶组件 
- 高阶组件就是一个函数，传给它一个组件，它返回一个新的组件
- 高阶组件的作用其实就是为了组件之间的代码复用
```js
const NewComponent = higherOrderComponent(OldComponent)
```
### 13.1 cra支持装饰器
#### 13.1.1 安装
```sh
cnpm i react-app-rewired customize-cra @babel/plugin-proposal-decorators -D
```
#### 13.1.2 修改package.json
```json
  "scripts": {
    "start": "react-app-rewired start",
    "build": "react-app-rewired build",
    "test": "react-app-rewired test",
    "eject": "react-app-rewired eject"
  }
```
#### 13.1.3 config-overrides.js
```js
const {override,addBabelPlugin} = require('customize-cra');

module.exports = override(
  addBabelPlugin( [
    "@babel/plugin-proposal-decorators", { "legacy": true }
  ])
)
```
#### 13.1.4 jsconfig.json
```json
{
  "compilerOptions": {
     "experimentalDecorators": true
  }
}
```
### 13.2 属性代理
- 基于属性代理：操作组件的props
```js
import React from 'react';
import ReactDOM from 'react-dom';
const loading = message =>OldComponent =>{
    return class extends React.Component{
        render(){
            const state = {
                show:()=>{
                    let div = document.createElement('div');
                    div.innerHTML = `<p id="loading" style="position:absolute;top:100px;z-index:10;background-color:black">${message}</p>`;
                    document.body.appendChild(div);
                },
                hide:()=>{
                    document.getElementById('loading').remove();
                }
            }
            return  (
                <OldComponent {...this.props} {...state} {...{...this.props,...state}}/>
            )
        }
    }
}
@loading('正在加载中')
class Hello extends React.Component{
  render(){
     return <div>hello<button onClick={this.props.show}>show</button><button onClick={this.props.hide}>hide</button></div>;
  }
}
let LoadingHello  = loading('正在加载')(Hello);

ReactDOM.render(
    <LoadingHello/>, document.getElementById('root'));
```
### 13.3 反向继承
- 基于反向继承：拦截生命周期、state、渲染过程
#### 13.3.1 index.js
index.js
```js
import React from 'react';
import ReactDOM from 'react-dom';
class Button extends React.Component{
    state = {name:'张三'}
    componentDidMount(){
        console.log('Button componentDidMount');
    }
    render(){
        console.log('Button render');
        return <button name={this.state.name} title={this.props.title}/>
    }
}
const wrapper = OldComponent =>{
    return class NewComponent extends OldComponent{
        state = {number:0}
        componentWillMount(){
            console.log('WrapperButton componentWillMount');
             super.componentWillMount();
        }
        componentDidMount(){
            console.log('WrapperButton componentDidMount');
             super.componentDidMount();
        }
        handleClick = ()=>{
            this.setState({number:this.state.number+1});
        }
        render(){
            console.log('WrapperButton render');
            let renderElement = super.render();
            let newProps = {
                ...renderElement.props,
                ...this.state,
                onClick:this.handleClick
            }
            return  React.cloneElement(
                renderElement,
                newProps,
                this.state.number
            );
        }
    }
}
let WrappedButton = wrapper(Button);
ReactDOM.render(
    <WrappedButton title="标题"/>, document.getElementById('root'));
```
#### 13.3.2 src\react.js
src\react.js
```js
+function cloneElement(element,newProps,...newChildren){
+  let oldChildren = element.props&&element.props.children;
+  let children = [...(Array.isArray(oldChildren)?oldChildren:[oldChildren]),...newChildren]
+  .filter(item=>item!==undefined)
+  .map(wrapToVdom);
+  if(children.length===1) children=children[0];
+  let props = {...element.props,...newProps,children};
+  return {...element,props};
+}
const React = {
    createElement,
    Component,
    createRef,
    createContext,
+   cloneElement
};
export default React;
```
### 13.3 @修饰符
## 14. render props
- [render-props](https://zh-hans.reactjs.org/docs/render-props.html)
- render prop 是指一种在 React 组件之间使用一个值为函数的 prop 共享代码的简单技术
- 具有 render prop 的组件接受一个函数，该函数返回一个 React 元素并调用它而不是实现自己的渲染逻辑
- render prop 是一个用于告知组件需要渲染什么内容的函数 prop
- 这也是逻辑复用的一种方式
### 14.1 原生实现
```js
import React from 'react';
import ReactDOM from 'react-dom';
class MouseTracker extends React.Component {
    constructor(props) {
        super(props);
        this.state = { x: 0, y: 0 };
    }

    handleMouseMove = (event) => {
        this.setState({
            x: event.clientX,
            y: event.clientY
        });
    }

    render() {
        return (
            <div onMouseMove={this.handleMouseMove}>
                <h1>移动鼠标!</h1>
                <p>当前的鼠标位置是 ({this.state.x}, {this.state.y})</p>
            </div>
        );
    }
}
ReactDOM.render(<MouseTracker />, document.getElementById('root'));
```
### 14.2 children
- children是一个渲染的方法
```js
import React from './react';
import ReactDOM from './react-dom';

class MouseTracker extends React.Component {
    constructor(props) {
        super(props);
        this.state = { x: 0, y: 0 };
    }

    handleMouseMove = (event) => {
        this.setState({
            x: event.clientX,
            y: event.clientY
        });
    }

    render() {
        return (
            <div onMouseMove={this.handleMouseMove}>
                {this.props.children(this.state)}
            </div>
        );
    }
}
ReactDOM.render(<MouseTracker >
    {
        (props) => (
            <div>
                <h1>移动鼠标!</h1>
                <p>当前的鼠标位置是 ({props.x}, {props.y})</p>
            </div>
        )
    }
</MouseTracker >, document.getElementById('root'));
```
### 14.3 render属性
```js
import React from 'react';
import ReactDOM from 'react-dom';
class MouseTracker extends React.Component {
    constructor(props) {
        super(props);
        this.state = { x: 0, y: 0 };
    }

    handleMouseMove = (event) => {
        this.setState({
            x: event.clientX,
            y: event.clientY
        });
    }

    render() {
        return (
            <div onMouseMove={this.handleMouseMove}>
                {this.props.render(this.state)}
            </div>
        );
    }
}

ReactDOM.render(< MouseTracker render={params => (
    <>
        <h1>移动鼠标!</h1>
        <p>当前的鼠标位置是 ({params.x}, {params.y})</p>
    </>
)} />, document.getElementById('root'));
```
### 14.4 HOC
```js
import React from 'react';
import ReactDOM from 'react-dom';
function withTracker(OldComponent){
  return class MouseTracker extends React.Component{
    constructor(props){
        super(props);
        this.state = {x:0,y:0};
    }
    handleMouseMove = (event)=>{
        this.setState({
            x:event.clientX,
            y:event.clientY
        });
    }
    render(){
        return (
            <div onMouseMove = {this.handleMouseMove}>
               <OldComponent {...this.state}/>
            </div>
        )
    }
 }
}
//render
function Show(props){
    return (
        <React.Fragment>
          <h1>请移动鼠标</h1>
          <p>当前鼠标的位置是: x:{props.x} y:{props.y}</p>
        </React.Fragment>
    )
}
let HighShow = withTracker(Show);
ReactDOM.render(
    <HighShow/>, document.getElementById('root'));
```
## 15. shouldComponentUpdate
- 当一个组件的props或state变更，React会将最新返回的元素与之前渲染的元素进行对比，以此决定是否有必要更新真实的 DOM，当它们不相同时 React 会更新该 DOM
- 如果渲染的组件非常多时可以通过覆盖生命周期方法 shouldComponentUpdate 来进行优化
- shouldComponentUpdate 方法会在重新渲染前被触发。其默认实现是返回 true,如果组件不需要更新，可以在shouldComponentUpdate中返回 false 来跳过整个渲染过程。其包括该组件的 render 调用以及之后的操作
### 15.1 src\index.js
```js
import React from './react';
import ReactDOM from './react-dom';
const ChildCounter3 =  (props)=>{
  console.log('ChildCounter3 render');
  return (
    <div>
      ChildCounter3:{props.number}
    </div>
  )
}
const MemoChildCounter3 = React.memo(ChildCounter3);
class Counter extends React.Component {
  state = { number1: 0, number2: 0, number3: 0 }
  addNumber1 = () => {
    this.setState({ number1:this.state.number1+1 });
  }
  addNumber2 = () => {
    this.setState({ number2:this.state.number2+1 });
  }
  addNumber3 = () => {
    this.setState({ number3:this.state.number3+1 });
  }
  render() {
    console.log('Counter render');
    return (
      <div>
        <ChildCounter1 number={this.state.number1} />
        <ChildCounter2 number={this.state.number2} />
        <MemoChildCounter3 number={this.state.number3} />
        <button onClick={this.addNumber1}>ChildCounter1+</button>
        <button onClick={this.addNumber2}>ChildCounter2+</button>
        <button onClick={this.addNumber3}>ChildCounter3+</button>
      </div>
    )
  }
}
class ChildCounter1 extends React.PureComponent {
  render() {
    console.log('ChildCounter1 render');
    return (
      <div>
        ChildCounter1:{this.props.number}
      </div>
    )
  }
}
class ChildCounter2 extends React.PureComponent {
  render() {
    console.log('ChildCounter2 render');
    return (
      <div>
        ChildCounter2:{this.props.number}
      </div>
    )
  }
}

ReactDOM.render(<Counter />, document.getElementById('root'));
```
### 15.2 src\Component.js
src\Component.js
```js
+export class PureComponent extends Component{
+    //重写了此方法,只有状态或者 属性变化了才会进行更新，否则 不更新
+    shouldComponentUpdate(nextProps,nextState){
+        return !shallowEqual(this.props,nextProps)||!shallowEqual(this.state,nextState)
+    }
+}
+/**
+ * 用浅比较 obj1和obj2是否相等
+ * 只要内存地址一样，就认为是相等的，不一样就不相等
+ * @param {} obj1 
+ * @param {*} obj2 
+ */
+function shallowEqual(obj1,obj2){
+    if(obj1 === obj2)//如果引用地址是一样的，就相等.不关心属性变没变
+        return true;
+   //任何一方不是对象或者 不是null也不相等  null null  NaN!==NaN
+    if(typeof obj1 !== 'object' || obj1 ===null || typeof obj2 !== 'object' || obj2 ===null){
+        return false;
+    }    
+    let keys1 = Object.keys(obj1);
+    let keys2 = Object.keys(obj2);
+    if(keys1.length !== keys2.length){
+        return false;//属性的数量不一样，不相等
+    }
+    for(let key of keys1){
+        if(!obj2.hasOwnProperty(key) || obj1[key]!== obj2[key]){
+            return false;
+        }
+    }
+    return true;
+}
```
### 15.3 src\react.js
src\react.js
```js
import {wrapToVdom} from './utils';
+import {Component,PureComponent} from './Component';
function createElement(type, config, children) {
    let ref;
    if (config) {
        delete config._owner;
        delete config._store;
        delete config.__self;
        delete config.__source;
        ref=config.ref;
        delete config.ref;
    }
    let props = { ...config };
    if (arguments.length > 3) {
        props.children = Array.prototype.slice.call(arguments,2).map(wrapToVdom);
    }else{
        props.children = wrapToVdom(children);
    }
    return {
        type,
        ref,
        props
    };
}
function createRef() {
    return { current: null };
}
function createContext(initialValue={}){
    let context = {Provider,Consumer};
    function Provider(props){
      context._currentValue=context._currentValue||initialValue;
      Object.assign(context._currentValue,props.value);
      return props.children;
    }
    function Consumer(props){
      return props.children(context._currentValue);
    }
    return context;
}
function cloneElement(element,newProps,...newChildren){
  let oldChildren = element.props&&element.props.children;
  let children = [...(Array.isArray(oldChildren)?oldChildren:[oldChildren]),...newChildren]
  .filter(item=>item!==undefined)
  .map(wrapToVdom);
  if(children.length===1) children=children[0];
  let props = {...element.props,...newProps,children};
  return {...element,props};
}
+function memo(OldComponent){
+    return class extends React.PureComponent{
+      render(){
+        return <OldComponent {...this.props}/>
+      }
+}
+}
const React = {
    createElement,
    Component,
+   PureComponent,
    createRef,
    createContext,
    cloneElement,
+   memo
};
export default React;
```