## 1.前端项目最佳实践
### 1.1 工具选择
| 类别 | 选择 |
| --- | --- |
| 框架 | [react](https://reactjs.org/) |
| JS 语言 | [TypeScript](http://www.typescriptlang.org/) |
| CSS 语言 | [css-modules](https://github.com/css-modules/css-modules)+[less](http://lesscss.org/)+[postcss](https://github.com/postcss/postcss) | 
| JS 编译 | [babel](https://www.babeljs.cn/) |
| 模块打包 | [webpack 全家桶](https://webpack.github.io/) |
| 单元测试 | [jest](https://github.com/facebook/jest)+[enzyme](https://github.com/airbnb/enzyme)+[puppteer](https://github.com/puppeteer/puppeteer)+[jsdom](https://github.com/jsdom/jsdom) |
| 路由 | [react-router](https://github.com/ReactTraining/react-router) |
| 数据流 | [dva](https://dvajs.com/)+[redux生态](https://www.redux.org.cn/) |
| 代码风格 | [eslint](https://eslint.org/) + [prettier](https://prettier.io/) |
| JS 压缩 | [TerserJS](https://github.com/terser/terser) |
| CSS 压缩 | [cssnano](https://github.com/cssnano/cssnano) |
| 请求库 | [umi-request](https://github.com/umijs/umi-request#readme) |
| UI | [AntDesign](https://ant.design/docs/react/introduce-cn) + [AntDesignPro](https://pro.ant.design/index-cn) |
| 国际化 | [react-intl](https://github.com/formatjs/react-intl) |
| hooks 库 | [umi-hooks](https://hooks.umijs.org/) |
| 静态文档 | [docz](https://www.docz.site/) |
| 微前端 | [qiankun](https://github.com/umijs/qiankun) |
| 图表库 | [antv](https://antv.vision/) |

### 1.2 技术栈选型
#### 1.2.1 固定化
- React 框架
- TypeScript 语言
- Less+CSS Modules
- Eslint+Prettier+固定配置
- 固定数据流方案 dva
- 固定 babel 插件
- Jest+Enzyme
- 框架版本不允许锁定，^前缀必须有
- 主要依赖不允许自定义依赖版本
#### 1.2.2 配置化
- 不仅是框架功能，还有 UI 界面
- 路由、布局、菜单、导航、面包屑、权限、请求、埋点、错误处理
- 只管写 Page 页面就可以了
##### 1.2.2.1 编译态配置
- 给 node.js 使用，比如 webpack、babel 相关配置，静态路由配置
##### 1.2.2.2 运行态配置
- 给浏览器用、比如渲染逻辑、动态修改路由、获取用户信息
### 1.3 约定化
- 国际化
- 数据流
- MOCK
- 目录结构
- 404
- 权限策略
- Service
- 配置文件
### 1.4 理念
- 通过最佳实践减少不必要的选择的差异
- 通过插件和插件集的架构方式，满足不同场景的业务
- 通过资产市场和场景市场着力解决 70%的开发者问题
- 通过对垂直场景采取强约束的方式，进一步提升研发效率
- 不给选择、配置化、约定化
## 2.Ant Design Pro
- Ant Design Pro 是一个企业级中后台前端/设计解决方案，我们秉承 Ant Design 的设计价值观，致力于在设计规范和基础组件的基础上，继续向上构建，提炼出典型模板/业务组件/配套设计资源，进一步提升企业级中后台产品设计研发过程中的『用户』和『设计者』的体验。
- [pro.ant.design](https://pro.ant.design/)
- [beta-pro.ant.design](https://beta-pro.ant.design/index-cn/)
- [procomponents.ant.design](https://procomponents.ant.design/components/table/)
- [getting-started-cn](https://beta-pro.ant.design/docs/getting-started-cn)
### 2.1 启动项目
#### 2.1.1 安装
- 新建一个空的文件夹作为项目目录，并在目录下执行
- [python-380](https://www.python.org/downloads/release/python-380/)
```sh
//npm config set python "C:/Python38/python.exe"
yarn create umi
```
#### 2.1.2 目录结构
- 我们已经为你生成了一个完整的开发框架，提供了涵盖中后台开发的各类功能和坑位，下面是整个项目的目录结构。

```
├─config # umi 配置，包含路由，构建等配置
├─mock   # 本地模拟数据
├─public
│  └─icons
├─src
│  ├─components # 业务通用组件
│  │  ├─Footer
│  │  ├─HeaderDropdown
│  │  ├─HeaderSearch
│  │  ├─NoticeIcon
│  │  └─RightContent
│  ├─e2e       # 集成测试用例
│  ├─locales   # 国际化资源
│  │  ├─en-US
│  │  ├─id-ID
│  │  ├─pt-BR
│  │  ├─zh-CN
│  │  └─zh-TW
│  ├─pages    # 业务页面入口和常用模板
│  │  ├─ListTableList
│  │  │  └─components
│  │  └─user
│  │      └─login
│  ├─services # 后台接口服务
│  └─utils    # 工具库
└─tests       # 测试工具
```

#### 2.1.3 本地开发
```sh
npm install
npm start:dev
```
```sh
git init
git add -A
git commit -m"1.init"
```
### 2.2 用户登录
#### 2.2.1 config\proxy.ts
config\proxy.ts
```ts
export default {
  dev: {
    '/api/': {
+     target: 'http://localhost:4000/',
      changeOrigin: true,
      pathRewrite: { '^': '' }
    }
  }
};
```
#### 2.2.2 src\app.tsx
src\app.tsx
```tsx
import React from 'react';
import { BasicLayoutProps, Settings as LayoutSettings, PageLoading } from '@ant-design/pro-layout';
import { notification } from 'antd';
import { history, RequestConfig } from 'umi';
import RightContent from '@/components/RightContent';
import Footer from '@/components/Footer';
import { ResponseError } from 'umi-request';
import { queryCurrent } from './services/user';
import defaultSettings from '../config/defaultSettings';

export const initialStateConfig = {
  loading: <PageLoading />,
};

export async function getInitialState(): Promise<{
  settings?: LayoutSettings;
  currentUser?: API.CurrentUser;
  fetchUserInfo?: () => Promise<API.CurrentUser | undefined>;
}> {
  const fetchUserInfo = async () => {
    try {
      const currentUser = await queryCurrent();
      return currentUser;
    } catch (error) {
      history.push('/user/login');
    }
    return undefined;
  };
  // 如果是登录页面，不执行
  if (history.location.pathname !== '/user/login') {
    const currentUser = await fetchUserInfo();
    return {
      fetchUserInfo,
      currentUser,
      settings: defaultSettings,
    };
  }
  return {
    fetchUserInfo,
    settings: defaultSettings,
  };
}

export const layout = ({
  initialState,
}: {
  initialState: { settings?: LayoutSettings; currentUser?: API.CurrentUser };
}): BasicLayoutProps => {
  return {
    rightContentRender: () => <RightContent />,
    disableContentMargin: false,
    footerRender: () => <Footer />,
    onPageChange: () => {
      const { currentUser } = initialState;
      const { location } = history;
      // 如果没有登录，重定向到 login
      if (!currentUser && location.pathname !== '/user/login') {
        history.push('/user/login');
      }
    },
    menuHeaderRender: undefined,
    ...initialState?.settings,
  };
};

const codeMessage = {
  200: '服务器成功返回请求的数据。',
  201: '新建或修改数据成功。',
  202: '一个请求已经进入后台排队（异步任务）。',
  204: '删除数据成功。',
  400: '发出的请求有错误，服务器没有进行新建或修改数据的操作。',
  401: '用户没有权限（令牌、用户名、密码错误）。',
  403: '用户得到授权，但是访问是被禁止的。',
  404: '发出的请求针对的是不存在的记录，服务器没有进行操作。',
  405: '请求方法不被允许。',
  406: '请求的格式不可得。',
  410: '请求的资源被永久删除，且不会再得到的。',
  422: '当创建一个对象时，发生一个验证错误。',
  500: '服务器发生错误，请检查服务器。',
  502: '网关错误。',
  503: '服务不可用，服务器暂时过载或维护。',
  504: '网关超时。',
};

/**
 * 异常处理程序
 */
const errorHandler = (error: ResponseError) => {
  const { response } = error;
  if (response && response.status) {
    const errorText = codeMessage[response.status] || response.statusText;
    const { status, url } = response;

    notification.error({
      message: `请求错误 ${status}: ${url}`,
      description: errorText,
    });
  }

  if (!response) {
    notification.error({
      description: '您的网络发生异常，无法连接服务器',
      message: '网络异常',
    });
  }
  throw error;
};

export const request: RequestConfig = {
   errorHandler,
+  headers:{
+    Authorization:`Bearer ${localStorage.getItem('token')}`
+  }
};
```
#### 2.2.3 src\services\API.d.ts 
src\services\API.d.ts
```js
declare namespace API {
  export interface CurrentUser {
    avatar?: string;
   username?: string;
    title?: string;
    group?: string;
    signature?: string;
    tags?: {
      key: string;
      label: string;
    }[];
    userid?: string;
    access?: 'user' | 'guest' | 'admin';
    unreadCount?: number;
  }

  export interface LoginStateType {
    status?: 'ok' | 'error';
    type?: string;
+   token?:string;
  }

  export interface NoticeIconData {
    id: string;
    key: string;
    avatar: string;
    title: string;
    datetime: string;
    type: string;
    read?: boolean;
    description: string;
    clickClose?: boolean;
    extra: any;
    status: string;
  }
}
```
#### 2.2.4 login\index.tsx
src\pages\user\login\index.tsx
```tsx
import {
  AlipayCircleOutlined,
  LockTwoTone,
  MailTwoTone,
  MobileTwoTone,
  TaobaoCircleOutlined,
  UserOutlined,
  WeiboCircleOutlined,
} from '@ant-design/icons';
import { Alert, Space, message, Tabs } from 'antd';
import React, { useState } from 'react';
import ProForm, { ProFormCaptcha, ProFormCheckbox, ProFormText } from '@ant-design/pro-form';
import { useIntl, Link, history, FormattedMessage, SelectLang } from 'umi';
import Footer from '@/components/Footer';
import { fakeAccountLogin, getFakeCaptcha, LoginParamsType } from '@/services/login';

import styles from './index.less';

const LoginMessage: React.FC<{
  content: string;
}> = ({ content }) => (
  <Alert
    style={{
      marginBottom: 24,
    }}
    message={content}
    type="error"
    showIcon
  />
);

/**
 * 此方法会跳转到 redirect 参数所在的位置
 */
const goto = () => {
  const { query } = history.location;
  const { redirect } = query as { redirect: string };
  window.location.href = redirect || '/';
};

const Login: React.FC<{}> = () => {
  const [submitting, setSubmitting] = useState(false);
  const [userLoginState, setUserLoginState] = useState<API.LoginStateType>({});
  const [type, setType] = useState<string>('account');
  const intl = useIntl();

  const handleSubmit = async (values: LoginParamsType) => {
    setSubmitting(true);
    try {
      // 登录
      const msg = await fakeAccountLogin({ ...values, type });
+      if (msg.status === 'ok' && msg.token) {
+        localStorage.setItem('token',msg.token);
        message.success('登录成功！');
        goto();
        return;
      }
      // 如果失败去设置用户错误信息
      setUserLoginState(msg);
    } catch (error) {
      message.error('登录失败，请重试！');
    }
    setSubmitting(false);
  };
  const { status, type: loginType } = userLoginState;

  return (
    <div className={styles.container}>
      <div className={styles.lang}>{SelectLang && <SelectLang />}</div>
      <div className={styles.content}>
        <div className={styles.top}>
          <div className={styles.header}>
            <Link to="/">
              <img alt="logo" className={styles.logo} src="/logo.svg" />
              <span className={styles.title}>Ant Design</span>
            </Link>
          </div>
          <div className={styles.desc}>Ant Design 是西湖区最具影响力的 Web 设计规范</div>
        </div>

        <div className={styles.main}>
          <ProForm
            initialValues={{
              autoLogin: true,
            }}
            submitter={{
              searchConfig: {
                submitText: intl.formatMessage({
                  id: 'pages.login.submit',
                  defaultMessage: '登录',
                }),
              },
              render: (_, dom) => dom.pop(),
              submitButtonProps: {
                loading: submitting,
                size: 'large',
                style: {
                  width: '100%',
                },
              },
            }}
            onFinish={async (values) => {
              handleSubmit(values);
            }}
          >
            <Tabs activeKey={type} onChange={setType}>
              <Tabs.TabPane
                key="account"
                tab={intl.formatMessage({
                  id: 'pages.login.accountLogin.tab',
                  defaultMessage: '账户密码登录',
                })}
              />
            </Tabs>

            {status === 'error' && loginType === 'account' && (
              <LoginMessage
                content={intl.formatMessage({
                  id: 'pages.login.accountLogin.errorMessage',
                  defaultMessage: '账户或密码错误（admin/ant.design)',
                })}
              />
            )}
            {type === 'account' && (
              <>
                <ProFormText
                  name="username"
                  fieldProps={{
                    size: 'large',
                    prefix: <UserOutlined className={styles.prefixIcon} />,
                  }}
                  placeholder={intl.formatMessage({
                    id: 'pages.login.username.placeholder',
                    defaultMessage: '用户名: admin or user',
                  })}
                  rules={[
                    {
                      required: true,
                      message: (
                        <FormattedMessage
                          id="pages.login.username.required"
                          defaultMessage="请输入用户名!"
                        />
                      ),
                    },
                  ]}
                />
                <ProFormText.Password
                  name="password"
                  fieldProps={{
                    size: 'large',
                    prefix: <LockTwoTone className={styles.prefixIcon} />,
                  }}
                  placeholder={intl.formatMessage({
                    id: 'pages.login.password.placeholder',
                    defaultMessage: '密码: ant.design',
                  })}
                  rules={[
                    {
                      required: true,
                      message: (
                        <FormattedMessage
                          id="pages.login.password.required"
                          defaultMessage="请输入密码！"
                        />
                      )
                    },
                  ]}
                />
              </>
            )}
            <div style={{marginBottom: 24}}></div>
          </ProForm>
        </div>
      </div>
      <Footer />
    </div>
  );
};

export default Login;
```
## 3.后端 
### 3.1 安装依赖
```sh
cnpm i express body-parser  jwt-simple cors express-session connect-mongo mongoose axios -S
```
```sh
/api/register
{"name":"admin","password":"123456","autoLogin":true,"type":"account"}
/api/login/account
{"username":"admin","password":"123456"}
```
### 3.2 app.js
```js
let express = require("express");
let bodyParser = require("body-parser");
let jwt = require('jwt-simple');
let cors = require("cors");
let Models = require('./model');
let session = require("express-session");
let MongoStore = require('connect-mongo')(session);
let config = require('./config');
let app = express();
app.use(
    cors({
        origin: config.origin,
        credentials: true,
        allowedHeaders: "Content-Type,Authorization",
        methods: "GET,HEAD,PUT,PATCH,POST,DELETE,OPTIONS"
    })
);
app.use(bodyParser.urlencoded({ extended: false }));
app.use(bodyParser.json());
app.use(
    session({
        secret: config.secret,
        resave: false,
        saveUninitialized: true,
        store: new MongoStore({
            url: config.dbUrl,
            mongoOptions: {
                useNewUrlParser: true,
                useUnifiedTopology: true
            }
        })
    })
);
app.get('/', async (req, res) => {
    res.json({ code: 0, data: `hello` });
});

app.post('/api/register', async (req, res) => {
    let user = req.body;
    let hash = require('crypto').createHash('md5').update(user.email).digest('hex');
    user.avatar = `https://secure.gravatar.com/avatar/${hash}?s=48`;
    user = await Models.UserModel.create(user);
    res.send({ status: 'ok', currentAuthority: 'user' });
});
app.post('/api/login/account', async (req, res) => {
    let user = req.body;
    let query = {};
    if (user.type == 'account') {
        query.name = user.username;
        query.password = user.password;
    }
    let dbUser = await Models.UserModel.findOne(query);
    if (dbUser) {
        dbUser.userid = dbUser._id;
        let token = jwt.encode(dbUser, config.secret);
        return res.send({ status: 'ok', token, type: user.type, currentAuthority: dbUser.currentAuthority });
    } else {
        return res.send({
            status: 'error',
            type: user.type,
            currentAuthority: 'guest'
        });
    }
});

app.get('/api/currentUser', async (req, res) => {
    let authorization = req.headers['authorization'];
    if (authorization) {
        try {
            let user = jwt.decode(authorization.split(' ')[1], config.secret);
            res.json(user);
        } catch (err) {
            res.status(401).send({});
        }
    } else {
        res.status(401).send({});
    }
});
app.get('/api/login/outLogin', async (req, res) => {
    res.send({ data: {}, success: true });
});
app.listen(4000, () => {
    console.log('服务器在4000端口启动!');
});
```
### 3.3 model.js
```js
const mongoose = require('mongoose');
const Schema = mongoose.Schema;
let config = require('./config');
const conn = mongoose.createConnection(config.dbUrl, { useNewUrlParser: true, useUnifiedTopology: true });
const UserModel = conn.model('User', new Schema({
    userid: { type: String },
    email: { type: String },//邮箱
    name: { type: String },//用户名
    password: { type: String, required: true },//密码
    avatar: { type: String, required: true },//头像
    currentAuthority: { type: String, required: true,default:'user' }//当前用户的权限
}));

module.exports = {
    UserModel
}
```
### 3.4 config.js
```js
module.exports = {
    secret: 'pro',
    dbUrl: "mongodb://localhost:27017/pro",
    origin: ["http://localhost:8000"]
}
```
### 3.5 package.json
```json
  "scripts": {
    "start": "nodemon app.js"
  }
```