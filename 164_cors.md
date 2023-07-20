## 1.什么是跨域？

### 1.1 URL

URL(Uniform Resource Locator)是互联网上的一种资源的简洁标识。它是一种特定格式的字符串，可以指向互联网上的任何资源。

以下是一个 URL 的完整示例: [http://username:password@www.example.com:80/path/to/myfile.html?key1=value1&key2=value2#SomewhereInTheDocument](http://username:password@www.example.com:80/path/to/myfile.html?key1=value1&key2=value2#SomewhereInTheDocument)

这个 URL 的各个部分具有以下含义：

- `http`：这部分被称为协议或方案。它定义了如何访问和互动资源。常见的协议有 HTTPS, HTTP, FTP, FILE 等。
- `user:pass` 这是可选部分，用于需要身份验证的服务。
- `site.com` 这部分被称为主机名或者域名。它定义了我们想要访问的服务器的地址。这可以是一个 IP 地址或者一个注册的域名。
- `:80` 这部分是可选的，称为端口号。它定义了服务器上的哪个服务我们要访问。如果未指定，那么默认端口是协议的标准端口（例如，对于 HTTP 是 80，HTTPS 是 443）。
- `/path/xxx` 这部分是路径，它指定了服务上的哪个具体资源我们想要访问。
- `?q=val` 这部分是查询字符串，用于发送参数到服务器。它以问号开始，参数以键值对的形式存在，并用&符号分割。
- `#hash` 这部分被称为片段或者锚点，它指定了网页中的一个位置。当你访问一个 URL 时，浏览器会尝试滚动到这个位置。

![](/public/images/url_1684216406780.webp)

### 1.2 什么是跨域？

"跨域"是指浏览器为了安全性，设置的同源策略限制

同源策略是一种约定，它是浏览器的一种安全功能，只允许 web 页面请求同一个源（协议，域名和端口）的资源

换句话说，如果在浏览器中运行的网页试图请求来自不同源的资源，就会发生”跨域“，这通常会被浏览器所禁止。

以下是几种跨域的情况：

1. 网站的 URL 协议不同。例如，一个用 HTTP 协议，一个用 HTTPS 协议
2. 网站的域名不同。例如，一个是 [www.example.com](), 一个是[www.test.com]()。
3. 网站的端口不同。例如，一个是 [www.example.com:8080](), 一个是 [www.example.com:9090]()。

## 2. JSONP

### 2.1 什么是 JSONP

JSONP 是 ”JSON with Padding“的缩写，这是一个非官方的通信协议，允许在服务器之间进行数据交换，为了解决同源策略问题。这是一种用于处理跨源数据请求的方法。

在同源策略中，基于安全考虑，一个 Web 页面上的 JS 脚本只能访问与它同源的数据。如果要访问其他源的数据，就会遇到同源策略的限制。

那么，如何突破这个限制呢？其中一种方式就是使用 JSONP。事实上，尽管同源策略限制了 XMLHttpRequest 的跨域请求，但 `<script>`标签的 src 属性却不受此限制，因此我们可以使用 script 元素来实现跨域请求。

### 2.2 JSONP 工作原理

JSONP 工作原理是这样的：

1. 客户端通过`<script>`标签向服务器发送请求，同时定义好回调函数。
2. 服务器接收到请求后，将数据填充进这个回调函数中，然后将这个函数返回给客户端。
3. 客户端接收到数据后，因为这个数据实际上是一个函数调用，所以会立即执行这个函数，从而可以获取到服务器返回的数据。

### 2.3 百度搜索

下面示例为一个搜索输入框添加自动补全功能。它会监听用户在输入框中的输入事件，然后根据用户的输入内容发起 jsonp 请求到百度的搜索建议接口，获取搜索建议，然后将这些建议显示在页面上。

```
https://www.baidu.com/sugrec?prod=pc&wd=a&cb=myCallback
```

#### 2.3.1 html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  </head>
  <body>
    <input id="search-input" type="text" placeholder="搜索关键词" />
    <ul id="suggestion-list"></ul>
    <script>
      // 定义 jsonp 函数，用于发起 jsonp 请求
      function jsonp({ url, jsonp, data }) {
        // 返回一个新的 Promise 对象
        return new Promise((resolve, reject) => {
          // 创建一个唯一的回调函数名，使用当前时间戳
          let callbackName = `jQuery_${Date.now()}`;
          // 创建一个 script 标签
          let script = document.createElement("script");
          // 在全局对象 window 上创建一个新的函数，函数名是我们前面创建的 callbackName
          window[callbackName] = function (result) {
            // 请求成功后，删除这个全局函数
            delete window[callbackName];
            document.body.removeChild(script);
            // 使用请求到的结果来解决这个 Promise
            resolve(result);
          };
          // 生成请求的查询字符串
          let queryStr = url.indexOf("?") === -1 ? "?" : "&";
          for (let key in data) {
            queryStr += `${key}=${data[key]}&`;
          }
          // 设置 script 标签的 src 属性，发起 jsonp 请求
          script.src = `${url}${queryStr}${jsonp}=${callbackName}`;
          // 将这个 script 标签添加到页面中
          document.body.appendChild(script);
        });
      }
      // 为页面上的搜索输入框添加一个输入事件的监听器
      document
        .getElementById("search-input")
        .addEventListener("input", function (e) {
          // 在输入事件发生时，发起 jsonp 请求
          jsonp({
            url: "http://localhost:3000/sugrec",
            jsonp: "cb",
            data: {
              prod: "pc",
              wd: e.target.value,
            },
          }).then((response) => {
            // 从返回的结果中获取建议列表
            const suggestions = response.g || [];
            // 获取显示建议列表的元素
            const suggestionList = document.getElementById("suggestion-list");
            // 生成建议列表的 HTML 字符串
            let html = "";
            for (let i = 0; i < suggestions.length; i++) {
              html += `<li>${suggestions[i].q}</li>`;
            }
            // 更新建议列表的 HTML 内容
            suggestionList.innerHTML = html;
          });
        });
    </script>
  </body>
</html>
```

#### 2.3.2 server.js

```js
// 导入 Express.js 框架
const express = require("express");
// 创建一个 Express.js 应用实例
const app = express();
// 定义服务器要监听的端口号
const port = 3000;
// 为 '/sugrec' 路径设置一个 GET 请求的处理器
app.get("/sugrec", (req, res) => {
  // 从请求的查询参数中获取 'cb' 和 'wd' 参数
  const { cb, wd } = req.query;
  // 根据 'wd' 参数创建一个响应数据对象
  const result = {
    g: Array.from({ length: 10 }, (_, i) => ({ q: `${wd}${i + 1}` })),
  };
  // 设置响应的 Content-Type 为 'text/javascript'
  res.type("text/javascript");
  // 发送一个 JavaScript 脚本作为响应，该脚本调用 'cb' 参数指定的函数，并将响应数据作为参数传入
  res.send(`${cb}(${JSON.stringify(result)})`);
});

app.listen(port, () => {
  // 当服务器开始监听后，输出一条消息到控制台
  console.log(`Server is running on http://localhost:${port}`);
});
```

虽然 JSONP 是一种解决跨域请求的有效方法，但是它也有一些安全风险。由于它是通过插入 `<script>` 标签来实现的，因此可能会被恶意站点利用。如果返回的数据中包含了恶意代码，那么这些代码也会被执行。因此，在使用 JSONP 时，需要确保你的数据源是可信的。

## 3. CORS

CORS, 即跨源资源共享（Cross-Origin Resource Sharing），是一种用于解决 AJAX 请求在 web 应用中跨域问题的技术。由于浏览器的同源策略限制，一个 Web 页面只能向与该页面相同域名、协议和端口的服务器发出 HTTP 请求。如果向不同的源发送请求，就会触发同源策略的限制。CORS 技术通过添加特殊的 HTTP 头，允许浏览器和服务器进行跨域通信。

当发出一个跨域请求时，浏览器会首先发送一个预检（preflight）请求，这是一个 OPTIONS 请求，询问目标服务器是否允许当前源进行跨域请求。服务器通过相应的 HTTP 头信息告诉浏览器其支持的 HTTP 方法、允许的源等信息。

服务器通过以下 HTTP 头来控制 CORS:

| 响应头名称                       | 含义                                                                                                                                                                                                           | 示例                                                                       |
| -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| Access-Control-Allow-Origin      | 指定哪些源（协议+域名+端口） 可以访问资源。如果设置为"\*",则表示所有源都可以访问。                                                                                                                             | Access-Control-Allow-Origin:[https://example.com]()                        |
| Access-Control-Allow-Methods     | 指定服务器支持的 HTTP 方法。多个方法以逗号分隔。                                                                                                                                                               | Access-Control-Allow-Methods: GET, POST, PUT                               |
| Access-Control-Allow-Headers     | 指定服务器接收的自定义请求头。多个头以逗号分隔。                                                                                                                                                               | Access-Control-Allow-Headers: X-My-Custom-Header, Content-Type             |
| Access-Control-Allow-Credentials | 指定是否允许携带凭据（如 Cookie 或 HTTP 认证信息）进行跨域请求。如果设置为 "true", 则表示可以携带凭据。需要注意的是，如果这个头设置为"true", Access-Control-Allow-Origin 则不能设置为 "\*", 必须指定具体的源。 | Access-Control-Allow-Credentials: true                                     |
| Access-Control-Expose-Headers    | 指定浏览器可以访问的服务器的响应头列表。多个头以逗号分隔。                                                                                                                                                     | Access-Control-Expose-Headers: X-My-Custom-Header, X-Another-Custom-Header |
| Access-Control-Max-Age           | 指定预检请求的结果（即服务器的 CORS 配置）可以被缓存多久，单位是秒。在这段时间内，对同一资源的请求将不再触发预检请求。                                                                                         | Access-Control-Max-Age: 86400                                              |

### 3.1 GET

#### 3.1.1 Access-Control-Allow

`Access-Control-Allow-Origin`: 这个响应头定义了哪些源(Origin) 有权通过他们的前端代码来访问你的资源。源包括协议(http, https), 主机（例如，www.example.com） 以及端口号（例如，8080）。这个响应头的值可以是特定的值，例如 `https://www.example.com`, 或者 `*`，代表所有源。

`Access-Control-Allow-Headers`: 当实际请求的头部信息不在默认支持的头部列表中，浏览器在发送实际请求之前会发送预检请求，此响应头用于在预检请求的响应中，告诉浏览器实际的请求可以使用哪些 HTTP 头。例如，如果你设置 `Access-Control-Allow-Headers: 'Content-Type, X-Api-Key'`, 这将表示请求可以包含 `Content-Type` 和 `X-Api-Key` 这两个头。

`Access-Control-Expose-Headers`: 这个响应头让服务器把响应的头暴露给浏览器处理，并且这些头信息在脚本中可通过 `XMLHttpRequest.getResponseHeader()` 方法进行访问。

#### 3.1.2 servera.js

src\2.cors\servera.js

```js
// 引入 Express 框架
const express = require("express");
// 创建一个 Express 应用
const app = express();
// 设置静态文件夹
app.use(express.static("public"));
// 定义监听端口
const port = 3000;
// 让应用监听在指定端口并在控制台输出信息
app.listen(3000, () => {
  // 当服务器开始运行时，打印一条消息
  console.log(`Server is running on http://localhost:${port}`);
});
```

#### 3.1.3 index.html

src\2.cors\public\index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>a</title>
  </head>
  <body>
    <script>
      // 定义一个立即执行的异步函数
      (async function () {
        try {
          // 使用 fetch API 发送 GET 请求到指定的 URL
          const response = await fetch("http://localhost:4000/users", {
            // 指定请求方法为 GET
            method: "GET",
            // 指定请求头，表明期望的响应格式为 JSON
            headers: {
              Accept: "application/json",
            },
            // 使用 then 方法将响应对象转换为 JSON 格式
          }).then((response) => {
            for (let [key, value] of response.headers) {
              console.log(`${key}: ${value}`);
            }
            console.log(response.headers.get("content-type"));
            return response.json();
          });
          // 打印获取到的响应
          console.log("response", response);
        } catch (error) {
          // 如果有任何错误，打印错误信息
          console.log("Error", error);
        }
        // 立即执行上述定义的函数
      })();
    </script>
  </body>
</html>
```

#### 3.1.4 serverb.js

src\2.cors\serverb.js

```js
// 引入 Express 框架
const express = require("express");
// 创建一个 Express 应用
const app = express();
// 使用中间件来设置响应头，以处理跨域问题
app.use((req, res, next) => {
  // 允许所以来源的访问
  res.header("Access-Control-Allow-Origin", "*");
  // 允许接受的请求头
  res.header("Access-Control-Allow-Headers", "Accept");
  // 指定对外暴露的响应头
  res.header("Access-Control-Expose-Headers", "X-My-Custom-Header");
  // 设置自定义的响应头
  res.setHeader("X-My-Custom-Header", "X-My-Custom-Header");
  // 调用 next 函数, 以便将控制权交给下一个中间件
  next();
});
// 定义一个用户列表
const users = [{ id: 1, name: "用户1" }];

// 创建一个端点，返回用户列表
app.get("/users", (req, res) => {
  // 将用户列表以 JSON 格式返回
  res.json(users);
});
// 定义监听端口
const port = 4000;
// 让应用监听在指定端口并在控制台输出信息
app.listen(port, () => {
  // 当服务器开始运行时，打印一条消息
  console.log(`Server is running on http://localhost:${port}`);
});
```

### 3.2 POST

#### 3.2.1 Access-Control-Max-Age

`Access-Control-Max-Age`: 这个响应头定义了预检请求结果的缓存时间。预检请求是一种由浏览器自动发出，用来检查服务器是否接受真正的请求的请求。这个值以秒为单位，代表该预检请求结果能够被缓存多久。在这个时间段内，浏览器无需为同样的请求再次发送预检请求。

```
res.header("Access-Control-Max-Age", "3600");
```

#### 3.2.2 add.html

src\2.cors\public\add.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>a</title>
  </head>
  <body>
    <script>
      // 定义一个立即执行的异步函数
      (async function () {
        try {
          // 使用 fetch API 发送 GET 请求到指定的 URL
          const response = await fetch("http://localhost:4000/users", {
            // 指定请求方法为 GET
            method: "POST",
            // 指定请求头，表明期望的响应格式为 JSON
            headers: {
              "Content-Type": "application/json",
              Accept: "application/json",
            },
            body: JSON.stringify({ name: "李四" }),
            // 使用 then 方法将响应对象转换为 JSON 格式
          }).then((response) => response.json());
          // 打印获取到的响应
          console.log("response ", response);
        } catch (error) {
          // 如果有任何错误，打印错误信息
          console.error("Error:", error);
        }
        // 立即执行上述定义的函数
      })();
    </script>
  </body>
</html>
```

#### 3.2.3 serverb.js

src\2.cors\serverb.js

```js
// 引入 Express 框架
const express = require('express');
// 创建一个 Express 应用
const app = express();
+// 设置静态文件夹
+app.use(express.static('public'));
+// 使用中间件来解析请求体中的 JSON 数据
+app.use(express.json());
+// 使用中间件来解析请求体中的 urlencoded 数据
+app.use(express.urlencoded({ extended: false }));
// 使用中间件来设置响应头，以处理跨域问题
app.use((req, res, next) => {
    // 允许所有来源的访问
    res.header("Access-Control-Allow-Origin", "*");
    // 允许接受的请求头
+   res.header("Access-Control-Allow-Headers", "Accept,Content-Type");
    // 指定对外暴露的响应头
    res.header('Access-Control-Expose-Headers', 'X-My-Custom-Header');
    // 设置自定义的响应头
    res.setHeader('X-My-Custom-Header','X-My-Custom-Header');
    // 设置预检请求的结果可以被缓存多久
+   res.header("Access-Control-Max-Age", "3600");
+   if(req.method === 'OPTIONS'){
+       return res.sendStatus(200);
+   }
   // 调用 next 函数，以便将控制权交给下一个中间件
    next();
});
// 定义一个用户列表
const users = [
    {id:1,name:'张三'}
];
// 创建一个端点，返回用户列表
app.get('/users', (req, res) => {
    // 将用户列表以 JSON 格式返回
    res.json(users);
});
+app.post('/users', (req, res) => {
+    const user = req.body;
+    user.id = users[users.length-1].id + 1;
+    users.push(user);
+    res.json(users);
+});
// 定义监听端口
const port = 4000;
// 让应用监听在指定端口并在控制台输出信息
app.listen(port, () => {
    // 当服务器开始运行时，打印一条消息
    console.log(`Server is running on http://localhost:${port}`);
});
```

### 3.3 Credentials

#### 3.3.1 cookie

Cookie 是一种在客户端存储信息的机制，它通常用于保存用户偏好或者跟踪用户行为。HTTP Cookie，也叫网页饼干，是服务器发送到用户浏览器并保存在本地的一小片数据，它会在浏览器下一次向同一服务器再发起请求时被携带并发送到服务器上。

一些重要的基本概念和属性:

- **名称和值**：每个 Cookie 都有一个名称和一个值。这是实际保存的数据。
- **过期时间**：每个 Cookie 都有一个过期日期。如果没有设置过期日期，那么这个 Cookie 就是会话 Cookie，会话 Cookie 在浏览器关闭时会被自动删除。如果设置了过期日期，那么这个 Cookie 就是持久 Cookie，它会在过期日期之后被删除。
- **路径**：可以为每个 Cookie 设置一个路径。这个路径决定了在哪些页面上这个 Cookie 可以被读取。如果路径设置为'/'，那么这个 Cookie 在整个网站上都是可用的。
- **域**：可以为每个 Cookie 设置一个域。Cookie 只能被设置为创建它的页面的同源域名读取。不过，如果设置了域，那么这个 Cookie 就可以在整个域中被读取。
- **Secure**：标记为 Secure 的 Cookie 只能通过 HTTPS 协议加密传输。这样可以防止 Cookie 被嗅探者窃取。
- **HttpOnly**：标记为 HttpOnly 的 Cookie 不能被 JavaScript 通过 Document.cookie 等属性访问，只能由服务器读取。这样可以防止 Cookie 被跨站脚本（XSS）攻击窃取。

在 JavaScript 中，可以通过`document.cookie`来获取和设置 Cookie。例如：

```js
// 设置一个cookie
document.cookie =
  "username=John Doe; expires=Thu, 18 Dec 2023 12:00:00 UTC; path=/";

// 读取所有cookie
console.log(document.cookie);
```

#### 3.3.2 Access-Control-Allow-Credentials

`Access-Control-Allow-Credentials`: 这个响应头表示是否允许发送 Cookie。只有当设置为 `true` 时，浏览器才会在 CORS 请求中包含凭据（如 Cookies 或 HTTP 认证信息）。同时，浏览器端的请求也必须设置 `withCredentials = true`。这样，服务器在响应中就可以根据 Cookie 进行用户识别。
注意，如果这个值被设置为 `true`, `Access-Control-Allow-Origin`的值不能为 `*`,必须指定明确的源。

#### 3.3.3 serverb.js

src\2.cors\serverb.js

```js
// 引入 Express 框架
const express = require('express');
+// 引入 cookie-parser 中间件，用于处理 Cookie
+const cookieParser = require('cookie-parser');
// 创建一个 Express 应用
const app = express();
// 设置静态文件夹
app.use(express.static('public'));
+// 使用 cookie-parser 中间件
+app.use(cookieParser());
// 使用中间件来解析请求体中的 JSON 数据
app.use(express.json());
// 使用中间件来解析请求体中的 urlencoded 数据
app.use(express.urlencoded({ extended: false }));
// 使用中间件来设置响应头，以处理跨域问题
app.use((req, res, next) => {
    // 允许所有来源的访问
+   res.header("Access-Control-Allow-Origin", req.headers.origin);
    // 允许接受的请求头
    res.header("Access-Control-Allow-Headers", "Accept,Content-Type");
    // 指定对外暴露的响应头
    res.header('Access-Control-Expose-Headers', 'X-My-Custom-Header');
    // 设置自定义的响应头
    res.setHeader('X-My-Custom-Header','X-My-Custom-Header');
    // 设置预检请求的结果可以被缓存多久
    res.header("Access-Control-Max-Age", "3600");
    // 允许发送 Cookie
+   res.header("Access-Control-Allow-Credentials", "true");
    if(req.method === 'OPTIONS'){
        return res.sendStatus(200);
    }
    // 调用 next 函数，以便将控制权交给下一个中间件
    next();
});
// 定义一个用户列表
const users = [
    {id:1,name:'张三'}
];
// 创建一个端点，返回用户列表
app.get('/users', (req, res) => {
    // 将用户列表以 JSON 格式返回
    res.json(users);
});
app.post('/users', (req, res) => {
    const user = req.body;
    user.id = users[users.length-1].id + 1;
    users.push(user);
    res.json(users);
});
+// 创建一个路由，用于计数
+app.get('/count', (req, res) => {
+    // 从请求的 Cookies 中获取 "count" 的值，如果不存在，则默认为 0
+    let count = req.cookies.count || 0;
+    // 将计数器的值加 1
+    count++;
+    // 在响应的 Cookies 中设置 "count" 的值，同时设置最大过期时间为 24 小时，并设置只能通过 HTTP 访问
+    res.cookie('count', count, { maxAge: 24 * 60 * 60 * 1000 });
+    // 将计数器的值以 JSON 格式返回给客户端
+    res.json({ count });
+});
// 定义监听端口
const port = 4000;
// 让应用监听在指定端口并在控制台输出信息
app.listen(port, () => {
    // 当服务器开始运行时，打印一条消息
    console.log(`Server is running on http://localhost:${port}`);
});
```

#### 3.3.4 count.html

src\2.cors\public\count.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <p id="notice"></p>
    <script>
      // 使用 fetch API 发送 GET 请求到计数端点
      async function getCount() {
        try {
          const response = await fetch("http://localhost:4000/count", {
            method: "GET",
            credentials: "include", // 需要包含此行以发送 Cookie
          }).then((response) => response.json());
          document.getElementById(
            "notice"
          ).innerHTML = `Count: ${response.count}`;
        } catch (error) {
          console.error("Error:", error);
        }
      }
      getCount();
    </script>
  </body>
</html>
```

## 4. postMessage

`postMessage` 是 Web API 的一部分，用于在不同的浏览器上下文之间进行安全的跨源通信，如不同的窗口、iframe 或者和工作线程之间的通信。
这里是如何使用 `postMessage` 实现跨域的基本步骤：

1. **发送消息**：你可以通过调用 `otherWindow.postMessage(message, targetOrigin)`来发送消息。`otherWindow`是一个对其他窗口的引用，如 iframe 的 `contentWindow` 属性、执行窗口打开的结果，或是命名窗口。`message` 是你要发送的数据，可以是任何结构化克隆算法能处理的 JavaScript 数据类型。`targetOrigin` 是一个字符串，指定了接受消息的文档必须来自哪个源。

2. **接受消息**：在接收窗口，你需要监听 `message` 事件来接收消息。事件处理函数将会接收一个 `MessageEvent` 实例，这个实例有几个有用的属性，包括 `data` (发送的消息数据)、`origin` (发送消息的文档源) 和 `source` (发送消息的`window`对象)。

### 4.1 servera.js

src\3.postMessage\servera.js

```js
// 引入 Express 框架
const express = require("express");
// 创建一个 Express 应用
const app = express();
// 设置静态文件夹
app.use(express.static("public"));
// 定义监听端口
const port = 3000;
// 让应用监听在指定端口并在控制台输出信息
app.listen(port, () => {
  // 当服务器开始运行时，打印一条消息
  console.log(`Server is running on http://localhost:${port}`);
});
```

### 4.2 a.html

src\3.postMessage\public\a.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>a</title>
  </head>
  <body>
    <!-- 在 HTML 中插入一个 iframe 元素，id 是 "b-iframe"，指向的源是 "http://localhost:4000/b.html"，并在 iframe 加载完成后调用 "handleIframeLoad()" 函数 -->
    <iframe
      id="b-iframe"
      src="http://localhost:4000/b.html"
      onload="handleIframeLoad()"
    ></iframe>
    <script>
      // 定义目标源（targetOrigin），即 iframe 的源
      const targetOrigin = "http://localhost:4000";
      // 定义 iframe 加载完成后的处理函数
      function handleIframeLoad() {
        // 获取 iframe 的 Window 对象
        let bWindow = document.getElementById("b-iframe").contentWindow;
        // 使用 postMessage 向 iframe 发送消息
        bWindow.postMessage("hello", targetOrigin);
      }
      // 在当前窗口添加 message 事件监听器，用于处理从其他源发送过来的消息
      window.addEventListener(
        "message",
        function (event) {
          // 输出接收到的消息
          console.log("Received message: ", event.data);
        },
        false
      );
    </script>
  </body>
</html>
```

### 4.3 serverb.js

src\3.postMessage\serverb.js

```js
// 引入 Express 框架
const express = require("express");
// 创建一个 Express 应用
const app = express();
// 设置静态文件夹
app.use(express.static("public"));
// 定义监听端口
const port = 4000;
// 让应用监听在指定端口并在控制台输出信息
app.listen(port, () => {
  // 当服务器开始运行时，打印一条消息
  console.log(`Server is running on http://localhost:${port}`);
});
```

### 4.4 b.html

src\3.postMessage\public\b.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>b</title>
  </head>
  <body>
    <script>
      // 在当前窗口添加 message 事件监听器，用于处理从其他源发送过来的消息
      window.addEventListener(
        "message",
        function (event) {
          console.log(event.source); //Window
          console.log(event.origin); //http://localhost:3000
          // 使用 postMessage 向发送消息的源返回消息
          event.source.postMessage("world", event.origin);
        },
        false
      );
    </script>
  </body>
</html>
```

## 5.websocket

WebSocket 是一种网络通信协议，提供了全双工(full-duplex)通信渠道，也就是说，服务器和客户端可以同时进行数据传输，而不需要像 HTTP 请求那样由客户端先发起请求。WebSocket 在单个 TCP 连接上提供持久的连接，以便在客户端和服务器之间进行实时通信。

这是 WebSocket 的基本使用方法：

1. **创建 WebSocket 连接**：首先，你需要在浏览器端创建一个新的 webSocket 对象，向其传入你想要连接的服务器的 WebSocket 地址。

```js
const socket = new WebSocket("ws://www.example.com/socketserver");
```

2. **处理连接事件**：你可以监听几个重要的事件，例如`open`、`message`、`error`和`close`。

- 当 WebSocket 连接成功时， `open` 事件将会被触发。
- 当 从服务器接收到数据时， `message` 事件将会被触发。
- 当 发送错误时，`error` 事件将会被触发。
- 当 连接关闭时， `close` 事件将会被触发。

```js
socket.onopen = function (event) {
  console.log("Connection open");
};

socket.onmessage = function (event) {
  console.log("Received data: " + event.data);
};

socket.onerror = function (error) {
  console.log("Error detected: " + error);
};

socket.onclose = function (event) {
  console.log("Connection closed");
};
```

3. **发送和接收数据**：通过 WebSocket 连接，你可以使用 `send` 方法发送数据到服务器，服务器也可以随时发送数据到客户端。

```js
// Sending a text string
socket.send("Hello, server!");

// Sending a JavaScript object, need to convert it to a JSON string
socket.send(JSON.stringify({ key: "value" }));
```

4. **关闭连接**：当你不再需要 WebSocket 连接时，应当关闭它，以释放资源。

```js
socket.close();
```

### 5.1 servera.js

src\4.websocket\servera.js

```js
// 引入 Express 框架
const express = require("express");
// 创建一个 Express 应用
const app = express();
// 设置静态文件夹
app.use(express.static("public"));
// 定义监听端口
const port = 3000;
// 让应用监听在指定端口并在控制台输出信息
app.listen(port, () => {
  // 当服务器开始运行时，打印一条消息
  console.log(`Server is running on http://localhost:${port}`);
});
```

### 5.2 a.html

src\4.websocket\public\a.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>a</title>
  </head>
  <body>
    <script>
      // 创建一个新的 WebSocket 对象，连接到指定的URL
      const socket = new WebSocket("ws://localhost:4000");
      // 设置连接打开时的回调函数
      socket.onopen = function (event) {
        // 在控制台打印消息，表示连接已经打开
        console.log("Connetion open");
        // 通过WebSocket连接发送一条消息，消息内容为'hello'
        socket.send("hello");
      };
      // 设置接收到服务器消息时的回调函数
      socket.onmessage = function (event) {
        // 在控制台打印接收到的消息
        console.log("Received data: " + event.data);
      };
    </script>
  </body>
</html>
```

### 5.3 serverb.js

src\4.websocket\serverb.js

```js
// 导入 'ws' 模块
const WebSocket = require("ws");
// 创建一个新的 WebSocket 服务器，监听 4000 端口
const server = new WebSocket.Server({ port: 4000 });
// 当有新的客户端连接时，为这个客户端添加消息监听器
server.on("connection", (ws) => {
  // 当从客户端收到消息时
  ws.on("message", (message) => {
    // 在控制台打印收到的消息
    console.log("Received: " + message);
    // 向客户端发送 'world' 消息
    ws.send("world");
  });
});
// 在控制台输出 WebSocket 服务器启动的信息

console.log("WebSocket server is running on http://localhost:4000");
```

## 6. nginx

- [Nginx](http://nginx.org/en/download.html)是一个开源的、高性能的、可扩展的 HTTP 和反向代理服务器。它也可以用作邮件代理服务器和通用的 TCP/UDP 代理服务器。

在 Windows 环境下，Nginx 命令和在类 Unix 系统中略有不同。一般来说，你需要打开命令提示符(cmd)或 Powershell 窗口，然后切换到你 Nginx 程序的安装目录下进行操作。以下是一些基本的 Nginx 命令：

1. 启动 **Nginx**: 直接运行 nginx.exe 文件即可。你需要在命令提示符中切换到 Nginx 的目录，然后运行以下命令：

```
start nginx
```

2. 停止 **Nginx**: 运行以下命令：

```
nginx -s stop
```

3. 重启 **Nginx**: 首先停止 Nginx, 然后再次运行 nginx.exe 文件:

```
nginx -s stop
start nginx
```

4. 重载 **Nginx 配置**：如果你更改了 Nginx 的配置文件，可以使用这个命令来使更改生效，而无需完全停止服务：

```
nginx -s reload
```

注意，以上的命令都需要在 Nginx 的安装目录中运行。例如，如果你的 Nginx 安装在 `C:\nginx` 目录，那么你需要先在命令提示符中运行 `cd C:\nginx` 命令，然后再运行上述命令。

### 6.1 HTTP 服务器

**HTTP 服务器**： Nginx 可以作为一个 HTTP 服务器，提供静态资源服务，也可以作为应用服务器（例如 PHP,Python 等）的前端服务器。

要使用 Nginx 来处理跨域问题，你需要配置一些响应头。你可以在 Nginx 的配置文件中进行配置，通常，这个配置文件位于 `/etc/nginx/nginx.conf` 或 `/etc/nginx/sites-available/default`

#### 6.1.1 add_header

- [add_header](http://nginx.org/en/docs/http/ngx_http_headers_module.html#add_header)

- ~ 符号表示接下来的是一个正则表达式。 Nginx 会使用这个正则表达式来匹配请求的 URL 路径。

- .json$ 是一个正则表达式，它匹配任何以 .json 结尾的 URL 路径

```
server {
  location ~ \.json$ {
    root  data;
    add_header  'Access-Control-Allow-Origin' '*';
  }
}
```

#### 6.1.2 users.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>a</title>
  </head>

  <body>
    <script>
      // 定义一个立即执行的异步函数
      (async function () {
        try {
          // 使用 fetch API 发送 GET 请求到指定的 URL
          const response = await fetch("http://localhost:8080/users.json", {
            // 指定请求方法为 GET
            method: "GET",
            // 指定请求头，表明期望的响应格式为 JSON
            headers: {
              Accept: "application/json",
            },
            // 使用 then 方法将响应对象转换为 JSON 格式
          }).then((response) => response.json());
          // 打印获取到的响应
          console.log("response ", response);
        } catch (error) {
          // 如果有任何错误，打印错误信息
          console.error("Error:", error);
        }
        // 立即执行上述定义的函数
      })();
    </script>
  </body>
</html>
```

#### 6.1.3 users.json

```json
[{ "id": 1, "name": "张三" }]
```

### 6.2 跨域代理

![](/public/images/kua_yu_dai_li_1687600216717.svg)

- proxy_pass [http://localhost:4000](); 是一个指令，它告诉 Nginx 将所有匹配的请求代理到 [http://localhost:4000]()

#### 6.2.1 nginx

```
server {
  listen 8080;
  location /api {
    proxy_pass http://localhost:4000;
  }
}
```

#### 6.2.2 serverb.js

src\5.nginx\serverb.js

```js
const express = require("express");
const app = express();
const users = [{ id: 1, name: "张三" }];
app.get("/api/users", (req, res) => {
  res.json(users);
});
const port = 4000;
app.listen(port, () => {
  console.log(`Server is running on http://localhost:${port}`);
});
```

#### 6.2.3 users.html

src\5.nginx\users.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>a</title>
  </head>

  <body>
    <script>
      // 定义一个立即执行的异步函数
      (async function () {
        try {
          // 使用 fetch API 发送 GET 请求到指定的 URL
          const response = await fetch("http://localhost:8080/api/users", {
            // 指定请求方法为 GET
            method: "GET",
            // 指定请求头，表明期望的响应格式为 JSON
            headers: {
              Accept: "application/json",
            },
            // 使用 then 方法将响应对象转换为 JSON 格式
          }).then((response) => response.json());
          // 打印获取到的响应
          console.log("response ", response);
        } catch (error) {
          // 如果有任何错误，打印错误信息
          console.error("Error:", error);
        }
        // 立即执行上述定义的函数
      })();
    </script>
  </body>
</html>
```

## 7. node 中间件

### 7.1 http-proxy-middleware

`http-proxy-middleware` 是一个用于 Node.js 的中间件，它可以在你的应用中创建一个反向代理。这在处理跨域请求、添加负载均衡、或者在开发环境中连接到不同的服务等场景中非常有用。

以下是一些基本的使用方法：

1. 安装 http-proxy-middleware:
   你可以使用 npm 或者 yarn 来安装这个包：

```
npm install http-proxy-middleware
# 或者
yarn add http-proxy-middleware
```

2. 创建一个代理：

你可以使用 `createProxyMiddleware` 函数来创建一个代理。这个函数接收一个配置对象，你可以在这个对象中指定代理的目标、路径重写规则等选项：

```js
const { createProxyMiddleware } = require("http-proxy-middleware");

app.use(
  "/api",
  createProxyMiddleware({
    target: "http://localhost:4000",
    changeOrigin: true,
    pathRewrite: {
      "^/api": "",
    },
  })
);
```

在这个例子中，所有以 `/api` 开头的请求都会被代理到 `http://localhost:4000`, 并且路径中的 `/api` 会被去掉。

以下是 `createProxyMiddleware` 配置对象的一些常用选项：

- `target`: 这是代理的目标服务器的 URL。
- `changeOrigin`: 如果设置为 `true`, 代理服务器会在请求转发时修改请求头中的 host 为目标服务器的 host。
- `pathRewrite`: 这是一个对象，它定义了如何重写路径。例如，你可以将路径中的 `/api` 替换 为 `/`。

### 7.2 servera.js

src\6.node\servera.js

```js
const express = require("express");
const { createProxyMiddleware } = require("http-proxy-middleware");
const app = express();
app.use(express.static("public"));
// 使用 express 应用的 use 方法添加一个中间件
app.use(
  // 当请求的路径以 '/api' 开头时，这个中间件将会处理该请求
  "/api",
  // 使用 http-proxy-middleware 创建一个代理中间件
  createProxyMiddleware({
    // 代理的目标服务器地址
    target: "http://localhost:4000",
    // 设置 changeOrigin 为 true ，代理服务器会在请求转发时修改请求头中的 host 为目标服务器的 host
    changeOrigin: true,
    // 使用 pathRewrite 重写请求路径。这里的配置会将路径中的 '^/api' 替换为 ''
    pathRewrite: {
      "^/api": "",
    },
  })
);

app.listen(3000, () => {
  console.log("Proxy server is running on port 3000");
});
```

### 7.3 index.html

src\6.node\public\index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>a</title>
  </head>

  <body>
    <script>
      // 定义一个立即执行的异步函数
      (async function () {
        try {
          // 使用 fetch API 发送 GET 请求到指定的 URL
          const response = await fetch("http://localhost:3000/api/users", {
            // 指定请求方法为 GET
            method: "GET",
            // 指定请求头，表明期望的响应格式为 JSON
            headers: {
              Accept: "application/json",
            },
            // 使用 then 方法将响应对象转换为 JSON 格式
          }).then((response) => response.json());
          // 打印获取到的响应
          console.log("response ", response);
        } catch (error) {
          // 如果有任何错误，打印错误信息
          console.error("Error:", error);
        }
        // 立即执行上述定义的函数
      })();
    </script>
  </body>
</html>
```

### 7.3 serverb.js

src\6.node\serverb.js

```js
const express = require("express");
const app = express();
const users = [{ id: 1, name: "张三" }];
app.get("/users", (req, res) => {
  res.json(users);
});
const port = 4000;
app.listen(port, () => {
  console.log(`Server is running on http://localhost:${port}`);
});
```

## 8. window.name

window.name 是一个旧的 HTML 属性，它允许在同一浏览器窗口或者 Tab 中的不同页面之间传递数据。它有一个非常有趣的特性： 当你改变窗口的 URL 时，window.name 的值会保持不变。这个特性使得 window.name 可以被用于跨域通信。

### 8.1 servera.js

src\7.name\servera.js

```js
// 引入 Express 框架
const express = require("express");
// 创建一个 Express 应用
const app = express();
// 设置静态文件夹
app.use(express.static("public"));
// 定义监听端口
const port = 3000;
// 让应用监听在指定端口并在控制台输出信息
app.listen(port, () => {
  // 当服务器开始运行时，打印一条消息
  console.log(`Server is running on http://localhost:${port}`);
});
```

### 8.2 serverb.js

src\7.name\serverb.js

```js
// 引入 Express 框架
const express = require("express");
// 创建一个 Express 应用
const app = express();
// 设置静态文件夹
app.use(express.static("public"));
// 定义监听端口
const port = 4000;
// 让应用监听在指定端口并在控制台输出信息
app.listen(port, () => {
  // 当服务器开始运行时，打印一条消息
  console.log(`Server is running on http://localhost:${port}`);
});
```

### 8.3 a1.html

src\7.name\public\a1.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>origin</title>
  </head>

  <body>
    <iframe
      id="myIframe"
      src="http://localhost:4000/b.html"
      style="display: none;"
    ></iframe>
    <script>
      let iframe = document.getElementById("myIframe");
      let firstLoaded = true;
      iframe.onload = function () {
        if (firstLoaded) {
          iframe.src = "http://localhost:3000/a2.html";
          firstLoaded = false;
        } else {
          //Uncaught DOMException: Blocked a frame with origin "http://localhost:3000" from accessing a cross-origin frame.at iframe.onload (http://localhost:3000/a1.html:16:46)
          console.log(iframe.contentWindow.name);
        }
      };
    </script>
  </body>
</html>
```

### 8.4 a2.html

src\7.name\public\a2.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>origin</title>
  </head>

  <body></body>
</html>
```

### 8.5 b.html

src\7.name\public\b.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>origin</title>
  </head>

  <body>
    <script>
      window.onload = function () {
        window.name = JSON.stringify({ message: "hello" });
      };
    </script>
  </body>
</html>
```

## 9.location.hash

使用 `location.hash` 和 `iframe` 可以实现跨域通信，它利用的是 `hash` 不会引起页面刷新，也不会被服务器接收的特性。下面是一个具体的例子：

### 9.1 servera.js

src\8.hash\servera.js

```js
// 引入 Express 框架
const express = require("express");
// 创建一个 Express 应用
const app = express();
// 设置静态文件夹
app.use(express.static("public"));
// 定义监听端口
const port = 3000;
// 让应用监听在指定端口并在控制台输出信息
app.listen(port, () => {
  // 当服务器开始运行时，打印一条消息
  console.log(`Server is running on http://localhost:${port}`);
});
```

### 9.2 serverb.js

src\8.hash\serverb.js

```js
// 引入 Express 框架
const express = require("express");
// 创建一个 Express 应用
const app = express();
// 设置静态文件夹
app.use(express.static("public"));
// 定义监听端口
const port = 4000;
// 让应用监听在指定端口并在控制台输出信息
app.listen(port, () => {
  // 当服务器开始运行时，打印一条消息
  console.log(`Server is running on http://localhost:${port}`);
});
```

### 9.3 a1.html

src\8.hash\public\a1.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>origin</title>
  </head>

  <body>
    <iframe
      id="myIframe"
      src="http://localhost:4000/b.html"
      style="display: none;"
    ></iframe>
    <script>
      window.addEventListener("hashchange", () => {
        console.log(location.hash.slice(1));
      });
    </script>
  </body>
</html>
```

### 9.3 a2.html

src\8.hash\public\a2.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>origin</title>
  </head>

  <body>
    <script>
      window.parent.parent.location.hash = location.hash.slice(1);
    </script>
  </body>
</html>
```

### 9.4 b.html

src\8.hash\public\b.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>origin</title>
  </head>

  <body>
    <script>
      let iframe = document.createElement("iframe");
      iframe.src = "http://localhost:3000/a2.html#hello";
      document.body.appendChild(iframe);
    </script>
  </body>
</html>
```

## 10. document.domain

`document.domain` 是一种在同一主域名下的不同子域之间实现跨域通信的方法。这种方法的基本思想是通过将每个子域的 `document.domain` 设置为相同的主域名，使得这些子域在 JavaScript 中被认为是同源的。

```
C:\Windows\System32\drivers\etc\
```

### 10.1 a.html

src\9.domain\public\a.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>origin</title>
  </head>

  <body>
    <iframe
      id="myIframe"
      src="http://sub2.zhufeng.com:4000/b.html"
      style="display: none;"
    ></iframe>
    <script>
      document.domain = "zhufeng.com";
      window.onload = function () {
        var iframe = document.getElementById("myIframe");
        console.log(
          iframe.contentWindow.document.getElementById("container").innerHTML
        );
      };
    </script>
  </body>
</html>
```

### 10.2 b.html

src\9.domain\public\b.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>origin</title>
  </head>

  <body>
    <div id="container">container</div>
    <script>
      document.domain = "zhufeng.com";
    </script>
  </body>
</html>
```
