## 1.项目说明
### 1.1 项目目录说明
- src 项目源码目录
- index.js 入口文件
- containers 容器
- components 组件
- common 公共样式
### 1.2 存入 Redux
- 编写服务端接口
- 用fetch获取接口方法
- 通过action将获取的数据发送到reducer
- 通过reducer改变redux中的状态
### 1.3 样式
- [资源链接](http://at.alicdn.com/t/font_pgg5jafnob51m7vi.css)
- .icon-uilist
- .icon-xiaolian
- .icon-xiugaimima
- .icon-book
- .icon-fanhui
- .icon-guanbi
- .icon-guanyuwomen
- .icon-jianjie
- .icon-renturn
- .icon-xingqiu
- .icon-kecheng-copy
- .icon-common-changjianwenti-copy
- .icon-react
![](/public/images/iconfont.png)
## 2.首页导航
### 2.1 src\index.html
```html
+ <link rel="stylesheet" href="http://at.alicdn.com/t/font_pgg5jafnob51m7vi.css">
```
### 2.2 src\index.tsx
```tsx
import * as React from 'react';
import * as ReactDOM from 'react-dom';
import Home from './components/Home';
import Lesson from './components/Lesson';
import Profile from './components/Profile';
import App from './containers/App';
import { Provider } from 'react-redux';
import store from './store';
import {Route } from 'react-router-dom';
import { ConnectedRouter } from 'connected-react-router'
import history from './store/history';
ReactDOM.render((
    <Provider store={store}>
        <ConnectedRouter history={history}>
            <App>
                <Route exact path="/" component={Home} />
                <Route path="/lesson" component={Lesson} />
                <Route path="/profile" component={Profile}/>
            </App>
        </ConnectedRouter>
    </Provider>
),document.getElementById('root'));
```
### 2.3 webpack.config.js
```js
module: {
    rules: [
+            {
+                test: /\.less$/,
+                use:['style-loader','css-loader','less-loader']
+            }
    ]
},
```
### 2.4 src/common/common.less
src/common/common.less
```less
*{
    margin: 0;
    padding: 0;
}
ul,li{
    list-style: none;
}
a{
    text-decoration: none;
}
html,body,#root{
    width:100%;
    height:100%;
    overflow: hidden;
}
```
### 2.5 src/containers/App.tsx
src/containers/App.tsx
```tsx
import * as React from 'react';
import Tab from '../components/Tab';
import '../common/common.less'
export interface Props{
    children?: any,
}
export default class App extends React.Component<Props,{}> {
    render() {
        return (
            <React.Fragment>
                {this.props.children}
                <Tab/>
            </React.Fragment>
        );
    }
}
```
### 2.6 src/components/Tab/index.tsx
src/components/Tab/index.tsx
```tsx
import * as React from 'react';
import { NavLink } from 'react-router-dom';
import './index.less'
export interface Props{
    children?: any,
}
export default class Tab extends React.Component<Props> {
    render() {
        return (
            <nav className="footer">
                <NavLink exact to="/" activeClassName="active">
                    <i className="iconfont icon-xingqiu"></i>首页
                </NavLink>
                <NavLink exact to="/lesson" activeClassName="active">
                    <i className="iconfont icon-react"></i>我的课程
                </NavLink>
                <NavLink exact to="/profile" activeClassName="active">
                    <i className="iconfont icon-xiaolian"></i>个人中心
                </NavLink>
            </nav>
        );
    }
}
```
### 2.7 src/components/Tab/index.less
src/components/Tab/index.less
```less
.footer{
    position: fixed;
    width:100%;
    height:53px;
    bottom:0;
    display:flex;
    background: #FFF;
    border-top:1px solid #D5D4D5;
    a{
        display: flex;
        flex:1;
        justify-content: center;
        align-items: center;
        color:#B5B5B6;
        flex-direction: column;
        i{
            font-size:20px;
        }
        &.active{
            color:#198AE4;
        }
    }
}
```
### 2.8 src/components/Home/index.tsx
src/components/Home/index.tsx
```tsx
import * as React from 'react';
export interface Props{
    children: any,
}
export default class Home extends React.Component<Props> {
    render() {
        return (
            <div>
               Home
            </div>
        );
    }
}
```
### 2.9 src/components/Lesson/index.tsx
src/components/Lesson/index.tsx
```tsx
import * as React from 'react';
export interface Props{
    children: any,
}
export default class Lesson extends React.Component<Props> {
    render() {
        return (
            <div>
               Lesson
            </div>
        );
    }
}
```
### 2.10 src/components/Profile/index.tsx
src/components/Profile/index.tsx
```tsx
import * as React from 'react';
export interface Props{
    children: any,
}
export default class Profile extends React.Component<Props> {
    render() {
        return (
            <div>
               Profile
            </div>
        );
    }
}
```
## 3.首页头部动画
### 3.1 src/store/types/index.tsx
src/store/types/index.tsx
```tsx
+ export interface Props{
+     children?: any,
+ }
```
### 3.2 webpack.config.js
webpack.config.js
```js
+ {
+     test: /\.(jpg|png|gif)$/,
+     use:'url-loader'
+ }
```
### 3.3 src/components/HomeHeader/index.tsx
src/components/HomeHeader/index.tsx
```tsx
import * as React from 'react';
import './index.less'
import { CSSTransition,TransitionGroup } from 'react-transition-group';
declare function require(url: string): string;
const logo=require('../../common/images/logo.png');
import { Props } from '../../store/types';
export default class HomeHeader extends React.Component<Props> {
    state={
        showList:false
    }
    render() {
        return (
            <div className="home-header">
                <div className="header-menu">
                    <img src={logo} alt="logo" />
                    <div onClick={() => this.setState({showList:!this.state.showList})}>
                        {
                            this.state.showList?<i className="iconfont icon-guanbi"></i>:<i className="iconfont icon-uilist"></i>
                        }
                    </div>
                </div>
                <TransitionGroup>
                        {
                            this.state.showList&&<CSSTransition
                            timeout={500}
                            classNames="fade"
                              ><ul className="menu-list">
                                    <li data-category="1">React课程</li>
                                    <li data-category="2">Vue课程</li>
                                </ul></CSSTransition>
                        }
                </TransitionGroup>    
            </div>
        );
    }
}
```
### 3.4 src/components/HomeHeader/index.less
src/components/HomeHeader/index.less
```less
.home-header{
    background:#2A2A2A;
    height:56px;
    width:100%;
    position:fixed;
    top:0;
    left:0;
    .header-menu{
        display:flex;
        height:56px;
        justify-content: space-between;
        align-items: center;
        img{
            height:30px;
            width:105px;
            margin-left:10px;
        }
        i{
            color:#FFF;
            margin-right:10px;
        }
    }
    .menu-list{
        position: absolute;
        top:56px;
        left:0;
        width:100%;
        background-color:#000;
        li{
            border-top:1px solid #464646;
            height:43px;
            line-height:43px;
            text-align: center;
            color:#FFF;
            border-top:1px solid #464646;
            &.active{
                color:green;
            }
        }
    }
}
.fade-enter {
    opacity: 0.01;
}
.fade-enter-active {
    opacity: 1;
    transition: opacity 500ms ease-in;
}
.fade-exit {
    opacity: 1;
}
.fade-exit-active {
    opacity: 0.01;
    transition: opacity 500ms ease-in;
}
```
## 4. 当前分类存入redux
### 4.1 src/components/HomeHeader/index.tsx
src/components/HomeHeader/index.tsx
```tsx
import * as React from 'react';
import './index.less'
import { CSSTransition,TransitionGroup } from 'react-transition-group';
declare function require(url: string): string;
const logo=require('../../common/images/logo.png');
export interface Props{
    currentCategory?: any,
    setCurrentCategory?: any
}
export default class HomeHeader extends React.Component<Props> {
    state={
        showList:false
    }
    setCurrentCategory=(event:any) => {
        let category=event.target.dataset.category;
        this.props.setCurrentCategory(category);
    }
    render() {
        let {currentCategory}=this.props;
        return (
            <div className="home-header">
                <div className="header-menu">
                    <img src={logo} alt="logo" />
                    <div onClick={() => this.setState({showList:!this.state.showList})}>
                        {
                            this.state.showList?<i className="iconfont icon-guanbi"></i>:<i className="iconfont icon-uilist"></i>
                        }
                    </div>
                </div>
                <TransitionGroup>
                        {
                            this.state.showList&&<CSSTransition
                                  timeout={500}
                                  classNames="fade"
                        ><ul className="menu-list" onClick={this.setCurrentCategory}>
                                    <li data-category="react" className={currentCategory=='react'?'active':''}>React</li>
                                    <li data-category="vue"  className={currentCategory=='vue'?'active':''}>Vue</li>
                                </ul></CSSTransition>
                        }
                </TransitionGroup>    
            </div>
        );
    }
}
```
### 4.2 src/containers/Home/index.tsx
src/containers/Home/index.tsx
```tsx
import * as React from 'react';
import HomeHeader from '../../components/HomeHeader';
import actions from '../../store/actions/home';
import { Store} from '../../store/types';
import { connect } from 'react-redux';
export interface Props{
    currentCategory?: any,
    setCurrentCategory?: any
}
class Home extends React.Component<Props> {
    render() {
        return (
            <React.Fragment>
                <HomeHeader
                    currentCategory={this.props.currentCategory}
                    setCurrentCategory={this.props.setCurrentCategory}
                />
            </React.Fragment>
        );
    }
}

export default connect(
    (state:Store)=>state.home,
    actions
)(Home);
```
### 4.3 src/store/action-types.tsx
src/store/action-types.tsx
```tsx
+ export const SET_CURRENT_CATEGORY='SET_CURRENT_CATEGORY';
```
### 4.4 src/store/reducers/index.tsx
src/store/reducers/index.tsx
```tsx
import { combineReducers } from 'redux';
import history from '../history';
import home from './home';
import { connectRouter } from 'connected-react-router'
let reducers=combineReducers({
    home,
    router: connectRouter(history)
});
export default reducers;
```
### 4.5 src/store/types/index.tsx
src/store/types/index.tsx
```tsx
export interface Store{
    home: Home
}
export interface Home{
    currentCategory: string
}

export interface Props{
    children?: any,
}
```
### 4.6 src/store/actions/home.tsx
src/store/actions/home.tsx
```tsx
import {SET_CURRENT_CATEGORY} from '../action-types';
export interface setCurrentCategory{
    type: typeof SET_CURRENT_CATEGORY,
    currentCategory:string
}
export type Action=setCurrentCategory;
export default {
    setCurrentCategory(currentCategory:string): setCurrentCategory {
        return { type: SET_CURRENT_CATEGORY,currentCategory };
    }
}
```
### 4.7 src/store/reducers/home.tsx
src/store/reducers/home.tsx
```tsx
import * as types from '../action-types';
import { Home } from '../types';
import {Action} from '../actions/home';
export default function (state: Home={ currentCategory: '' },action: Action): Home {
    switch (action.type) {
        case types.SET_CURRENT_CATEGORY:
            return {...state,currentCategory:action.currentCategory};
        default:
            return state;
    }
}
```
## 5.轮播图
### 5.1 src/components/HomeSwiper/index.tsx
src/components/HomeSwiper/index.tsx
```tsx
import * as React from 'react';
import * as ReactSwipe from 'react-swipe';
import './index.less';
interface Props{
    sliders:any
}
export interface State{
    index:number
}
export default class Swiper extends React.Component<Props,State> {
    state={ index:0}
    render() {
        let options={
            continuous: true,
            callback: (index: number) => {
                this.setState({index});
            }
        }
        let swiper=(
            <ReactSwipe className="carousel" swipeOptions={options}>
                {this.props.sliders.map((item:string,index:number) => (
                    <div key={index}><img src={item}/></div>
                ))}
            </ReactSwipe>
        )
        return (
            <div className="home-sliders">
                {this.props.sliders.length>0? swiper:null}
                <div className="dots">
                    {
                        this.props.sliders.map((item:string,index:number) => (
                            <span key={index} className={`dot ${this.state.index ==index?'active':''}`}></span>
                        ))
                    }
                </div>
            </div>
        );
    }
}
```
### 5.2 src/components/HomeSwiper/index.less
src/components/HomeSwiper/index.less
```less
.home-sliders{
    position: relative;
    img{
        width:100%;
    }
    .dots{
        width:100%;
        position: absolute;
        bottom:7px;
        display: flex;
        justify-content: center;
        align-items: center;
        .dot{
            width:8px;
            height:8px;
            border-radius: 5px;
            background-color:#FFF;
            margin-left:5px;
            &.active{
                background-color:salmon;
            }
        }
    }
}
```
### 5.3 server/app.js
server/app.js
```js
let express=require('express');
let app=express();
app.use(function (req,res,next) {
    res.header('Access-Control-Allow-Methods','PUT,POST,GET,DELETE,OPTIONS');
    res.header('Access-Control-Allow-Origin','http://localhost:8081');
    res.header('Access-Control-Allow-Credentials','true');
    if (req.method === 'OPTIONS') {
        return res.sendStatus(200);
    }
    next();
});
app.listen(3000);
let sliders=require('./mock/sliders');
app.get('/sliders',function (req,res) {
    res.json(sliders);
});
```
### 5.4 src/api/index.tsx
src/api/index.tsx
```tsx
const API_HOST='http://localhost:3000';
export const get=(url:string) => {
    return fetch(API_HOST+url,{
        method: 'GET',
        credentials: 'include',//跨域携带cookie
        headers: {
            accept:'application/json'
        }
    }).then(res=>res.json());
}
export const post=(url:string,data:object) => {
    return fetch(API_HOST+url,{
        method: 'POST',
        body: JSON.stringify(data),
        headers: {
            'Content-Type': 'application/json',
            'Accept':'application/json'
        }
    });
}
```
### 5.5 src/api/home.tsx
src/api/home.tsx
```tsx
import {get} from './index';
export const getSliders=() => {
    return get('/sliders');
}
```
### 5.6 server/app.js
server/app.js
```js
let express=require('express');
let app=express();
app.use(function (req,res,next) {
    res.header('Access-Control-Allow-Methods','PUT,POST,GET,DELETE,OPTIONS');
    res.header('Access-Control-Allow-Origin','http://localhost:8081');
    res.header('Access-Control-Allow-Credentials','true');
    if (req.method === 'OPTIONS') {
        return res.sendStatus(200);
    }
    next();
});
app.listen(3000);
let sliders=require('./mock/sliders');
app.get('/sliders',function (req,res) {
    res.json(sliders);
});
```
### 5.7 server/mock/sliders.js
server/mock/sliders.js
```js
module.exports = [
    'http://www.zhufengpeixun.cn/themes/jianmo2/images/reactnative.png',
    'http://www.zhufengpeixun.cn/themes/jianmo2/images/react.png',
    'http://www.zhufengpeixun.cn/themes/jianmo2/images/vue.png',
    'http://www.zhufengpeixun.cn/themes/jianmo2/images/wechat.png',
    'http://www.zhufengpeixun.cn/themes/jianmo2/images/architect.jpg'
];
```
## 6.课程列表
### 6.1 server/app.js
```js
let lessons = require('./mock/lessons');
// http://getLessons/vue?offset=0&limit=5
app.get('/getLessons/:category',function(req,res){
   let category = req.params.category;
   let {offset,limit} = req.query;
   offset = isNaN(offset)?0:parseInt(offset);//偏移量 
   limit = isNaN(limit)?5:parseInt(limit); //每页条数
   let list = JSON.parse(JSON.stringify(lessons));
   if(category!='all'){
     list = list.filter(item=>item.category==category);
   }
   let total = list.length;
   //分页数据
   list = list.slice(offset,offset+limit);
   //list.forEach(item=>item.title= item.title+Math.random());
   setTimeout(function(){
    res.json({
        list,
        hasMore:total>offset+limit
    });
   },1000);
});
```
### 6.2 server/mock/lessons.js
server/mock/lessons.js
```js
module.exports = [
    {
      id: 1,
      title: '1.React全栈架构',
      "video":"http://7xil5b.com1.z0.glb.clouddn.com/zhufengpeixun.mp4",
      poster:"http://www.zhufengpeixun.cn/react/img/react.jpg",
      url: 'http://www.zhufengpeixun.cn/themes/jianmo2/images/react.png',
      price: '¥100.00元',
      category:'react'
    },
    {
      id: 2,
      title: '2.React全栈架构',
      video:"http://7xil5b.com1.z0.glb.clouddn.com/zhufengpeixun.mp4",
      poster:"http://www.zhufengpeixun.cn/react/img/react.jpg",
      url: 'http://www.zhufengpeixun.cn/themes/jianmo2/images/react.png',
      price: '¥200.00元',
      category:'react'
    },
    {
      id: 3,
      title: '3.React全栈架构',
      video:"http://7xil5b.com1.z0.glb.clouddn.com/zhufengpeixun.mp4",
      poster:"http://www.zhufengpeixun.cn/react/img/react.jpg",
      url: 'http://www.zhufengpeixun.cn/themes/jianmo2/images/react.png',
      price: '¥300.00元',
      category:'react'
    },
    {
      id: 4,
      title: '4.React全栈架构',
      video:"http://7xil5b.com1.z0.glb.clouddn.com/zhufengpeixun.mp4",
      poster:"http://www.zhufengpeixun.cn/react/img/react.jpg",
      url: 'http://www.zhufengpeixun.cn/themes/jianmo2/images/react.png',
      price: '¥400.00元',
      category:'react'
    },
    {
      id: 5,
      title: '5.React全栈架构',
      video:"http://7xil5b.com1.z0.glb.clouddn.com/zhufengpeixun.mp4",
      poster:"http://www.zhufengpeixun.cn/react/img/react.jpg",
      url: 'http://www.zhufengpeixun.cn/themes/jianmo2/images/react.png',
      price: '¥500.00元',
      category:'react'
    },
    {
      id: 6,
      title: '6.Vue从入门到项目实战',
      video:"http://7xil5b.com1.z0.glb.clouddn.com/zhufengpeixun.mp4",
      poster:"http://www.zhufengpeixun.cn/vue/img/vue.png",
      url: 'http://www.zhufengpeixun.cn/themes/jianmo2/images/vue.png',
      price: '¥100.00元',
      category:'vue'
    },
    {
      id: 7,
      title: '7.Vue从入门到项目实战',
      video:"http://7xil5b.com1.z0.glb.clouddn.com/zhufengpeixun.mp4",
      poster:"http://www.zhufengpeixun.cn/vue/img/vue.png",
      url: 'http://www.zhufengpeixun.cn/themes/jianmo2/images/vue.png',
      price: '¥200.00元',
      category:'vue'
    },
    {
      id: 8,
      title: '8.Vue从入门到项目实战',
      video:"http://7xil5b.com1.z0.glb.clouddn.com/zhufengpeixun.mp4",
      poster:"http://www.zhufengpeixun.cn/vue/img/vue.png",
      url: 'http://www.zhufengpeixun.cn/themes/jianmo2/images/vue.png',
      price: '¥300.00元',
      category:'vue'
    },
    {
      id: 9,
      title: '9.Vue从入门到项目实战',
      video:"http://7xil5b.com1.z0.glb.clouddn.com/zhufengpeixun.mp4",
      poster:"http://www.zhufengpeixun.cn/vue/img/vue.png",
      url: 'http://www.zhufengpeixun.cn/themes/jianmo2/images/vue.png',
      price: '¥400.00元',
      category:'vue'
    },
    {
      id: 10,
      title: '10.Vue从入门到项目实战',
      video:"http://7xil5b.com1.z0.glb.clouddn.com/zhufengpeixun.mp4",
      poster:"http://www.zhufengpeixun.cn/vue/img/vue.png",
      url: 'http://www.zhufengpeixun.cn/themes/jianmo2/images/vue.png',
      price: '¥500.00元',
      category:'vue'
    },
    {
      id: 11,
      title: '11.React全栈架构',
      "video":"http://7xil5b.com1.z0.glb.clouddn.com/zhufengpeixun.mp4",
      poster:"http://www.zhufengpeixun.cn/react/img/react.jpg",
      url: 'http://www.zhufengpeixun.cn/themes/jianmo2/images/react.png',
      price: '¥600.00元',
      category:'react'
    },
    {
      id: 12,
      title: '12.React全栈架构',
      video:"http://7xil5b.com1.z0.glb.clouddn.com/zhufengpeixun.mp4",
      poster:"http://www.zhufengpeixun.cn/react/img/react.jpg",
      url: 'http://www.zhufengpeixun.cn/themes/jianmo2/images/react.png',
      price: '¥700.00元',
      category:'react'
    },
    {
      id: 13,
      title: '13.React全栈架构',
      video:"http://7xil5b.com1.z0.glb.clouddn.com/zhufengpeixun.mp4",
      poster:"http://www.zhufengpeixun.cn/react/img/react.jpg",
      url: 'http://www.zhufengpeixun.cn/themes/jianmo2/images/react.png',
      price: '¥800.00元',
      category:'react'
    },
    {
      id: 14,
      title: '14.React全栈架构',
      video:"http://7xil5b.com1.z0.glb.clouddn.com/zhufengpeixun.mp4",
      poster:"http://www.zhufengpeixun.cn/react/img/react.jpg",
      url: 'http://www.zhufengpeixun.cn/themes/jianmo2/images/react.png',
      price: '¥900.00元',
      category:'react'
    },
    {
      id: 15,
      title: '15.React全栈架构',
      video:"http://7xil5b.com1.z0.glb.clouddn.com/zhufengpeixun.mp4",
      poster:"http://www.zhufengpeixun.cn/react/img/react.jpg",
      url: 'http://www.zhufengpeixun.cn/themes/jianmo2/images/react.png',
      price: '¥1000.00元',
      category:'react'
    },
    {
      id: 16,
      title: '16.Vue从入门到项目实战',
      video:"http://7xil5b.com1.z0.glb.clouddn.com/zhufengpeixun.mp4",
      poster:"http://www.zhufengpeixun.cn/vue/img/vue.png",
      url: 'http://www.zhufengpeixun.cn/themes/jianmo2/images/vue.png',
      price: '¥600.00元',
      category:'vue'
    },
    {
      id: 17,
      title: '17.Vue从入门到项目实战',
      video:"http://7xil5b.com1.z0.glb.clouddn.com/zhufengpeixun.mp4",
      poster:"http://www.zhufengpeixun.cn/vue/img/vue.png",
      url: 'http://www.zhufengpeixun.cn/themes/jianmo2/images/vue.png',
      price: '¥700.00元',
      category:'vue'
    },
    {
      id: 18,
      title: '18.Vue从入门到项目实战',
      video:"http://7xil5b.com1.z0.glb.clouddn.com/zhufengpeixun.mp4",
      poster:"http://www.zhufengpeixun.cn/vue/img/vue.png",
      url: 'http://www.zhufengpeixun.cn/themes/jianmo2/images/vue.png',
      price: '¥800.00元',
      category:'vue'
    },
    {
      id: 19,
      title: '19.Vue从入门到项目实战',
      video:"http://7xil5b.com1.z0.glb.clouddn.com/zhufengpeixun.mp4",
      poster:"http://www.zhufengpeixun.cn/vue/img/vue.png",
      url: 'http://www.zhufengpeixun.cn/themes/jianmo2/images/vue.png',
      price: '¥900.00元',
      category:'vue'
    },
    {
      id: 20,
      title: '20.Vue从入门到项目实战',
      video:"http://7xil5b.com1.z0.glb.clouddn.com/zhufengpeixun.mp4",
      poster:"http://www.zhufengpeixun.cn/vue/img/vue.png",
      url: 'http://www.zhufengpeixun.cn/themes/jianmo2/images/vue.png',
      price: '¥1000.00元',
      category:'vue'
    }
  ]
```
### 6.3 src/api/home.tsx
src/api/home.tsx
```tsx
import {get} from './index';
export const getSliders=() => {
    return get('/sliders');
}
export function getLessons(category:string,offset:number,limit:number){
    return get(`/getLessons/${category}?offset=${offset}&limit=${limit}`);
}
```
### 6.4 src/components/HomeHeader/index.tsx
src/components/HomeHeader/index.tsx
```tsx
import * as React from 'react';
import './index.less'
import { CSSTransition,TransitionGroup } from 'react-transition-group';
declare function require(url: string): string;
const logo=require('../../common/images/logo.png');
export interface Props{
    currentCategory?: any,
    setCurrentCategory?: any,
    fetchLessons:any
}
export default class HomeHeader extends React.Component<Props> {
    state={
        showList:false
    }
    setCurrentCategory=(event:any) => {
        let category=event.target.dataset.category;
        this.setState({ showList: false },() => {
            this.props.setCurrentCategory(category);
            this.props.fetchLessons();
        });

    }
    render() {
        let {currentCategory}=this.props;
        return (
            <div className="home-header">
                <div className="header-menu">
                    <img src={logo} alt="logo" />
                    <div onClick={() => this.setState({showList:!this.state.showList})}>
                        {
                            this.state.showList?<i className="iconfont icon-guanbi"></i>:<i className="iconfont icon-uilist"></i>
                        }
                    </div>
                </div>
                <TransitionGroup>
                        {
                            this.state.showList&&<CSSTransition
                                  timeout={500}
                                  classNames="fade"
                        ><ul className="menu-list" onClick={this.setCurrentCategory}>
                                    <li data-category="react" className={currentCategory=='react'?'active':''}>React</li>
                                    <li data-category="vue"  className={currentCategory=='vue'?'active':''}>Vue</li>
                                </ul></CSSTransition>
                        }
                </TransitionGroup>    
            </div>
        );
    }
}
```
### 6.5 src/containers/Home/index.tsx
src/containers/Home/index.tsx
```tsx
import * as React from 'react';
import HomeHeader from '../../components/HomeHeader';
import actions from '../../store/actions/home';
import { Store} from '../../store/types';
import { connect } from 'react-redux';
import HomeSwiper from '../../components/HomeSwiper';
import './index.less';
import HomeLessons from './HomeLessons';
export interface Props{
    currentCategory?: any,
    setCurrentCategory?: any,
    sliders: any,
    getSliders: any,
    fetchLessons: any,
    lessons:any
}
class Home extends React.Component<Props> {
    mainContent: any
    componentDidMount() {
        this.props.getSliders();
        this.props.fetchLessons();
    }
    render() {
        let {currentCategory,setCurrentCategory,fetchLessons,lessons}=this.props;
        return (
            <React.Fragment>
                <HomeHeader
                    currentCategory={currentCategory}
                    setCurrentCategory={setCurrentCategory}
                    fetchLessons={fetchLessons}
                />
                <div className="main-content" ref={ref => this.mainContent=ref}>
                    <HomeSwiper sliders={this.props.sliders} />
                    <HomeLessons
                    lessons={lessons}
                    fetchLessons={this.props.fetchLessons}
                  />
                </div>
            </React.Fragment>
        );
    }
}

export default connect(
    (state:Store)=>state.home,
    actions
)(Home);
```
### 6.6 src/store/action-types.tsx
src/store/action-types.tsx
```tsx
export const SET_CURRENT_CATEGORY='SET_CURRENT_CATEGORY';
export const SET_HOME_SLIDERS='SET_HOME_SLIDERS';

export const SET_LOADING_LESSONS = 'SET_LOADING_LESSONS';
export const SET_LESSONS='SET_LESSONS';
export const FETCH_LESSONS='FETCH_LESSONS';
```
### 6.7 src/store/actions/home.tsx
src/store/actions/home.tsx
```tsx
import * as types from '../action-types';
import {getSliders,getLessons} from '../../api/home';
export interface setCurrentCategory{
    type: typeof types.SET_CURRENT_CATEGORY,
    currentCategory:string
}
export type Action=any;
export default {
    setCurrentCategory(currentCategory:string): setCurrentCategory {
        return { type: types.SET_CURRENT_CATEGORY,currentCategory };
    },
    getSliders(){
        //这二个参数是redux-thunk 提供的 
        return function(dispatch:any,getState:any){
            getSliders().then((sliders:any)=>{
                dispatch({type:types.SET_HOME_SLIDERS,payload:sliders});
            });
        }
    },
    fetchLessons() {
        return (dispatch:any,getState:any) => {
            let {currentCategory,lessons: {hasMore,offset,limit,loading}}=getState().home;
            if (hasMore &&!loading) {
                dispatch({type:types.SET_LOADING_LESSONS,payload:true});
                getLessons(currentCategory,offset,limit).then(payload => {
                    dispatch({type: types.FETCH_LESSONS,payload});
                });
            }
        }
    }
}
```
### 6.8 src/store/reducers/home.tsx
src/store/reducers/home.tsx
```tsx
import * as types from '../action-types';
import { Home } from '../types';
import {Action} from '../actions/home';
export default function (state: Home={ currentCategory: 'all',sliders:[],lessons:{loading:false,hasMore:true,list:[],offset:0} },action: Action): Home {
    switch (action.type) {
        case types.SET_CURRENT_CATEGORY:
            return { ...state,currentCategory: action.currentCategory };
        case types.SET_HOME_SLIDERS:
            return { ...state,sliders: action.payload };
        case types.FETCH_LESSONS:
            return {
                ...state,
                lessons: {
                    ...state.lessons,
                    loading: false,
                    hasMore:action.payload.hasMore,
                    list: action.payload.list,
                    offset:action.payload.list.length
                }
            };
        default:
            return state;
    }
}
```
### 6.9 src/store/types/index.tsx
src/store/types/index.tsx
```tsx
export interface Store{
    home: Home
}
export interface Home{
    currentCategory: string,
    sliders: string[],
    lessons:any
}

export interface Props{
    children?: any,
}
```
### 6.10 src/components/Loading/index.tsx
src/components/Loading/index.tsx
```tsx
import * as React from 'react';
import './index.less'
export interface Props{
    children?: any,
}
export default class Loading extends React.Component<Props> {
    render() {
        return (
            <div className="loading">
                loading
            </div>
        );
    }
}
```
### 6.11 src/containers/Home/HomeLessons/index.tsx
src/containers/Home/HomeLessons/index.tsx
```tsx
import * as React from 'react';
import './index.less';
import Loading from '../../../components/Loading';
import { Link } from 'react-router-dom';
interface Props {
    lessons: any,
    fetchLessons:any
}
export default class HomeLessons extends React.Component<Props>{
    render(){
        let {list,hasMore,loading} = this.props.lessons;
        return (
            <div className="home-lessons">
                <div className="all-lessons">
                    <i className="iconfont icon-kecheng-copy"></i>
                    <span>全部课程</span>
                </div>
                {
                    list.length>0?list.map((item:any,index:number)=>(
                        <Link key={index} to={{pathname:`/detail/${item.id}`,state:item}}>
                            <div key={index} className="lesson">
                                <img src={item.poster}/>
                                <p>{item.title}</p>
                                <p>{item.price}</p>
                            </div>
                        </Link>
                    )):<div className="no-data">暂无数据</div>
                }
                {
                    loading?<Loading/>:(!hasMore&& <div className="loading-more" >到底了</div>)
                }
            </div>
        )
    }
}
```
### 6.12 src/containers/Home/HomeLessons/index.less
src/containers/Home/HomeLessons/index.less
```less
.home-lessons{
    width:100%;
    box-sizing: border-box;
    padding:7.5px;
    .all-lessons{
        margin:10px 0;
        span{
            margin-left:8px;
        }
    }
    .no-data{
        width:100%;
        height:35px;
        line-height: 35px;
        border-radius: 5px;
        color:#000;
        text-align:center;
    }
    .lesson{
        overflow: hidden;
        margin-bottom:17px;
        border-radius: 8px;
        box-shadow: 1px 1px 3px 2px #c1c1c1,-1px -1px 3px 2px #c1c1c1;
        img{
            width:100%;
            height:140px;
            border-radius: 8px 8px 0 0;
        }
        p{
            height:37px;
            text-align: center;
            line-height: 37px;
            &:nth-child(2){
                color:#b3b3b3;
            }
            &:nth-child(3){
                color:#ed3a3a;
            }
        }
    }
    .loading-more{
        width:100%;
        height:35px;
        line-height: 35px;
        border-radius: 5px;
        background-color:seagreen;
        color:#FFF;
        text-align:center;
    }
}
```
## 7.加载更多
### 7.1 server/app.js
server/app.js
```js
let express=require('express');
let app=express();
app.use(function (req,res,next) {
    res.header('Access-Control-Allow-Methods','PUT,POST,GET,DELETE,OPTIONS');
    res.header('Access-Control-Allow-Origin','http://localhost:8080');
    res.header('Access-Control-Allow-Credentials','true');
    if (req.method === 'OPTIONS') {
        return res.sendStatus(200);
    }
    next();
});
app.listen(3000);
let sliders=require('./mock/sliders');
app.get('/sliders',function (req,res) {
    res.json(sliders);
});
let lessons = require('./mock/lessons');
// http://getLessons/vue?offset=0&limit=5
app.get('/getLessons/:category',function(req,res){
   let category = req.params.category;
   let {offset,limit} = req.query;
   offset = isNaN(offset)?0:parseInt(offset);//偏移量 
   limit = isNaN(limit)?5:parseInt(limit); //每页条数
   let list = JSON.parse(JSON.stringify(lessons));
   if(category!='all'){
     list = list.filter(item=>item.category==category);
   }
   let total = list.length;
   //分页数据
   list = list.slice(offset,offset+limit);
   //list.forEach(item=>item.title= item.title+Math.random());
   setTimeout(function(){
    res.json({
        list,
        hasMore:total>offset+limit
    });
   },1000);
});
```
### 7.2 src/components/Loading/index.less
src/components/Loading/index.less
```less
.loading{
    width:100%;
    height:43px;
    line-height: 43px;
    text-align: center;
    img{
        height:43px;
        width:43px;
    }
}
```
### 7.3 src/components/Loading/index.tsx
src/components/Loading/index.tsx
```tsx
import * as React from 'react';
import './index.less';
declare function require(url: string): string;
let loading=require('../../images/loading.gif');
export interface Props{
    children?: any,
}
export default class Loading extends React.Component<Props> {
    render() {
        return (
            <div className="loading">
                 <img src={loading}/>
            </div>
        );
    }
}
```
### 7.4 src/containers/Home/HomeLessons/index.tsx
src/containers/Home/HomeLessons/index.tsx
```tsx
import * as React from 'react';
import './index.less';
import Loading from '../../../components/Loading';
import { Link } from 'react-router-dom';
interface Props {
    lessons: any,
    getLessons:any
}
export default class HomeLessons extends React.Component<Props>{
    render(){
        let { lessons: { list,hasMore,loading },getLessons} = this.props;
        return (
            <div className="home-lessons">
                <div className="all-lessons">
                    <i className="iconfont icon-kecheng-copy"></i>
                    <span>全部课程</span>
                </div>
                {
                    list.length>0?list.map((item:any,index:number)=>(
                        <Link key={index} to={{pathname:`/detail/${item.id}`,state:item}}>
                            <div key={index} className="lesson">
                                <img src={item.poster}/>
                                <p>{item.title}</p>
                                <p>{item.price}</p>
                            </div>
                        </Link>
                    )):<div className="no-data">暂无数据</div>
                }
                {
                    loading?<Loading/>:(hasMore?<div className="loading-more" onClick={getLessons} >加载更多</div>:<div className="loading-more">木有了</div>)
                }
            </div>
        )
    }
}
```
### 7.5 src/containers/Home/index.tsx
src/containers/Home/index.tsx
```tsx
import * as React from 'react';
import HomeHeader from './HomeHeader';
import actions from '../../store/actions/home';
import { Store} from '../../store/types';
import { connect } from 'react-redux';
import HomeSwiper from './HomeSwiper';
import HomeLessons from './HomeLessons';
import './index.less';

export interface Props{
    currentCategory?: any,
    setCurrentCategory?: any,
    sliders: any,
    getSliders: any,
    getLessons: any,
    lessons: any,
    refreshLessons:any
}
class Home extends React.Component<Props> {
    mainContent: any
    componentDidMount() {
        this.props.getSliders();
        this.props.getLessons();
    }
    render() {
        let {currentCategory,setCurrentCategory,getLessons,lessons,refreshLessons}=this.props;
        return (
            <React.Fragment>
                <HomeHeader
                    currentCategory={currentCategory}
                    setCurrentCategory={setCurrentCategory}
                    refreshLessons={refreshLessons}
                />
                <div className="main-content" ref={ref => this.mainContent=ref}>
                    <HomeSwiper sliders={this.props.sliders} />
                    <HomeLessons
                      lessons={lessons}
                      getLessons={getLessons}
                  />
                </div>
            </React.Fragment>
        );
    }
}

export default connect(
    (state:Store)=>state.home,
    actions
)(Home);
```
### 7.6 src/store/action-types.tsx
src/store/action-types.tsx
```tsx
export const SET_HOME_LESSONS_LOADING='SET_HOME_LESSONS_LOADING';
//追加一页数据
export const ADD_HOME_LESSONS='ADD_HOME_LESSONS';
//重置课程列表开始
export const REFRESH_HOME_LESSONS_LOADING='REFRESH_HOME_LESSONS_LOADING';
//重置课程列表
export const REFRESH_HOME_LESSONS='REFRESH_HOME_LESSONS';
```
### 7.7 src/store/actions/home.tsx
src/store/actions/home.tsx
```tsx
import * as types from '../action-types';
import {getSliders,getLessons} from '../../api/home';
export interface setCurrentCategory{
    type: typeof types.SET_CURRENT_CATEGORY,
    currentCategory:string
}
export type Action=any;
export default {
    setCurrentCategory(currentCategory:string): setCurrentCategory {
        return { type: types.SET_CURRENT_CATEGORY,currentCategory };
    },
    getSliders(){
        //这二个参数是redux-thunk 提供的 
        return function(dispatch:any,getState:any){
            getSliders().then((sliders:any)=>{
                dispatch({type:types.SET_HOME_SLIDERS,payload:sliders});
            });
        }
    },
    //获取下一页的课程
    getLessons() {
        return (dispatch:any,getState:any) => {
            let {currentCategory,lessons: {hasMore,offset,limit,loading}}=getState().home;
            if (hasMore &&!loading) {
                dispatch({type:types.SET_HOME_LESSONS_LOADING,payload:true});
                getLessons(currentCategory,offset,limit).then(payload => {
                    dispatch({type: types.ADD_HOME_LESSONS,payload});
                });
            }
        }
    },
    //重新获取第一页的数据 
    refreshLessons(){
        return function(dispatch:any,getState:any){
            let {currentCategory,lessons: {limit,loading}}=getState().home;
            if (!loading) {
                dispatch({type:types.REFRESH_HOME_LESSONS_LOADING});
                getLessons(currentCategory,0,limit).then(payload=>{
                    dispatch({type:types.REFRESH_HOME_LESSONS,payload});
                });
            }
        }
    }
}
```
### 7.8 src/store/reducers/home.tsx
src/store/reducers/home.tsx
```tsx
import * as types from '../action-types';
import { Home } from '../types';
import {Action} from '../actions/home';
export default function (state: Home={ currentCategory: 'all',sliders:[],lessons:{loading:false,hasMore:true,list:[],offset:0} },action: Action): Home {
    switch (action.type) {
        case types.SET_CURRENT_CATEGORY:
            return { ...state,currentCategory: action.currentCategory };
        case types.SET_HOME_SLIDERS:
            return { ...state,sliders: action.payload };
        case types.SET_HOME_LESSONS_LOADING:
            return {...state,lessons:{
                    ...state.lessons,
                    loading:action.payload
            }};
        case types.ADD_HOME_LESSONS://增加一页数据
            return {...state,lessons:{
                ...state.lessons,
                list:[...state.lessons.list,...action.payload.list],
                hasMore:action.payload.hasMore,
                offset:state.lessons.offset+action.payload.list.length,
                loading:false
            }
            }; 
        case types.REFRESH_HOME_LESSONS_LOADING:
            return {...state,lessons:{
                ...state.lessons,
                list:[],
                hasMore:true,
                offset:0,
                loading:true
            }}; 
        case types.REFRESH_HOME_LESSONS://刷新课程数据
            return {...state,lessons:{
                ...state.lessons,
                list:action.payload.list,
                hasMore:action.payload.hasMore,
                offset:action.payload.list.length,
                loading:false
            }}; 
        default:
            return state;
    }
}
```
### 7.9 src/containers/Home/HomeHeader/index.tsx
src/containers/Home/HomeHeader/index.tsx
```tsx
import * as React from 'react';
import './index.less'
import { CSSTransition,TransitionGroup } from 'react-transition-group';
declare function require(url: string): string;
const logo=require('../../../common/images/logo.png');
export interface Props{
    currentCategory?: any,
    setCurrentCategory?: any,
    refreshLessons:any
}
export default class HomeHeader extends React.Component<Props> {
    state={
        showList:false
    }
    setCurrentCategory=(event:any) => {
        let category=event.target.dataset.category;
        this.props.setCurrentCategory(category);
        this.setState({ showList: false },this.props.refreshLessons);

    }
    render() {
        let {currentCategory}=this.props;
        return (
            <div className="home-header">
                <div className="header-menu">
                    <img src={logo} alt="logo" />
                    <div onClick={() => this.setState({showList:!this.state.showList})}>
                        {
                            this.state.showList?<i className="iconfont icon-guanbi"></i>:<i className="iconfont icon-uilist"></i>
                        }
                    </div>
                </div>
                <TransitionGroup>
                        {
                            this.state.showList&&<CSSTransition
                                  timeout={500}
                                  classNames="fade"
                        ><ul className="menu-list" onClick={this.setCurrentCategory}>
                                    <li data-category="react" className={currentCategory=='react'?'active':''}>React</li>
                                    <li data-category="vue"  className={currentCategory=='vue'?'active':''}>Vue</li>
                                </ul></CSSTransition>
                        }
                </TransitionGroup>    
            </div>
        );
    }
}
```
### 7.10
```js
```
### 7.11 src/containers/Home/HomeSwiper/index.tsx
src/containers/Home/HomeSwiper/index.tsx
```tsx
import * as React from 'react';
import * as ReactSwipe from 'react-swipe';
import './index.less';
interface Props{
    sliders:any
}
export interface State{
    index:number
}
export default class Swiper extends React.Component<Props,State> {
    state={ index:0}
    render() {
        let options={
            continuous: true,
            auto: 3000,
            callback: (index: number) => {
                this.setState({index});
            }
        }
        let swiper=(
            <ReactSwipe className="carousel" swipeOptions={options}>
                {this.props.sliders.map((item:string,index:number) => (
                    <div key={index}><img src={item}/></div>
                ))}
            </ReactSwipe>
        )
        return (
            <div className="home-sliders">
                {this.props.sliders.length>0? swiper:null}
                <div className="dots">
                    {
                        this.props.sliders.map((item:string,index:number) => (
                            <span key={index} className={`dot ${this.state.index ==index?'active':''}`}></span>
                        ))
                    }
                </div>
            </div>
        );
    }
}
```
## 8. 上拉加载
### 8.1 src/containers/Home/index.tsx
src/containers/Home/index.tsx
```tsx
import * as React from 'react';
import HomeHeader from './HomeHeader';
import actions from '../../store/actions/home';
import { Store} from '../../store/types';
import { connect } from 'react-redux';
import HomeSwiper from './HomeSwiper';
import HomeLessons from './HomeLessons';
import './index.less';
import {loadMore,downReferesh} from '../../utils';
export interface Props{
    currentCategory?: any,
    setCurrentCategory?: any,
    sliders: any,
    getSliders: any,
    getLessons: any,
    lessons: any,
    refreshLessons:any
}
class Home extends React.Component<Props> {
    mainContent: any
    componentDidMount() {
        this.props.getSliders();
        this.props.getLessons();
        loadMore(this.mainContent,this.props.getLessons);
        downReferesh(this.mainContent,this.props.refreshLessons);
    }
    render() {
        let {currentCategory,setCurrentCategory,getLessons,lessons,refreshLessons}=this.props;
        return (
            <React.Fragment>
                <HomeHeader
                    currentCategory={currentCategory}
                    setCurrentCategory={setCurrentCategory}
                    refreshLessons={refreshLessons}
                />
                <div className="main-content" ref={ref => this.mainContent=ref}>
                    <HomeSwiper sliders={this.props.sliders} />
                    <HomeLessons
                      lessons={lessons}
                      getLessons={getLessons}
                  />
                </div>
            </React.Fragment>
        );
    }
}

export default connect(
    (state:Store)=>state.home,
    actions
)(Home);
```
### 8.2 src/utils.tsx
src/utils.tsx
```tsx
//ele 要实现此功能DOM对象 callback加载更多的方法
export function loadMore(element:any,callback:any){
    let timer:any;
    element.addEventListener('scroll',function(){
        timer&&clearTimeout(timer);
        timer = setTimeout(function(){
            let clientHeight = element.clientHeight;
            let scrollTop = element.scrollTop;
            let scrollHeight = element.scrollHeight;
            if(clientHeight+scrollTop+10 >= scrollHeight){
                callback();
            }
        },300);
    });
}
//下拉刷新
export function downReferesh(element:any,callback:any){
      let startY:number;//刚按下的时候初始纵坐标
      let distance:number;//下拉的距离
      let originTop=element.offsetTop;//最初的距离父级顶部的距离
   element.addEventListener('touchstart',function(event:any){
      if(element.offsetTop == originTop && element.scrollTop ==0){
        startY= event.touches[0].pageY;
        element.addEventListener('touchmove',touchMove);
        element.addEventListener('touchend',touchEnd);
      } 

      function touchMove(event:any){
        let pageY = event.touches[0].pageY;
        if(pageY > startY){//如果越来越大，表示下拉
            distance = pageY - startY;
            element.style.top = originTop+distance+'px';
        }else{
            element.removeEventListener('touchmove',touchMove);
            element.removeEventListener('touchend',touchEnd);
        }
      }
      function touchEnd(){
        element.removeEventListener('touchmove',touchMove);
        element.removeEventListener('touchend',touchEnd);
         let timer = setInterval(function(){
            if(distance<1){
                element.style.top = originTop+'px';//11.5
                clearInterval(timer);
            }else{
                element.style.top = originTop+(--distance)+'px';
            }
        },13);
        if(distance>30){
            callback();
        }
      }
   });
}
```
## 9.课程详情
### 9.1 src/index.tsx
```tsx
import * as React from 'react';
import * as ReactDOM from 'react-dom';
import Home from './containers/Home';
import Detail from './containers/Detail';
import Profile from './containers/Profile';
import App from './containers/App';
import { Provider } from 'react-redux';
import store from './store';
import {Route } from 'react-router-dom';
import { ConnectedRouter } from 'connected-react-router'
import history from './store/history';
ReactDOM.render((
    <Provider store={store}>
        <ConnectedRouter history={history}>
            <App>
                <Route exact path="/" component={Home} />
                <Route path="/detail/:id" component={Detail} />
                <Route path="/profile" component={Profile}/>
            </App>
        </ConnectedRouter>
    </Provider>
),document.getElementById('root'));
```
### 9.2 src/components/NavHeader/index.tsx
src/components/NavHeader/index.tsx
```tsx
import * as React from 'react';
import './index.less';
interface Props{
    history: any,
    title: string
}
export default class NavHeader extends React.Component<Props>{
    render(){
        return (
            <div className="navheader">
              <i 
              onClick={()=>this.props.history.goBack()}
              className="iconfont icon-fanhui"></i>
                {this.props.title}
            </div>
        )
    }
}
```
### 9.3 src/components/NavHeader/index.less
src/components/NavHeader/index.less
```less
.navheader{
    height:56px;
    background-color: #000;
    color:#FFF;
    line-height: 56px;
    text-align: center;
    position: fixed;
    top:0;
    left:0;
    width:100%;
    i{
        position: absolute;
        left:10px;
    }
}
```
### 9.4 src/containers/Detail/index.tsx
src/containers/Detail/index.tsx
```tsx
import * as React from 'react';
import NavHeader from '../../components/NavHeader';
import {Redirect} from 'react-router-dom';
import './index.less';
interface Props{
    location: any,
    history:any,
    match:any
}
export default class Detail extends React.Component<Props>{
    state = {
        lesson:this.props.location.state||{}
    }

    render(){
        let { lesson }=this.state;
        return (
            lesson?(
                <div className="lesson-detail">
                    <NavHeader title="课程详情" history={this.props.history} />
                    <img src={lesson.poster}/>
                    <p>{lesson.title}</p>
                    <p>{lesson.price}</p>
                </div>
            ):<Redirect to="/"/>

        )
    }
}
```
### 9.5 src/containers/Detail/index.less
src/containers/Detail/index.less
```less
.lesson-detail{
    padding-top:56px;
    img{
        height:167px;
        width:100%;
    }
    p{
       text-align: center;
       line-height:34px;
       height:34px;
       &:nth-child(2){
           color:#CCC;
       } 
       &:nth-child(2){
        color:#F00;
    } 
    }
}
```
## 10. 记住滚动条位置
### 10.1 src/containers/Home/index.tsx
src/containers/Home/index.tsx
```tsx
componentDidMount() {
    if (this.props.lessons.list.length == 0) {
        this.props.getSliders();
        this.props.getLessons();
    } else {
        this.mainContent.scrollTop=store.get('scrollTop');
    }
    loadMore(this.mainContent,this.props.getLessons);
    downReferesh(this.mainContent,this.props.refreshLessons);
}
componentWillUnmount() {
    store.set('scrollTop',this.mainContent.scrollTop);
}
```
### 10.2 src/utils.tsx
src/utils.tsx
```tsx
export const store =  {
    set(key:string,val:string) {
        sessionStorage.setItem(key,val);
    },
    get(key:string) {
        return sessionStorage.getItem(key);
    }
}
```
## 11.个人中心
### 11.1 src/containers/Profile/index.tsx
src/containers/Profile/index.tsx
```tsx
import * as React from 'react';
import './index.less'
import { Link } from 'react-router-dom';
declare function require(url: string): string;
let profile=require('../../images/profile.png');
export default class Profile extends React.Component{
    render() {
        return (
            <div className="profile">
                <div className="profile-bg">
                    <img src={profile}/>
                    <div className="login-btn">
                        <Link to="/login">登录</Link>
                    </div>
                </div>
            </div>
        )
    }
}
```
### 11.2 src/containers/Profile/index.less
src/containers/Profile/index.less
```less
.profile{
    .profile-bg{
        width:100%;
        height:223px;
        background-image: url(../../images/login_bg.png);
        background-size:contain;
        display:flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        img{
            width:60px;
            height:60px;
            border-radius:50%;
        } 
        .login-btn{
            width:60px;
            height:25px;
            border-radius: 10px;
            background:#FFF;
            color:#188AE4;
            line-height:25px;
            text-align:center;
            font-size:13px;
            font-weight: bold;
            margin-top:10px;
            a{
                text-decoration: none;
                &:link{
                    text-decoration: none;
                    color:#188AE4;
                }
            }
        }
    }
}
```
## 12.登录
### 12.1 src/index.tsx
src/index.tsx
```tsx
+ <Route path="/login" component={Login}/>
```
### 12.2 src/containers/Login/index.tsx
src/containers/Login/index.tsx
```tsx
import * as React from 'react';
import './index.less'
import {Link} from 'react-router-dom';
import NavHeader from '../../components/NavHeader';
declare function require(url: string): string;
let profile=require('../../images/profile.png');
interface Props{
    history:any
}
export default class Login extends React.Component<Props>{
    render() {
        return (
            <div className="login-panel">
                <NavHeader title="登录" history={this.props.history} />
                <div className="login-logo">
                  <img  src={profile} />
                </div>
                <input type="text" placeholder="手机号" />
                <input type="text" placeholder="密码" />
                <Link to="/reg">前往注册</Link>
                <button>登&nbsp;录</button>
            </div>
        )
    }
}
```
### 12.3 src/containers/Login/index.less
src/containers/Login/index.less
```less
.login-panel{
    padding:56px 17px;
    display:flex;
    flex-direction: column;
    position: absolute;
    top:0;
    left:0;
    right:0;
    bottom:0;
    background:#FFF;
    z-index: 20;

    .login-logo{
        display:flex;
        height:223px;
        width:100%;
        justify-content: center;
        align-items: center;
        img{
            height:62px;
            width:62px;
        }
    }
    input{
        width:100%;
        outline:none;
        border:1px solid #CCC;
        border-radius: 5px;
        height:42px;
        margin-bottom:20px;
        padding-left:10px;
        box-sizing: border-box;
    }
    a{
        color:#188AE4;
        margin-bottom:20px;
    }
    button{
        width:100%;
        background-color: #188AE4;
        color:#FFF;
        height:40px;
        line-height: 40px;
        text-align:center;
        font-size:16px;
        border-radius:5px;
    }
}
```
## 13.注册
### 13.1 src/index.tsx
```tsx
<Route path="/reg" component={Reg}/>
```
### 13.2 src/containers/Reg/index.tsx
src/containers/Reg/index.tsx
```tsx
import * as React from 'react';
import './index.less'
import {Link} from 'react-router-dom';
import NavHeader from '../../components/NavHeader';
declare function require(url: string): string;
let profile=require('../../images/profile.png');
interface Props{
    history:any
}
export default class Login extends React.Component<Props>{
    render() {
        return (
            <div className="login-panel">
                <NavHeader title="注册" history={this.props.history}/>
                <div className="login-logo">
                  <img  src={profile} />
                </div>
                <input type="text" placeholder="手机号" />
                <input type="text" placeholder="密码" />
                <Link to="/login">前往登录</Link>
                <button>注&nbsp;册</button>
            </div>
        )
    }
}
```
### 13.3 src/containers/Reg/index.less
src/containers/Reg/index.less
```less
.login-panel{
    padding:56px 17px;
    display:flex;
    flex-direction: column;
    position: absolute;
    top:0;
    left:0;
    right:0;
    bottom:0;
    background:#FFF;
    z-index: 20;

    .login-logo{
        display:flex;
        height:223px;
        width:100%;
        justify-content: center;
        align-items: center;
        img{
            height:62px;
            width:62px;
        }
    }
    input{
        width:100%;
        outline:none;
        border:1px solid #CCC;
        border-radius: 5px;
        height:42px;
        margin-bottom:20px;
        padding-left:10px;
        box-sizing: border-box;
    }
    a{
        color:#188AE4;
        margin-bottom:20px;
    }
    button{
        width:100%;
        background-color: #188AE4;
        color:#FFF;
        height:40px;
        line-height: 40px;
        text-align:center;
        font-size:16px;
        border-radius:5px;
    }
}
```
## 14. 实现会话
### 14.1 server/app.js
```js
let express=require('express');
let bodyParser = require('body-parser');
let session=require('express-session');
let app=express();
app.use(bodyParser.json());
app.use(function (req,res,next) {
    res.header('Access-Control-Allow-Methods','PUT,POST,GET,DELETE,OPTIONS');
    res.header('Access-Control-Allow-Headers','Content-Type');
    res.header('Access-Control-Allow-Origin','http://localhost:8080');
    res.header('Access-Control-Allow-Credentials','true');
    if (req.method === 'OPTIONS') {
        return res.sendStatus(200);
    }
    next();
});
app.use(session({
    resave:true,
    secret:'zfpx',
    saveUninitialized:true
}));
app.listen(3000);
let sliders=require('./mock/sliders');
app.get('/sliders',function (req,res) {
    res.json(sliders);
});
let lessons = require('./mock/lessons');
// http://getLessons/vue?offset=0&limit=5
app.get('/getLessons/:category',function(req,res){
   let category = req.params.category;
   let {offset,limit} = req.query;
   offset = isNaN(offset)?0:parseInt(offset);//偏移量 
   limit = isNaN(limit)?5:parseInt(limit); //每页条数
   let list = JSON.parse(JSON.stringify(lessons));
   if(category!='all'){
     list = list.filter(item=>item.category==category);
   }
   let total = list.length;
   //分页数据
   list = list.slice(offset,offset+limit);
   //list.forEach(item=>item.title= item.title+Math.random());
   setTimeout(function(){
    res.json({
        list,
        hasMore:total>offset+limit
    });
   },1000);
});

// 30 6次 5条 30 offset=30 
let users = [];
app.post('/api/reg',function(req,res){
   let body = req.body;
   users.push(body);
    res.json({
       code:0,
       success:'注册成功'
   })
});
//user username password
app.post('/api/login',function(req,res){
    let body = req.body;//{username,password}
    let user = users.find(item=>item.username == body.username && item.password == body.password);
    if(user){
        req.session.user = user;
        res.json({
            user,
            code:0,
            success:'登录成功'
        });
    }else{
        res.json({
            code:1,
            error:'登录失败'
        });
    }
});
app.get('/api/validate',function(req,res){
  let user = req.session.user;
  if(user){
    res.json({
        code:0,
        success:'此用户已经登录',
        user
    });
  }else{
      res.json({
        code:1,
        error:'此用户未登录',
    });
  }
});
```
### 14.2 src/api/index.tsx
src/api/index.tsx
```tsx
const API_HOST='http://localhost:3000';
export const get=(url:string) => {
    return fetch(API_HOST+url,{
        method: 'GET',
        credentials: 'include',//跨域携带cookie
        headers: {
            accept:'application/json'
        }
    }).then(res=>res.json());
}
export const post=(url:string,data:object) => {
    return fetch(API_HOST+url,{
        method: 'POST',
        body: JSON.stringify(data),
        headers: {
            'Content-Type': 'application/json',
            'Accept':'application/json'
        }
    }).then(res=>res.json());
}
```
### 14.3 src/containers/Login/index.tsx
src/containers/Login/index.tsx
```tsx
import * as React from 'react';
import './index.less'
import {Link} from 'react-router-dom';
import NavHeader from '../../components/NavHeader';
import { connect } from 'react-redux';
import { Store } from '../../store/types';
import actions from '../../store/actions/session';
declare function require(url: string): string;
let profile=require('../../images/profile.png');
interface Props{
    history: any,
    login:any
}
class Login extends React.Component<Props>{
    username:any
    password:any
    handleLogin = ()=>{
        let username = this.username.value;
        let password = this.password.value;
        this.props.login({username,password});
    }
    render() {
        return (
            <div className="login-panel">
                <NavHeader title="登录" history={this.props.history} />
                <div className="login-logo">
                  <img  src={profile} />
                </div>
                <input ref={input=>this.username=input} type="text" placeholder="手机号" />
                <input ref={input=>this.password=input} type="text" placeholder="密码" />
                <Link to="/reg">前往注册</Link>
                <button onClick={this.handleLogin}>登&nbsp;录</button>
            </div>
        )
    }
}

export default connect(
    (state:Store)=>state.session,actions
)(Login);
```
### 14.4 src/containers/Profile/index.tsx
src/containers/Profile/index.tsx
```tsx
import * as React from 'react';
import './index.less'
import { Link } from 'react-router-dom';
import { Store } from '../../store/types';
import { connect } from 'react-redux';
import actions from '../../store/actions/session';
declare function require(url: string): string;
let profile=require('../../images/profile.png');
interface Props{
    user:any
}
class Profile extends React.Component<Props>{
    render() {
        return (
            <div className="profile">
                <div className="profile-bg">
                    <img src={profile}/>
                    <div className="login-btn">
                    {this.props.user?this.props.user.username:<Link to="/login">登录</Link>}
                    </div>
                </div>
            </div>
        )
    }
}
export default connect(
    (state:Store)=>state.session,actions
)(Profile);
```
### 14.5 src/containers/Reg/index.tsx
src/containers/Reg/index.tsx
```tsx
import * as React from 'react';
import './index.less'
import {Link} from 'react-router-dom';
import NavHeader from '../../components/NavHeader';
import { connect } from 'react-redux';
import { Store } from '../../store/types';
import actions from '../../store/actions/session';
declare function require(url: string): string;
let profile=require('../../images/profile.png');
interface Props{
    history: any,
    reg:any
}
class Login extends React.Component<Props>{
    username: any
    password: any
    handleReg = ()=>{
        let username = this.username.value;
        let password = this.password.value;
        this.props.reg({username,password});
    }
    render() {
        return (
            <div className="login-panel">
                <NavHeader title="注册" history={this.props.history}/>
                <div className="login-logo">
                  <img  src={profile} />
                </div>
                <input ref={input=>this.username=input}  type="text" placeholder="手机号" />
                <input ref={input=>this.password=input} type="text" placeholder="密码" />
                <Link to="/login">前往登录</Link>
                <button   onClick={this.handleReg} >注&nbsp;册</button>
            </div>
        )
    }
}

export default connect(
    (state:Store)=>state.session,actions
)(Login);
```
### 14.6 src/store/action-types.tsx
src/store/action-types.tsx
```tsx
export const SET_CURRENT_CATEGORY='SET_CURRENT_CATEGORY';
export const SET_HOME_SLIDERS='SET_HOME_SLIDERS';

//开始加载下一页的数据
export const SET_HOME_LESSONS_LOADING='SET_HOME_LESSONS_LOADING';
//追加一页数据
export const ADD_HOME_LESSONS='ADD_HOME_LESSONS';
//重置课程列表开始
export const REFRESH_HOME_LESSONS_LOADING='REFRESH_HOME_LESSONS_LOADING';
//重置课程列表
export const REFRESH_HOME_LESSONS='REFRESH_HOME_LESSONS';


//注册
export const REG = 'REG';
//登录
export const LOGIN = 'LOGIN';
//退出
export const LOGOUT = 'LOGOUT';

//清空消息
export const CLEAR_MESSAGES = 'CLEAR_MESSAGES';
export const VALIDATE = 'VALIDATE';
```
### 14.7 src/store/reducers/index.tsx
src/store/reducers/index.tsx
```tsx
import { combineReducers } from 'redux';
import history from '../history';
import home from './home';
import session from './session';
import { connectRouter } from 'connected-react-router'
let reducers=combineReducers({
    home,
    session,
    router: connectRouter(history)
});
export default reducers;
```
### 14.8 src/store/types/index.tsx
src/store/types/index.tsx
```tsx
export interface Store{
    home: Home,
    session:Session
}
export interface Home{
    currentCategory: string,
    sliders: string[],
    lessons:any
}
export interface Session{
    user?: any,
    error: any,
    success:any
}

export interface Props{
    children?: any,
}
```
### 14.9 src/api/session.tsx
```tsx
import {get,post} from './index';
//注册
export function reg(user:any){
  return post('/api/reg',user);//{username,password}
}
//登录
export function login(user:any){
  return post('/api/login',user);//{username,password}
}
//退出
export function logout(){
  return get('/api/logout');
}

export function validate(){
  return get('/api/validate');
}
```
### 14.10 src/store/actions/session.tsx
src/store/actions/session.tsx
```tsx
import * as types from '../action-types';
import { reg,login,logout,validate } from '../../api/session';
import {push} from 'connected-react-router';
export type Action=any;
interface Res{
    code:any,success:any,error:any
}
export default {
    reg(user:any) {
        return function (dispatch:any,getState:any) {
            reg(user).then((result) => {
                let { code,success,error }=result;
                dispatch({
                    type: types.REG,
                    payload: { success,error }
                });
                if (code==0) {//code=0表示成功 成功后跳到登录页
                    dispatch(push('/login'));
                }
            })
        }
    },
    login(user:any) {
        return function (dispatch:any,getState:any) {
            login(user).then(result => {
                let { code,success,error,user }=result;
                dispatch({
                    type: types.LOGIN,
                    payload: { success,error,user }
                });
                if (code==0) {
                    dispatch(push('/profile'));
                }
            })
        }
    },
    logout() {
        return function (dispatch:any,getState:any) {
            logout().then(result => {
                let { code,success,error }=result;
                dispatch({
                    type: types.LOGOUT,
                    payload: { success,error }
                });
                dispatch(push('/login'));
            });
        }
    },
    clearMessages() {
        return {
            type: types.CLEAR_MESSAGES
        }
    },
    validate() {
        return function (dispatch:any,getState:any) {
            validate().then(result => {
                let { code,success,error,user }=result;
                dispatch({
                    type: types.VALIDATE,
                    payload: { success,error,user }
                });
            });
        }
    }
}
```
### 14.11 src/store/reducers/session.tsx
src/store/reducers/session.tsx
```tsx
import * as types from '../action-types';
import { Session } from '../types';

let initState = {
  error: '',//错误消息
  success: '',//成功消息
  //如果登录成功的话，需要给此属性赋值为登录用户
}
export default function (state:Session = initState, action:any) {
  switch (action.type) {
    case types.REG:///注册方法调用完成后
      //不需要解构老状态
     return {
       ...action.payload
    };
    case types.LOGIN:///注册方法调用完成后
      return {
        ...action.payload
      };
    case types.LOGOUT:///退出方法调用完成后
      return {
        ...action.payload
      };
    case types.CLEAR_MESSAGES:
      return {
        ...state,
        error: '',
        success:''
      };
    case types.VALIDATE:
      return {
        ...state,
        ...action.payload
      };
    default:
      return state;
  }
}
```
## 15.受保护路由
### 15.1 src/containers/PrivateRoute/index.tsx
src/containers/PrivateRoute/index.tsx
```tsx
import * as React from 'react';
import {Route,Redirect} from 'react-router-dom';
import { connect } from 'react-redux';
import { Store } from '../../store/types';
interface Props{
    path: any,
    component: any,
    user:any
}
class PrivateRoute extends React.Component<Props> {
    render() {
        let { path,component: Comp,user }=this.props;
        console.log('user',user)
        if(user){
            return <Route path={path} render={(props) => user? <Comp {...props} />:<Redirect to="/login" />} />;
        }else{
            return <Redirect to="/login"/>;
        }
    }
}
export default connect(
    (state:Store) => state.session
)(PrivateRoute)
```
### 15.2 src/index.tsx
src/index.tsx
```tsx
import PrivateRoute from './containers/PrivateRoute';
<PrivateRoute path="/profile" component={Profile} />
```
## 参考
- [typescript-ketang](https://gitee.com/zhufengpeixun/typescript-ketang.git)