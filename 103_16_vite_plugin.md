## 1.初始化项目
### 1.1 安装
```sh
pnpm init -y
pnpm install vite @vitejs/plugin-vue @rollup/pluginutils vue/compiler-sfc hash-sum --save-dev
```
### 1.2 vite.config.js
```js
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";
export default defineConfig({
  plugins: [vue({})]
});
```
### 1.3 index.html
```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>vue</title>
</head>
<body>
  <div id="app"></div>
  <script type="module" src="/src/main.js"></script>
</body>
</html>
```
### 1.4 src\main.js
src\main.js

```js
import { createApp } from 'vue';
import App from './App.vue';
createApp(App).mount("#app");
```
### 1.5 src\App.vue
src\App.vue
```vue
<template>
  <h1>App</h1>
</template>
<script>
export default {
  name: 'App'
}
</script>
<style>
h1 {
  color: red;
}
</style>
```
### 1.6 package.json
```json
{
  "scripts": {
    "dev": "vite"
  }
}
```
## 2.实现vue插件
### 2.1 vite.config.js
vite.config.js
```js
import { defineConfig } from "vite";
-import vue from "@vitejs/plugin-vue";
+import vue from "./plugins/plugin-vue";
export default defineConfig({
  plugins: [vue({})]
});
```
### 2.2 plugin-vue.js
plugins\plugin-vue.js

```js
import { createFilter, normalizePath } from '@rollup/pluginutils';
import { parse, compileScript, rewriteDefault, compileTemplate, compileStyleAsync } from 'vue/compiler-sfc';
import hash from 'hash-sum';
import path from 'path';
import fs from 'fs';
const root = process.cwd();
const descriptorCache = new Map();
function vue(pluginOptions) {
  const { include = /\.vue$/, exclude } = pluginOptions;
  const filter = createFilter(include, exclude);
  return {
    name: 'vue',
    async load(id) {
      //.log('id', id);//C:\aproject\zhufengwebpack202202\16.viteplugin\src\App.vue
      const { filename, query } = parseVueRequest(id);
      if (!filter(filename)) {
        return null;
      }
      if (query.has('vue')) {
        const descriptor = await getDescriptor(filename);
        if (query.get('type') === 'style') {
          let block = descriptor.styles[Number(query.get('index'))];
          if (block) {
            return { code: block.content };
          }
        }
      }
    },
    async transform(code, id) {
      const { filename, query } = parseVueRequest(id);
      if (!filter(filename)) {
        return null;
      }
      if (query.get('type') === 'style') {
        const descriptor = await getDescriptor(filename);
        let result = await transformStyle(code, descriptor, query.get('index'));
        return result;
      } else {
        let result = await transformMain(code, filename);
        return result;
      }

    }
  }
}
async function transformStyle(code, descriptor, index) {
  const block = descriptor.styles[index];
  //如果是CSS，其实翻译之后和翻译之前内容是一样的，最终返回的JS靠packages\vite\src\node\plugins\css.ts
  const result = await compileStyleAsync({
    filename: descriptor.filename,
    source: code,
    id: `data-v-${descriptor.id}`,//必须传递，不然报错
    scoped: block.scoped
  });
  let styleCode = result.code;
  return {
    code: styleCode
  };
  /*  let styleScript = `
   let style = document.createElement('style');
   style.innerText = ${JSON.stringify(styleCode)};
   document.head.appendChild(style);
   `;
   return {
     code: styleScript
   }; */
}
async function transformMain(source, filename) {
  const descriptor = await getDescriptor(filename, source);
  const scriptCode = genScriptCode(descriptor, filename);
  const templateCode = genTemplateCode(descriptor, filename);
  const stylesCode = genStyleCode(descriptor, filename);
  let resolveCode = [
    stylesCode,
    templateCode,
    scriptCode,
    `_sfc_main.render=render`,
    `export default _sfc_main`
  ].join('\n');
  return {
    code: resolveCode
  }
}
function genStyleCode(descriptor, filename) {
  let styleCode = '';
  if (descriptor.styles.length) {
    descriptor.styles.forEach((style, index) => {
      const query = `?vue&type=style&index=${index}&lang=css`;
      const styleRequest = normalizePath(filename + query);// / 
      styleCode += `\nimport ${JSON.stringify(styleRequest)}`;
    });
    return styleCode;
  }
}
function genTemplateCode(descriptor, filename) {
  let result = compileTemplate({ source: descriptor.template.content, id: filename });
  return result.code;
}
/**
 * 获取此.vue文件编译 出来的js代码
 * @param {*} descriptor 
 * @param {*} filename 
 */
function genScriptCode(descriptor, filename) {
  let scriptCode = '';
  let script = compileScript(descriptor, { id: filename });
  scriptCode = rewriteDefault(script.content, '_sfc_main');//export default => const _sfc_main
  return scriptCode;
}

async function getDescriptor(filename, source) {
  let descriptor = descriptorCache.get(filename);
  if (descriptor) return descriptor;
  const content = await fs.promises.readFile(filename, 'utf8');
  const result = parse(content, { filename });
  descriptor = result.descriptor;
  descriptor.id = hash(path.relative(root, filename));
  descriptorCache.set(filename, descriptor);
  return descriptor;
}
function parseVueRequest(id) {
  const [filename, querystring = ''] = id.split('?');
  let query = new URLSearchParams(querystring);
  return {
    filename, query
  };
}
export default vue;
```
## 3.实现jsx插件
### 3.1 安装
```sh
pnpm install @vitejs/plugin-vue-jsx --save-dev
pnpm install @vue/babel-plugin-jsx @babel/plugin-syntax-import-meta @rollup/pluginutils @babel/plugin-transform-typescript hash-sum morgan fs-extra --save-dev
```
### 3.2 vite.config.js
vite.config.js
```js
import { defineConfig } from "vite";
-import vue from "@vitejs/plugin-vue";
-import vue from "./plugins/plugin-vue";
+import vueJsx from "./plugins/plugin-vue-jsx.js";
export default defineConfig({
+ plugins: [vueJsx({})]
});
```
### 3.3 plugin-vue-jsx.js
plugins\plugin-vue-jsx.js

```js
import { transformSync } from '@babel/core'
import jsx from '@vue/babel-plugin-jsx'
import importMeta from '@babel/plugin-syntax-import-meta'
import { createFilter } from '@rollup/pluginutils'
import typescript from '@babel/plugin-transform-typescript';
function vueJsxPlugin(options = {}) {
  let root;
  return {
    name: 'vite:vue-jsx',
    config() {
      return {
        esbuild: {
          //默认情况下在开发的时候会编译我们的代码，它会也会编译jsx,但是它会编译 成React.createElement
          include: /\.ts$/
        },
        define: {
          __VUE_OPTIONS_API__: true,
          __VUE_PROD_DEVTOOLS__: false
        }
      }
    },
    configResolved(config) {
      root = config.root
    },
    transform(code, id) {
      const {
        include,
        exclude,
        babelPlugins = [],
        ...babelPluginOptions
      } = options
      const filter = createFilter(include || /\.[jt]sx$/, exclude)
      const [filepath] = id.split('?')
      if (filter(id) || filter(filepath)) {
        const plugins = [importMeta, [jsx, babelPluginOptions], ...babelPlugins]
        if (id.endsWith('.tsx') || filepath.endsWith('.tsx')) {
          plugins.push([
            typescript,
            { isTSX: true, allowExtensions: true }
          ])
        }
        const result = transformSync(code, {
          babelrc: false,
          configFile: false,
          ast: true,
          plugins
        })
        return {
          code: result.code,
          map: result.map
        }
      }
    }
  }
}
export default vueJsxPlugin;
```
### 3.4 main.js
src\main.js

```js
import { createApp } from 'vue';
+import App from './App.jsx';
createApp(App).mount("#app");
```
### 3.5 src\App.jsx
src\App.jsx

```js
import { defineComponent } from 'vue';
export default defineComponent({
  setup() {
    return () => (
      <h1>App</h1>
    )
  }
})
```
## 4.HMR
### 4.1 更新消息
```json
{
  "type":"update",
  "updates":[
        {"type":"js-update","timestamp":1647485594371,"path":"/src/App.jsx","acceptedPath":"/src/App.jsx"}
    ]
  }    
```
### 4.2 热更新
```js
import { transformSync } from '@babel/core'
import jsx from '@vue/babel-plugin-jsx'
import importMeta from '@babel/plugin-syntax-import-meta'
import { createFilter } from '@rollup/pluginutils'
import typescript from '@babel/plugin-transform-typescript';
+import hash from 'hash-sum'
+import path from 'path'
function vueJsxPlugin(options = {}) {
+ let needHmr = false
  return {
    name: 'vite:vue-jsx',
    config() {
      return {
        esbuild: {
          include: /\.ts$/
        },
        define: {
          __VUE_OPTIONS_API__: true,
          __VUE_PROD_DEVTOOLS__: false
        }
      }
    },
    configResolved(config) {
      root = config.root
+     needHmr = config.command === 'serve' && !config.isProduction
    },
    transform(code, id) {
      const {
        include,
        exclude,
        babelPlugins = [],
        ...babelPluginOptions
      } = options
      const filter = createFilter(include || /\.[jt]sx$/, exclude)
      const [filepath] = id.split('?')
      if (filter(id) || filter(filepath)) {
        const plugins = [importMeta, [jsx, babelPluginOptions], ...babelPlugins]
        if (id.endsWith('.tsx') || filepath.endsWith('.tsx')) {
          plugins.push([
            typescript,
            { isTSX: true, allowExtensions: true }
          ])
        }
        const result = transformSync(code, {
          babelrc: false,
          configFile: false,
          ast: true,
          plugins
        })
+       if (!needHmr) {
          return { code: result.code, map: result.map }
+       }
+       const hotComponents = []
+       let hasDefault = false
+       for (const node of result.ast.program.body) {
+         if (node.type === 'ExportDefaultDeclaration') {
+           if (isDefineComponentCall(node.declaration)) {
+             hasDefault = true
+             hotComponents.push({
+               local: '__default__',
+               exported: 'default',
+               id: hash(id + 'default')
+             })
+           }
+         }
+       }
+       if (hotComponents.length) {
+         if (hasDefault && (needHmr)) {
+           result.code =
+             result.code.replace(
+               /export default defineComponent/g,
+               `const __default__ = defineComponent`
+             ) + `\nexport default __default__`
+         }

+         if (needHmr && !/\?vue&type=script/.test(id)) {
+           let code = result.code
+           let callbackCode = ``
+           for (const { local, exported, id } of hotComponents) {
+             code +=
+               `\n${local}.__hmrId = "${id}"` +
+               `\n__VUE_HMR_RUNTIME__.createRecord("${id}", ${local})`
+             callbackCode += `\n__VUE_HMR_RUNTIME__.reload("${id}", __${exported})`
+           }
+           code += `\nimport.meta.hot.accept(({${hotComponents
+             .map((c) => `${c.exported}: __${c.exported}`)
+             .join(',')}}) => {${callbackCode}\n})`
+           result.code = code
+         }
+       }
+       return {
+         code: result.code,
+         map: result.map
+       }
      }
    }
  }
}
function isDefineComponentCall(node) {
  return (
    node &&
    node.type === 'CallExpression' &&
    node.callee.type === 'Identifier' &&
    node.callee.name === 'defineComponent'
  )
}
export default vueJsxPlugin;
```
## 5.SSR
### 5.1 SSR-html
#### 5.1.1 ssr.js
```js
import express from "express";
import { createServer } from 'vite'
const app = express();
; (async function () {
  const vite = await createServer({
    server: {
      middlewareMode: 'html'
    }
  })
  app.use(vite.middlewares);
  app.listen(8000, () => console.log('ssr server started on 8000'))
})();
```
### 5.2 SSR-ssr
#### 5.2.1 entry-client.js
src\entry-client.js

```js
import { createApp } from './main'
const { app, router } = createApp()
router.isReady().then(() => {
  app.mount('#app')
})
```
#### 5.2.2 entry-server.js
src\entry-server.js
```js
import { createApp } from './main'
import { renderToString } from '@vue/server-renderer'
export async function render(url, manifest = {}) {
  const { app, router } = createApp()
  router.push(url)
  await router.isReady()
  const ctx = {}
  const html = await renderToString(app, ctx)
  const preloadLinks = renderPreloadLinks(ctx.modules, manifest)
  return [html, preloadLinks]
}
function renderPreloadLinks(modules, manifest) {
  let links = ''
  const seen = new Set()
  modules.forEach((id) => {
    const files = manifest[id]
    if (files) {
      files.forEach((file) => {
        if (!seen.has(file)) {
          seen.add(file)
          links += renderPreloadLink(file)
        }
      })
    }
  })
  return links
}
function renderPreloadLink(file) {
  console.log('file', file);
  if (file.endsWith('.js') || file.endsWith('.jsx')) {
    return `<link rel="modulepreload" crossorigin href="${file}">`
  } else if (file.endsWith('.css')) {
    return `<link rel="stylesheet" href="${file}">`
  } else {
    return ''
  }
}
```
#### 5.2.3 src\main.js
src\main.js
```js
import App from './App.jsx'
import { createSSRApp } from 'vue'
import { createRouter } from './router'
export function createApp() {
  const app = createSSRApp(App)
  const router = createRouter()
  app.use(router)
  return { app, router }
}
```
#### 5.2.4 src\router.js
src\router.js

```js
import {
  createMemoryHistory,
  createRouter as _createRouter,
  createWebHistory
} from 'vue-router'
const pages = import.meta.glob('./pages/*.jsx')
const routes = Object.keys(pages).map((path) => {
  const name = path.match(/\.\/pages(.*)\.jsx$/)[1].toLowerCase()
  return {
    path: name === '/home' ? '/' : name,
    component: pages[path]
  }
})
export function createRouter() {
  return _createRouter({
    history: import.meta.env.SSR ? createMemoryHistory() : createWebHistory(),
    routes
  })
}
```
#### 5.2.5 src\App.jsx

src\App.jsx
```js
import { defineComponent } from 'vue';

export default defineComponent({
  setup() {
    return () => (
      <div>
        <ul>
          <li><router-link to="/">Home</router-link></li>
          <li><router-link to="/user">User</router-link></li>
        </ul>
        <router-view></router-view>
      </div>
    )
  }
})
```
#### 5.2.6 src\pages\Home.jsx
src\pages\Home.jsx

```js
import { defineComponent } from 'vue';
import { useSSRContext } from "vue"
export default defineComponent({
  setup() {
    const ssrContext = useSSRContext()
    console.log(ssrContext.modules);
    return (props, ctx) => {
      console.log('props', props);
      console.log('ctx', ctx);
      return <h1>Home</h1>;
    }
  }
})
```
#### 5.2.7 src\pages\User.jsx
src\pages\User.jsx

```js
import { defineComponent } from 'vue';

export default defineComponent({
  setup() {
    return () => (
      <h1>User</h1>
    )
  }
})
```
#### 5.2.8 server.js
server.js

```js
import express from "express";
import logger from 'morgan';
import { createServer } from 'vite'
import fs from 'fs-extra';
import path from 'path';
const app = express();

; (async function () {
  const vite = await createServer({
    server: {
      middlewareMode: 'ssr'
    }
  })
  let manifest = JSON.parse(fs.readFileSync(path.resolve('dist/client/ssr-manifest.json'), 'utf-8'))
  app.use(vite.middlewares);
  app.use(logger('dev'));
  app.use('*', async (req, res) => {
    const url = req.originalUrl
    try {
      // 1. 读取 index.html
      let template = fs.readFileSync(path.resolve('index.html'), 'utf-8')
      // 2. 应用 Vite HTML 转换。这将会注入 Vite HMR 客户端，
      //    同时也会从 Vite 插件应用 HTML 转换。
      //    例如：@vitejs/plugin-react-refresh 中的 global preambles
      template = await vite.transformIndexHtml(url, template)

      // 3. 加载服务器入口。vite.ssrLoadModule 将自动转换
      //    你的 ESM 源码使之可以在 Node.js 中运行！无需打包
      //    并提供类似 HMR 的根据情况随时失效。
      const { render } = await vite.ssrLoadModule('/src/entry-server.js')

      // 4. 渲染应用的 HTML。这假设 entry-server.js 导出的 `render`
      //    函数调用了适当的 SSR 框架 API。
      //    例如 ReactDOMServer.renderToString()
      const [appHtml, preloadLinks] = await render(url, manifest)
      // 5. 注入渲染后的应用程序 HTML 到模板中。
      const html = template
        .replace(`<!--preload-links-->`, preloadLinks)
        .replace(`<!--app-html-->`, appHtml)
      // 6. 返回渲染后的 HTML。
      res.status(200).set({ 'Content-Type': 'text/html' }).end(html)
    } catch (e) {
      // 如果捕获到了一个错误，让 Vite 来修复该堆栈，这样它就可以映射回
      // 你的实际源码中。
      vite.ssrFixStacktrace(e)
      console.error(e)
      res.status(500).end(e.message)
    }
  })
  app.listen(8000, () => console.log('ssr server started on 8000'))
})();
```
#### 5.2.9 plugin-vue-jsx.js
plugins\plugin-vue-jsx.js

```js
import { transformSync } from '@babel/core'
import jsx from '@vue/babel-plugin-jsx'
import importMeta from '@babel/plugin-syntax-import-meta'
import { createFilter, normalizePath } from '@rollup/pluginutils'
import typescript from '@babel/plugin-transform-typescript';
import hash from 'hash-sum'
const path = require('path')
const ssrRegisterHelperId = '/__vue-jsx-ssr-register-helper'
const ssrRegisterHelperCode =
  `import { useSSRContext } from "vue"\n` +
  `export ${ssrRegisterHelper.toString()}`

function ssrRegisterHelper(comp, filename) {
  const setup = comp.setup
  comp.setup = (props, ctx) => {
    // @ts-ignore
    const ssrContext = useSSRContext()
      ; (ssrContext.modules || (ssrContext.modules = new Set())).add(filename)
    if (setup) {
      return setup(props, ctx)
    }
  }
}
function vueJsxPlugin(options = {}) {
  let root;
  let needHmr = false
  return {
    name: 'vite:vue-jsx',
    config() {
      return {
        esbuild: {
          include: /\.ts$/
        },
        define: {
          __VUE_OPTIONS_API__: true,
          __VUE_PROD_DEVTOOLS__: false
        }
      }
    },
    configResolved(config) {
      root = config.root
      needHmr = config.command === 'serve' && !config.isProduction
    },
    transform(code, id, { ssr }) {
      console.log('ssr', ssr);
      const {
        include,
        exclude,
        babelPlugins = [],
        ...babelPluginOptions
      } = options
      const filter = createFilter(include || /\.[jt]sx$/, exclude)
      const [filepath] = id.split('?')
      if (filter(id) || filter(filepath)) {
        const plugins = [importMeta, [jsx, babelPluginOptions], ...babelPlugins]
        if (id.endsWith('.tsx') || filepath.endsWith('.tsx')) {
          plugins.push([
            typescript,
            { isTSX: true, allowExtensions: true }
          ])
        }
        const result = transformSync(code, {
          babelrc: false,
          configFile: false,
          ast: true,
          plugins
        })
        if (!needHmr) {
          return { code: result.code, map: result.map }
        }
        const hotComponents = []
        let hasDefault = false
        for (const node of result.ast.program.body) {
          if (node.type === 'ExportDefaultDeclaration') {
            if (isDefineComponentCall(node.declaration)) {
              hasDefault = true
              hotComponents.push({
                local: '__default__',
                exported: 'default',
                id: hash(id + 'default')
              })
            }
          }
        }
        if (hotComponents.length) {
          if (hasDefault && (needHmr)) {
            result.code =
              result.code.replace(
                /export default defineComponent/g,
                `const __default__ = defineComponent`
              ) + `\nexport default __default__`
          }

          if (needHmr && !/\?vue&type=script/.test(id)) {
            let code = result.code
            let callbackCode = ``
            for (const { local, exported, id } of hotComponents) {
              code +=
                `\n${local}.__hmrId = "${id}"` +
                `\n__VUE_HMR_RUNTIME__.createRecord("${id}", ${local})`
              callbackCode += `\n__VUE_HMR_RUNTIME__.reload("${id}", __${exported})`
            }
            code += `\nimport.meta.hot.accept(({${hotComponents
              .map((c) => `${c.exported}: __${c.exported}`)
              .join(',')}}) => {${callbackCode}\n})`
            result.code = code
          }
        }
        if (ssr) {
          const normalizedId = normalizePath(path.relative(root, id))
          let ssrInjectCode =
            `\nimport { ssrRegisterHelper } from "${ssrRegisterHelperId}"` +
            `\nconst __moduleId = ${JSON.stringify(normalizedId)}`
         git  for (const { local } of hotComponents) {
            ssrInjectCode += `\nssrRegisterHelper(${local}, __moduleId)`
          }
          result.code += ssrInjectCode
        }
        return {
          code: result.code,
          map: result.map
        }
      }
    }
  }
}
function isDefineComponentCall(node) {
  return (
    node &&
    node.type === 'CallExpression' &&
    node.callee.type === 'Identifier' &&
    node.callee.name === 'defineComponent'
  )
}
export default vueJsxPlugin;
```