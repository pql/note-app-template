## 快速原型开发
可以快速识别.vue文件封装组件插件等功能
```sh
sudo npm install @vue/cli -g
sudo npm install -g @vue/cli-service-global
vue serve Home.vue
```
关闭eslint提示
module.exports = {
    devServer: {
        overlay: {
            warnings: false,
            errors: false
        }
    }
}
## 实现递归菜单组件
```html
<Menu>
    <MenuItem v-for="(item, key) in menuList" :key="key">
        {{item.title}}
    </MenuItem>
    <SubMenu>
        <template #title="title">标题1</template>
        <MenuItem>标题1-1</MenuItem>
        <MenuItem>标题1-2</MenuItem>
        <SubMenu>
            <template #title="title">标题1-1-1</template>
            <MenuItem>标题1-1-1-1</MenuItem>
            <MenuItem>标题1-1-1-1</MenuItem>
        </SubMenu>
    </SubMenu>
</Menu>
```
根据数据递归渲染
```js
menuList: [
    {
        title: "标题1",
        children: [
            {title: "标题1-1"},
            {title: "标题1-2"}
        ]
    },
    {
        title: "标题2",
        children: [
            {title: "标题2-1"},
            {title: "标题2-2"}
        ]
    },
    {
        title: "标题3",
        children: [
            {title: "标题3-1"},
            {title: "标题3-2"}
        ]
    }
]
```
ReSub组件实现
```html
<Menu>
    <template v-for="(item, key) in menuList">
        <MenuItem v-if="!item.children" :key="key">
            {{item.title}}
        </MenuItem>
        <ReSub v-else :key="key" :data="item"></ReSub>
    </template>
</Menu>

// ReSub组件主要作用是递归
<template>
    <SubMenu class="sub">
        <template #title="title">{{data.title}}</template>
        <template v-for="d in data.children">
            <MenuItem v-if="!d.children" :key="d.title">
                {{d.title}}
            </MenuItem>
            <ReSub v-else :data="d" :key="d.title" class="sub"></ReSub>
        </template>
    </SubMenu>
</template>

<script>
import SubMenu from './SubMenu';
import MenuItem from './MenuItem';
export default {
    props: ['data'],
    name: 'ReSub',
    components: {
        SubMenu,
        MenuItem
    }
}
</script>
```
## 使用vue-cli3.0创建vue项目
```sh
vue create <project-name>
```
可以通过vue ui创建项目 / 管理项目依赖
```
vue ui
```
配置vue-config.js
```js
const path = require("path");
module.exports = {
    publicPath: process.env.NODE_ENV === 'production' ? '/vue-project': '/',
    outputDir: 'myassets', // 输出路径
    assetsDir: 'static', // 生成静态目录的文件夹
    runtimeCompiler: true, // 是否可以使用 template 模板
    parallel: require('os').cpus().length > 1, // 多余1核cpu时 启动并行压缩
    productionSourceMap: false, // 生产环境下 不使用sourceMap

    // https://github.com/neutrinojs/webpack-chain
    chainWebpack: config =>  {
        // 控制webpack 内部配置
        config.resolve.alias.set('component', path.resolve(__dirname, 'src/components'))
    },
    // https://github.com/survivejs/webpack-merge
    configureWebpack: {
        // 新增插件等
        plugins: []
    },
    devServer: { // 配置代理
        proxy: {
            '/api': {
                target: "http://a.zf.cn:3000",
                changeOrigin: true
            }
        }
    },
    // 第三方插件配置
    pluginOptions: {
        'style-resources-loader': {
            preProcessor: 'less',
            patterns: [
                // 插入全局样式
                path.resolve(__dirname, 'src/assets/common.less')
            ]
        }
    }
}
```
## defer & async / preload & prefetch
- defer 和 async 在网络读取的过程中都是异步解析
- defer是有顺序依赖的，async只要脚本加载完成后就会执行
- preload 可以对当前页面所需的脚本、样式等资源进行预加载
- prefetch 加载的资源一般不是用于当前页面的，是未来很可能用到的这样一些资源

## 基于vue-cli编写组件
小球的滚动组件
- 更改小球的颜色
```vue
<template>
    <div :id="`ball${_uid}`" class="ball" :style="{background: color}"></div>
</template>
<script>
export default {
    name: 'ScrollBall',
    props: {
        color: {
            type: String,
            default: 'red'
        }
    }
}
</script>
<style lang="less">
.ball {
    width: 80px;
    height: 80px;
    border-radius: 50%;
}
</style>
```
- 球的滚动 requestAnimationFrame
```vue
<ScrollBall color="red" :target="500" v-model="pos1"></ScrollBall>
<ScrollBall color="blue" :target="300" v-model="pos2"></ScrollBall>

<script>
export default {
    props: {
        value: {
            type: Number,
            default: 0
        },
        target: {
            type: Number,
            required: true
        }
    },
    mounted() {
        let ele = document.getElementById(`ball${this._uid}`);
        let timer;
        let fn = () => {
            let left = this.value + 2;
            if(left > this.target) {
                return cancelAnimationFrame(timer);
            }
            ele.style.transform = `translate(${left}px)`;
            this.$emit('input', left);
            timer = requestAnimationFrame(fn);
        }
        timer = requestAnimationFrame(fn);
    }
}
</script>
```
- 增加球的内容
```js
<ScrollBall color="red" :target="500" v-model="pos">球1</ScrollBall>

<div :id="`ball${_uid}`" class="ball" :style="{background: color}">
    <slot></slot>
</div>
```
- 让小球停止运动，增加小球的停止方法，通过父亲用$refs获取子组件的方法
```js
<ScrollBall color="red" :target="500" v-model="pos1" ref="ball">球1</ScrollBall>
<button @click="stop">stop</button>
stop() {
    this.$refs.ball.stopMove()
}
// 组件中停止小球运动
methods: {
    stopMove() {
        cancelAnimationFrame(this.timer);
    },
    move() {
        let ele = document.getElementById(`ball${this._uid}`);
        let left = this.value + 2;
        if(left > this.target) {
            return this.stopMove();
        }
        ele.style.transform = `translate(${left}px)`;
        this.$emit('input', left);
        this.timer = requestAnimationFrame(this.move);
    }
},
mounted() {
    this.timer = requestAnimationFrame(this.move);
}
```
- 通知小球运动结束
```js
if(left > this.target) {
    this.$emit("end");
    return this.stopMove();
}
```