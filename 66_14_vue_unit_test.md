## 前端常见问题
修改js模块功能，其它模块也受影响，很难快速定位bug
多人开发代码越来越难以维护,不方便迭代,代码无法重构

## 什么是单元测试
单元测试就是测试最小单元(一个方法，一个组件)

## TDD & BDD
- Test-Driven Development, 测试驱动开发
    - 先编写测试用例代码，然后针对测试用例编写功能代码，使其能够通过
    - 很好的诠释了代码即文档
    - 清晰地了解软件的需求
- Behavior Driven Development，行为驱动开发
    - 系统业务专家、开发者、测试人员一起合作，分析软件的需求，然后将这些需求写成一个个的故事。开发者负责填充这些故事的内容
    - 保证程序实现效果与用户需求一致。
## 测试工具 mocha + chai / jest
- karma + Jasmine + chrome-launcher
- karma + mocha + chai / jest 使用jsdom

Karma为前端自动化测试提供了跨浏览器测试的能力,Mocha是前端自动化测试框架，测试框架需要解决兼容不同风格断言库,jest 是facebook推出的一款测试框架,集成了 Mocha,chai,jsdom,sinon等功能。

## 创建项目使用mocha+chai
```sh
vue create mocha-vue
// 增加setup文件
global.performance = window.performance;
// 配置启动文件
vue-cli-service test:unit -r tests/setup.js
```
## 测试第一个例子
```js
export let parser = (str) =>{
    let obj = {};
    str.replace(/([^&=]*)=([^&=]*)/g,function(){
        obj[arguments[1]] = arguments[2];
    });
    return obj;
}
export let stringify = (obj) =>{
    let arr = [];
    for(let key in obj){
        arr.push(`${key}=${obj[key]}`);
    }
    return arr.join('&');
}
// 测试parser 和 stringify
console.log(parser('name=zfpx'));
console.log(stringify({name:'zfpx'}));
```
> 用例无法保存，污染代码，不直观，所有的用例全部混在一起

## 编写测试用例
会默认测试tests文件夹下.spec 和 .test文件
```js
import { expect } from 'chai'
import {parser,stringify} from '@/code/parser';
describe('测试parser.js是否靠谱', () => {
  it('parser测试',()=>{
    expect(parser('name=zfpx&age=9')).to.be.deep.eq({name:'zfpx',age:'9'});
  });
  it('stringify测试',()=>{
    expect(stringify({name:'zfpx'})).to.be.eq('name=zfpx');
  })
});
```
## chai库的应用
```js
import { expect } from 'chai'
describe('测试常见的比较方法', () => {
  it('常见判断方式',()=>{
    expect('zfpx').to.be.equal('zfpx');
    // 简写
    expect(true).to.be.equal(true);
    expect(true).to.be.true;
    expect([1,2,3]).to.be.lengthOf(3);
    // 判断包含
    expect('welcome zf').to.contain('zf');    
    expect({name:'zfpx'}).to.not.have.haveOwnPropertyDescriptor('address');
    expect('zfpx').match(/zf/);
    // 大于小于
    expect(5).to.be.lessThan(6);
    expect(3).to.be.greaterThan(2);
  });
});
```
> https://www.chaijs.com/

## 测试vue组件
```js
import { expect } from 'chai';
import HelloWorld from '@/components/HelloWorld';
import Vue from 'vue';
describe('测试vue组件', () => {
  it('测试属性是否正确',()=>{
    let baseExtend = Vue.extend(HelloWorld);
    let vm = new baseExtend({
      propsData:{msg:'zfjg'}
    }).$mount();
    expect(vm.$el.querySelector('h1').innerHTML).to.contain('zfjg');
  })
});
```
需要自己挂载组件,而且还需要自己查找dom元素，很麻烦

## 使用 Vue Test Utils简化流程
```js
describe('测试vue组件', () => {
  it('测试属性是否正确',()=>{
    let wrapper = shallowMount(HelloWorld,{
      propsData:{msg:'zfjg'}
    });
    expect(wrapper.find('h1').text()).to.be.contain('zfjg');
  })
});
```

## 单元测试事件的触发
点击事件
```vue
<template>
    <div>
        <span id="count">{{count}}</span>
        <button @click="increment">点击</button>
    </div>
</template>
<script>
export default {
    data(){
        return {count:10}
    },
    methods:{
        increment(){
            this.count++;
        }
    }
}
</script>

// 对应单元测试
import {shallowMount} from '@vue/test-utils';
import Counter from '@/components/Counter';
import {expect} from 'chai';
describe('Conter组件',()=>{
    it('点击按钮是否可以加1',()=>{
        // 挂载counter组件
        let wrapper = shallowMount(Counter);
        wrapper.setData({count:10}); // 设置状态
        // mock状态
        expect(wrapper.find('span').text()).to.be.equal('10');
        wrapper.find('button').trigger('click');
        expect(wrapper.find('span').text()).to.be.equal('11');
    });
});
```
自定义事件
```vue
<template>
    <div>
        <Child @show="show"></Child>
        <p v-if="flag"> {{name}} </p>
    </div>
</template>
<script>
import Child from './Child.vue';
export default {
    data(){
        return {name:'姜文',flag:false}
    },
    methods:{
        show(){
            this.flag = true;
        }
    },
    components:{
        Child
    }    
}
</script>

// 测试用例
import {shallowMount} from '@vue/test-utils';
import Parent from '@/components/Parent';
import Child from '@/components/Child';
import {expect} from 'chai';

describe('测试子组件能否触发父组件方法',()=>{
    it('触发show方法',()=>{
        let wrapper = shallowMount(Parent);
        expect(wrapper.find('p').exists()).to.be.false;
        wrapper.find(Child).vm.$emit('show');
        expect(wrapper.find('p').exists()).to.be.true;
    })
});
```
## sinon应用
模拟函数调用,统计函数调用次数和调用时的参数
```sh
npm install sinon
```
```vue
<template>
    <div>
        <button @click="handleClick">点我啊</button>
    </div>
</template>
<script>
export default {
    props:{
        fn:{}
    },
    methods:{
        handleClick(){
            this.fn('hello','world');
        }
    }
}
</script>

// 测试
import {shallowMount} from '@vue/test-utils';
import PropFn from '@/components/PropFn.vue';
import {expect} from 'chai';
import sinon from 'sinon';
describe('测试propFn组件',()=>{
    it('判断点击按钮时是否可以触发函数调用',()=>{
        let callback = sinon.spy();
        let wrapper = shallowMount(PropFn,{
            propsData:{
                fn:callback
            }
        });
        wrapper.find('button').trigger('click');
        expect(callback.callCount).to.be.equal(1);
        expect(callback.getCall(0).args).to.be.lengthOf(2);
    });
});
```
## 使用moxios
在mocha中测试axios
```sh
npm install moxios
```
```vue
<template>
    <div>
        {{user}}
    </div>
</template>
<script>
import axios from 'axios';
export default {
    data(){
        return {user:''}
    },  
    mounted(){
        axios.get('/user').then((res)=>{
            this.user = res.data.user;
        }).catch(err=>{
            console.log(err);
        });
    }
}
</script>

// 单元测试
import {shallowMount} from '@vue/test-utils';
import Axios from '@/components/Axios';
import {expect} from 'chai';
import moxios from 'moxios';

describe('测试Axios.vue组件',()=>{
    beforeEach(()=>{
        moxios.install();
    });
    afterEach(()=>{
        moxios.uninstall();
    });
    it('使用mixios模拟接口',(done)=>{
        let wrapper = shallowMount(Axios);
        moxios.stubRequest('/user',  {
            status: 200,
            response: {user:'jw'}
        });
        moxios.wait(function () {
            expect(wrapper.text()).to.include('jw');
            done();
        });
    })
});
```
## 转向jest单元测试
```js
// 相等
expect(1+1).toBe(2);
expect({name:'zfpx'}).toEqual({name:'zfpx'});
// 包含
expect('zfpx').toContain('zf');
expect({a:1}).toHaveProperty('a');
expect('zf').toMatch('z');
// 路由运算
expect(5).toBeLessThan(10);
expect(5).toBeGreaterThan(12);
```
## mock axios
模拟ajax请求方法
```js
增加__mocks__文件夹
export default {
    get:()=>Promise.resolve({data:{user:'zfpx'}})
}

import { shallowMount } from '@vue/test-utils'
import Axios from '@/components/Axios.vue'
import Vue from 'vue';
jest.mock('axios');

describe('Axios.vue', () => {
  it('模拟ajax请求', () => {
    const wrapper = shallowMount(Axios);
    return Vue.nextTick().then(()=>{
      expect(wrapper.text()).toContain('jw');
    })
  })
})
```
## 在组件中测试vuex
模拟vuex的状态和action测试组件
```vue
<template>
    <div>
        <span>{{username}}</span>
        <button @click="change()">更改用户名</button>
    </div>
</template>
<script>
import {mapState,mapActions} from 'vuex';
export default {
    computed:{
       ...mapState(['username'])
    },
    methods:{
       ...mapActions(['change_username']),
        change(){
            this['change_username']('newUsername');
        }
    }
}
</script>

// 测试action
import { shallowMount , createLocalVue} from '@vue/test-utils'
import VuexComponent from '@/components/Vuex.vue';
import Vuex from 'vuex';
const localVue = createLocalVue()
localVue.use(Vuex)
describe('测试组件中的vuex', () => {
  let state;
  let actions;
  let store;
  let fn = jest.fn();
  beforeEach(() => {
    state = {username:'jw'}
    actions = {change_username:fn}
    store = new Vuex.Store({
      state,
      actions
    })
  })
  it('测试state是否正确', () => {
      let wrapper = shallowMount(VuexComponent,{
        store,
        localVue
      });
      expect(wrapper.find('span').text()).toContain('jw');
  });
  it('测试action是否正确调用',()=>{
      let wrapper = shallowMount(VuexComponent,{
        store,
        localVue
      });
      wrapper.find('button').trigger('click');
      expect(fn.mock.calls[0][1]).toEqual('newUsername');
      expect(fn).toHaveBeenCalled()
  })
})
```
## 分别单元化测试 getter、mutation 和 action (需要伪造commit 和 dispatch)

```js
export default {
  increment(state) {
    state.count++
  }
}
it('加1', () => {
  const state = {
    count: 0
  }
  mutations.increment(state)
  expect(state.count).toBe(1)
})
```
## 测试一个运行中的store
```js
import {createLocalVue} from '@vue/test-utils'
import Vuex from 'vuex';
import config from  '@/store.js'
import cloneDeep from 'lodash/cloneDeep'

describe('测试组件中的vuex', () => {
  it('测试store',()=>{
    const localVue = createLocalVue();
    localVue.use(Vuex);
    let store = new Vuex.Store(cloneDeep(config));
    expect(store.state.username).toEqual('zfpx');
    store.commit('set_username','jw');
    expect(store.state.username).toEqual('jw');
  });
});
```