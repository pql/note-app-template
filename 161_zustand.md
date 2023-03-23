## 1.前言

### 1.1 什么是 zustand

- [Zustand](http://www.godic.net/dicts/de/Zustand)是一个德语单词，表示状态
- [Zustand]是一个轻量级的`JavaScript`状态管理库，用于在 React 应用程序中管理状态
- `zustand`使用高阶函数和`hooks`来管理状态，具有极高的灵活性和易用性，使开发人员可以快速、方便地开发 React 应用程序
- `zustand`被称为当下复杂状态管理的最佳选择

### 1.2 zustand 和其他状态管理库的比较

- 简洁易用：`zustand`具有简洁的 API，不需要过多的配置，易于使用。它不需要学习复杂的概念和语法，可以快速上手
- 高效：`zustand`使用了高阶函数和 hooks 来管理状态，具有极高的效率和性能
- 灵活：`zustand`的灵活性极高，可以满足不同的业务需求。开发人员可以根据自己的需求来定制自己的状态管理方案
- 易于集成：`zustand`可以快速集成到现有的 React 项目中，不需要对现有代码进行大量的改动

### 1.3 useSyncExternalStore

- [useSyncExternalStore](https://beta.reactjs.org/reference/react/useSyncExternalStore)是一个自定义 Hook，它提供了一种简单的方法来订阅外部状态管理器，并将其状态同步到 React 组件中
- `useSyncExternalStore`函数的第一个参数是一个订阅函数，它接收一个回调函数作为参数，当状态发生变化时，该回调函数将被调用。该回调函数接收两个参数：当前的状态值和上一个状态值
- `useSyncExternalStore`函数的第二个参数是一个获取状态值的函数。当组件需要获取当前状态值时，它将调用该函数并返回当前状态值
- `useSyncExternalStore` 函数返回一个状态值，该值表示当前的状态。当外部状态管理器更新状态时，组件将自动更新状态

### 1.4 immer

- 在 React 应用中我们不希望修改状态对象，但如果每次都创建新对象又会带来额外的性能开销
- 不可变数据结构可以每次都返回一个新的对象，又可以共享未修改的部分
- [immer](https://github.com/immerjs/immer)以最小的成本实现了不可变数据结构
- 对 `draftState` 的修改都会反应到 `nextState` 上
- 而 `Immer` `使用的结构是共享的，nextState` 在结构上又与 `currentState` 共享未修改的部分

![](/public/images/immergraph-1.gif)

```js
import { produce } from "immer";
let baseState = {
  ids: [1],
  pos: {
    x: 1,
    y: 1,
  },
};

let nextState = produce(baseState, (draft) => {
  draft.ids.push(2);
});
console.log(baseState.ids === nextState.ids); //false
console.log(baseState.pos === nextState.pos); //true
```

```js
import { produce } from "immer";
const baseState = {
  list: ["1", "2"],
};
const result = produce(baseState, (draft) => {
  draft.list.push("3");
});
console.log(baseState);
console.log(result);
```

## 2.安装

```bash
npm install zustand immer --save
```

## 3.使用状态管理器

### 3.1 定义状态管理器

src\App.js

```js
// 引入 zustand 库中的 create 函数
import { create } from "./zustand";
/**
 * 定义了一个名为 useStore 的状态管理器，状态管理器有三个属性：number，add，minus
 * number 属性代表状态管理器中的状态，add 和 minus 函数分别是更新 number 属性的方法
 */
// 创建一个名为 useStore 的状态管理器
// 参数是一个createState方法，用来返回管理的状态
// useStore也是一个函数，可以通过调用它返回管理的状态
const useStore = create((set) => ({
  // 定义 number 属性，初始值为 0
  number: 0,
  // 定义 name 属性，初始值为 Number
  name: "Number",
  // 定义 add 函数，通过 set 函数更新状态
  add: () => set((state) => ({ number: state.number + 1 })),
  // 定义 minus 函数，通过 set 函数更新状态
  minus: () => set((state) => ({ number: state.number - 1 })),
}));
```

### 3.2 使用状态

src\App.js

```js
// 引入 zustand 库中的 create 函数
import { create } from './zustand';
/**
 * 定义了一个名为 useStore 的状态管理器，状态管理器有三个属性：number，add，minus
 * number 属性代表状态管理器中的状态，add 和 minus 函数分别是更新 number 属性的方法
 */
// 创建一个名为 useStore 的状态管理器
const useStore = create(set => ({
    // 定义 number 属性，初始值为 0
    number: 0,
    // 定义 name 属性，初始值为 Number
    name: 'Number',
    // 定义 add 函数，通过 set 函数更新状态
    add: () => set(state => ({ number: state.number + 1 })),
    // 定义 minus 函数，通过 set 函数更新状态
    minus: () => set(state => ({ number: state.number - 1 }))
}));
// React组件展示了如何使用状态管理器中的状态和方法
function App() {
+    // 从useStore状态管理器中解构出了四个状态：name，number，add，minus
+    const { name, number, add, minus } = useStore();
+    return (
+        <div>
+            <p>{name}: {number}</p>
+            <button onClick={add}>+</button>
+            <button onClick={minus}>-</button>
+        </div>
+    );
}
export default App;
```

## 4.读取状态

### 4.1 zustand\index.js

src\zustand\index.js

```js
export * from "./react";
```

### 4.2 react.js

src\zustand\react.js

```js
import { createStore } from "./vanilla";
export function useStore(api) {
  return api.getState();
}
const createImpl = (createState) => {
  const api = createStore(createState);
  return () => useStore(api);
};
export const create = (createState) => createImpl(createState);
export default create;
```

### 4.3 vanilla.js

src\zustand\vanilla.js

```js
const createStoreImpl = (createState) => {
  let state;
  const getState = () => state;
  const setState = (partial) => {
    const nextState = typeof partial === "function" ? partial(state) : partial;
    if (!Object.is(nextState, state)) {
      const previousState = state;
      state = Object.assign({}, state, nextState);
    }
  };
  const api = {
    getState,
    setState,
  };
  state = createState();
  return api;
};
export const createStore = (createState) => createStoreImpl(createState);
export default createStore;
```

## 5.修改状态

### 5.1 vanilla.js

src\zustand\vanilla.js

```js
// 定义 createStoreImpl 函数，接收 createState 函数作为参数
const createStoreImpl = createState => {
    // 定义变量 state 和 listeners
    let state;
+    let listeners = new Set();
    // 定义函数 getState，返回 state 的值
    const getState = () => state;
    // 定义函数 setState，接收 partial 作为参数
    const setState = (partial) => {
        // 根据 partial 的类型，判断 nextState 的值
        const nextState = typeof partial === 'function' ? partial(state) : partial;
        // 如果 nextState 和 state 的值不同
        if (!Object.is(nextState, state)) {
            // 记录上一个状态
            const previousState = state;
            // 将 state 更新为 nextState
            state = Object.assign({}, state, nextState);
            // 遍历 listeners，执行每个监听器的回调函数
+            listeners.forEach(listener => listener(state, previousState));
        }
    }
+    // 定义函数 subscribe，接收 listener 作为参数
+    const subscribe = (listener) => {
+        // 将 listener 添加到 listeners 中
+        listeners.add(listener);
+        // 返回一个函数，用于从 listeners 中删除 listener
+        return () => listeners.delete(listener);
+    }
    // 定义变量 api，包含 getState、setState 和 subscribe 函数
    const api = {
        getState,
        setState,
+        subscribe
    }
    // 调用 createState 函数，初始化 state 的值
+    state = createState(setState, getState, api);
    // 返回 api
    return api;
}

// 定义 createStore 函数，接收 createState 函数作为参数，返回 createStoreImpl(createState)
export const createStore = createState => createStoreImpl(createState);

// 默认导出 createStore 函数
export default createStore;
```

### 5.2 react.js

src\zustand\react.js

```js
// 导入 createStore 函数
import { createStore } from './vanilla';
// 导入 useSyncExternalStore 函数
+import { useSyncExternalStore } from 'react';

// 定义 useStore 函数，接收 api 对象作为参数
export function useStore(api) {
    // 使用 useSyncExternalStore 函数从 api 中获取状态值
+    let value = useSyncExternalStore(api.subscribe, api.getState);
+    return value;
}

// 定义 createImpl 函数，接收 createState 函数作为参数
const createImpl = createState => {
    // 调用 createStore 函数创建状态管理器
    const api = createStore(createState);
    // 返回一个函数，该函数将 api 对象传递给 useStore 函数
    return () => useStore(api)
}

// 定义 create 函数，接收 createState 函数作为参数，返回 createImpl 函数的调用结果
export const create = createState => createImpl(createState);

// 默认导出 create 函数
export default create;
```

## 6.异步

- 当你准备好时只需调用 `set`，`zustand` 不关心你的操作是否是异步的

```js
// 引入 zustand 库中的 create 函数
import { create } from './zustand';
/**
 * 定义了一个名为 useStore 的状态管理器，状态管理器有三个属性：number，add，minus
 * number 属性代表状态管理器中的状态，add 和 minus 函数分别是更新 number 属性的方法
 */
// 创建一个名为 useStore 的状态管理器
const useStore = create(set => ({
    // 定义 number 属性，初始值为 0
    number: 0,
    // 定义 name 属性，初始值为 Number
    name: 'Number',
    // 定义 add 函数，通过 set 函数更新状态
    add: () => set(state => ({ number: state.number + 1 })),
    // 定义 minus 函数，通过 set 函数更新状态
    minus: () => set(state => ({ number: state.number - 1 })),
+ asyncAdd: () => {
+     setTimeout(() => {
+         set(state => ({ number: state.number + 1 }));
+     }, 1000);
+ }
}));
// React组件展示了如何使用状态管理器中的状态和方法
function App() {
    // 从useStore状态管理器中解构出了四个状态：name，number，add，minus
+    const { name, number, add, minus, asyncAdd } = useStore();
    return (
        <div>
            <p>{name}: {number}</p>
            <button onClick={add}>+</button>
            <button onClick={minus}>-</button>
+         <button onClick={asyncAdd}>async +</button>
        </div>
    );
}
export default App;
```

## 7.日志中间件

- 中间件是一个函数，用于增强 store（即状态管理器）的功能
- 中间件可以截获`store`中的`set`方法调用并进行一些操作，例如日志记录、性能跟踪、异常处理等等
- 中间件提供了一种灵活且可扩展的方式来修改或增强 store 的行为，同时保持了 store 的独立性和可测试性
- 日志中间件是一个函数，它可以用作`store`的中间件来记录状态的变化和相关信息，例如时间戳、新旧状态等

### 7.1 src\App.js

src\App.js

```js
import { create } from './zustand'; // 导入 create 函数从 './zustand' 模块中
+import logger from './zustand/middleware/logger'; // 导入 logger 中间件从 './zustand/middleware/logger' 模块中

// 创建一个名为 useStore 的状态钩子
// 该状态钩子使用 logger 中间件
// 该状态钩子的状态有 number, name, add, minus, asyncAdd 五个属性
+const useStore = create(logger((set) => {
    return {
        number: 0,
        name: 'Number',
        add: () => set(state => ({ number: state.number + 1 })),
        minus: () => set(state => ({ number: state.number - 1 })),
        asyncAdd: () => {
            setTimeout(() => {
                set(state => ({ number: state.number + 1 }));
            }, 1000);
        }
    }
}));

// 定义一个函数组件 App
// 该组件使用 useStore 状态钩子，解构出其状态对象的 number、name、add、minus 和 asyncAdd 属性
// 在组件渲染时展示 number 和 name 属性值，并提供三个按钮：add、minus 和 asyncAdd，分别调用对应的函数
function App() {
    const { number, name, add, minus, asyncAdd } = useStore();
    return (
        <div>
            <p>{name}:{number}</p>
            <button onClick={add}>+</button>
            <button onClick={minus}>-</button>
            <button onClick={asyncAdd}>async +</button>
        </div>
    );
}

export default App; // 导出 App 组件作为默认导出
```

### 7.2 logger.js

src\zustand\middleware\logger.js

```js
// 定义了一个名为 logger 的函数，它是一个高阶函数，因为它接收了一个函数作为参数并返回了一个新的函数
// 定义一个名为 logger 的常量，该常量是一个高阶函数，它接收一个名为 createState 的参数
const logger =
  (createState) =>
  // 返回一个新的函数，该函数接收三个参数：set，get，api
  (set, get, api) =>
    // 调用 createState 函数，并将三个参数作为它的参数
    createState(
      // 返回一个函数，该函数接收任意数量的参数，并执行以下操作：
      (...args) => {
        // 在控制台输出旧状态
        console.log(`old state:`, get());
        // 调用 set 函数，并将 args 作为它的参数
        set(...args);
        // 在控制台输出新状态
        console.log(`new state`, get());
      },
      // 将 get 作为参数传递给 createState
      get,
      // 将 api 作为参数传递给 createState
      api
    );
export default logger;
```

## 8.状态分片

- 状态分片是将整个应用程序的状态(state)拆分为不同的部分，每个部分被称为状态分片(slice)
- 这样可以将状态(state)和状态更新的逻辑细分为多个独立的模块，从而使应用程序的状态管理更加清晰和可维护

### 8.1 App.js

src\App.js

```js
import { create } from './zustand'; // 导入 create 函数从 './zustand' 模块中
import logger from './zustand/middleware/logger'; // 导入 logger 中间件从 './zustand/middleware/logger' 模块中

// 创建一个名为 useStore 的状态钩子
// 该状态钩子使用 logger 中间件
// 该状态钩子的状态有 number, name, add, minus, asyncAdd 五个属性
const useStore = create(logger((set) => {
    return {
        number: 0,
        name: 'Number',
        add: () => set(state => ({ number: state.number + 1 })),
        minus: () => set(state => ({ number: state.number - 1 })),
        asyncAdd: () => {
            setTimeout(() => {
                set(state => ({ number: state.number + 1 }));
            }, 1000);
        }
    }
}));

// 定义一个函数组件 App
// 该组件使用 useStore 状态钩子，解构出其状态对象的 number、name、add、minus 和 asyncAdd 属性
// 在组件渲染时展示 number 和 name 属性值，并提供三个按钮：add、minus 和 asyncAdd，分别调用对应的函数
function App() {
+    const { number, name, add, minus, asyncAdd } = useStore(state => (
+        {
+            number: state.number,
+            add: state.add,
+            minus: state.minus,
+            asyncAdd: state.asyncAdd
+        }));
    return (
        <div>
            <p>{name}:{number}</p>
            <button onClick={add}>+</button>
            <button onClick={minus}>-</button>
            <button onClick={asyncAdd}>async +</button>
        </div>
    );
}

export default App; // 导出 App 组件作为默认导出
```

### 8.2 react.js

src\zustand\react.js

```js
// 导入 createStore 函数
import { createStore } from './vanilla';
// 导入 useSyncExternalStore 函数
+import { useSyncExternalStore, useRef, useCallback } from 'react';
// 定义 useStore 函数，接收 api 对象作为参数
+export function useStore(api, selector) {
+    const lastSnapshotRef = useRef(null);
+    const lastSelectionRef = useRef(null);
+    const getSelection = useCallback(() => {
+        let lastSelection = lastSelectionRef.current;
+        if (lastSelection === null) {
+            const nextSnapShot = api.getState();
+            lastSelection = selector(nextSnapShot);
+            lastSnapshotRef.current = nextSnapShot;
+            lastSelectionRef.current = lastSelection;
+            return lastSelection;
+        }
+        const lastSnapshot = lastSnapshotRef.current;
+        const nextSnapShot = api.getState();
+        if (Object.is(lastSnapshot, nextSnapShot)) {
+            return lastSelection;
+        }
+        const nextSelection = selector(nextSnapShot);
+        lastSnapshotRef.current = nextSnapShot;
+        lastSelectionRef.current = nextSelection;
+        return nextSelection;
+    }, []);
    // 使用 useSyncExternalStore 函数从 api 中获取状态值
+    let value = useSyncExternalStore(api.subscribe, getSelection);
    return value;
}

// 定义 createImpl 函数，接收 createState 函数作为参数
const createImpl = createState => {
    // 调用 createStore 函数创建状态管理器
    const api = createStore(createState);
    // 返回一个函数，该函数将 api 对象传递给 useStore 函数
+    return (getSelection) => useStore(api, getSelection)
}

// 定义 create 函数，接收 createState 函数作为参数，返回 createImpl 函数的调用结果
export const create = createState => createImpl(createState);

// 默认导出 create 函数
export default create;
```

## 9. Persist 中间件

### 9.1 src\App.js

src\App.js

```js
import { create } from './zustand'; // 导入 create 函数从 './zustand' 模块中
import logger from './zustand/middleware/logger'; // 导入 logger 中间件从 './zustand/middleware/logger' 模块中
+import { persist, createJSONStorage } from './zustand/middleware/persist'
// 创建一个名为 useStore 的状态钩子
// 该状态钩子使用 logger 中间件
// 该状态钩子的状态有 number, name, add, minus, asyncAdd 五个属性
+const useStore = create(persist((set) => {
    return {
        number: 0,
        name: 'Number',
        add: () => set(state => ({ number: state.number + 1 })),
        minus: () => set(state => ({ number: state.number - 1 })),
        asyncAdd: () => {
            setTimeout(() => {
                set(state => ({ number: state.number + 1 }));
            }, 1000);
        }
    }
+}, {
+    name: 'counter', // unique name
+    storage: createJSONStorage(sessionStorage)
+}));
// 定义一个函数组件 App
// 该组件使用 useStore 状态钩子，解构出其状态对象的 number、name、add、minus 和 asyncAdd 属性
// 在组件渲染时展示 number 和 name 属性值，并提供三个按钮：add、minus 和 asyncAdd，分别调用对应的函数
function App() {
    const { number, name, add, minus, asyncAdd } = useStore(state => (
        {
            number: state.number,
            name: state.name,
            add: state.add,
            minus: state.minus,
            asyncAdd: state.asyncAdd
        }));
    return (
        <div>
            <p>{name}:{number}</p>
            <button onClick={add}>+</button>
            <button onClick={minus}>-</button>
            <button onClick={asyncAdd}>async +</button>
        </div>
    );
}

export default App; // 导出 App 组件作为默认导出
```

### 9.2 persist.js

src\zustand\middleware\persist.js

```js
export function createJSONStorage(storage) {
  const persistStorage = {
    getItem: (name) => {
      const str = storage.getItem(name);
      return str ? JSON.parse(str) : {};
    },
    setItem: (name, newValue) =>
      storage.setItem(name, JSON.stringify(newValue)),
  };
  return persistStorage;
}

export const persist = (createState, { name, storage }) => {
  return (set, get, api) => {
    const result = createState(
      (...args) => {
        set(...args);
        storage.setItem(name, get());
      },
      get,
      api
    );
    queueMicrotask(() => {
      set(storage.getItem(name));
    });
    return result;
  };
};
```

## 10. immer 中间件

### 10.1 src\App.js

src\App.js

```js
import { create } from './zustand'; // 导入 create 函数从 './zustand' 模块中
import logger from './zustand/middleware/logger'; // 导入 logger 中间件从 './zustand/middleware/logger' 模块中
import { persist, createJSONStorage } from './zustand/middleware/persist'
+import { immer } from './zustand/middleware/immer'
// 创建一个名为 useStore 的状态钩子
// 该状态钩子使用 logger 中间件
// 该状态钩子的状态有 number, name, add, minus, asyncAdd 五个属性
+const useStore = create(immer((set) => {
    return {
        number: 0,
        name: 'Number',
+        add: () => set(state => { state.number += 1 }),
+        minus: () => set(state => { state.number -= 1 }),
        asyncAdd: () => {
            setTimeout(() => {
                set(state => ({ number: state.number + 1 }));
            }, 1000);
        }
    }
}));
// 定义一个函数组件 App
// 该组件使用 useStore 状态钩子，解构出其状态对象的 number、name、add、minus 和 asyncAdd 属性
// 在组件渲染时展示 number 和 name 属性值，并提供三个按钮：add、minus 和 asyncAdd，分别调用对应的函数
function App() {
    const { number, name, add, minus, asyncAdd } = useStore(state => (
        {
            number: state.number,
            name: state.name,
            add: state.add,
            minus: state.minus,
            asyncAdd: state.asyncAdd
        }));
    return (
        <div>
            <p>{name}:{number}</p>
            <button onClick={add}>+</button>
            <button onClick={minus}>-</button>
            <button onClick={asyncAdd}>async +</button>
        </div>
    );
}
export default App; // 导出 App 组件作为默认导出
```

### 10.2 immer.js

src\zustand\middleware\immer.js

```js
import { produce } from "immer";
const immer = (initializer) => (set, get, store) => {
  store.setState = (updater) => {
    const nextState = produce(updater);
    return set(nextState);
  };
  return initializer(store.setState, get, store);
};
export { immer };
```
