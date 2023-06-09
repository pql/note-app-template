## 1.HTTP 服务器

HTTP 全程是超文本传输协议，构建于 TCP 之上，属于应用层协议。

### 1.1 创建 HTTP 服务器

```js
const http = require("http");
const server = http.createServer([requestListener]);
server.on("request", requestListener);
```

- requestListener 当服务器收到客户端的连接后执行的处理
  - http.IncomingMessage 请求对象
  - http.ServerResponse 对象 服务器端响应对象

### 1.2 启动 HTTP 服务器

```js
server.listen(port, [host], [backlog], [callback]);
server.on("listening", callback);
```

- port 监听的端口号
- host 监听的地址
- backlog 指定位于等待队列中的客户端连接数

```js
const http = require("http");
const server = http
  .createServer(function (req, res) {})
  .listen(8080, "127.0.0.1", function () {
    console.log("服务器端开始监听！");
  });
```

### 1.3 关闭 HTTP 服务器

```js
server.close();
server.on("close", function () {});
```

```js
const http = require("http");
const server = http.createServer(function (req, res) {});
server.on("close", function () {
  console.log("服务器关闭");
});
server.listen(8080, "127.0.0.1", function () {
  console.log("服务器端开始监听！");
  server.close();
});
```

### 1.4 监听服务器错误

```js
server.on("error", function () {
  if (e.code == "EADDRINUSE") {
    console.log("端口号已经被占用!");
  }
});
```

### 1.5 connection

```js
const server = http.createServer(function (req, res) {});
server.on("connection", function () {
  console.log("客户端连接已经建立");
});
```

### 1.6 setTimeout

设置超时时间，超时后不可再复用已经建立的连接，需要发请求需要重新建立连接。默认超时时间 2 分钟

```js
server.setTimeout(msecs, callback);
server.on("timeout", function () {
  console.log("连接已经超时");
});
```

### 1.7 获取客户端请求信息

- request
  - method 请求的方法
  - url 请求的路径
  - headers 请求头对象
  - httpVersion 客户端的 http 版本
  - socket 监听客户端请求的 socket 对象

```js
const http = require("http");
const fs = require("fs");
const server = http
  .createServer(function (req, res) {
    if (req.url != "/favicon.ico") {
      let out = fs.createWriteStream(path.join(__dirname, "request.log"));
      out.write("method=" + req.method);
      out.write("url=" + req.url);
      out.write("headers=" + JSON.stringify(req.headers));
      out.write("httpVersion=" + req.httpVersion);
    }
  })
  .listen(8080, "127.0.0.1");
```

```js
const http = require("http");
const fs = require("fs");
const server = http
  .createServer(function (req, res) {
    let body = [];
    req.on("data", function (data) {
      body.push(data);
    });
    req.on("end", function () {
      let result = Buffer.concat(body);
      console.log(result.toString());
    });
  })
  .listen(8080, "127.0.0.1");
```

### 1.8 querystring

querystring 模块用来转换 URL 字符串和 URL 中的查询字符串

#### 1.8.1 parse 方法用来把字符串转换成对象

```js
querystring.parse(str, [sep], [eq], [options]);
```

#### 1.8.2 stringify 方法用来把对象转化成字符串

```js
querystring.stringify(obj, [sep], [eq]);
```

### 1.9 querystring

```js
url.parse(urlStr, [parseQueryString]);
```

- href 被转换的原 URL 字符串
- protocal 客户端发出请求时使用的协议
- slashes 在协议与路径之间是否使用了 // 分隔符
- host URL 字符串中的完整地址和端口号
- auth URL 字符串中的认证部分
- hostname URL 字符串中的完整地址
- port URL 字符串中的端口号
- pathname URL 字符串的路径，不包含查询字符串
- search 查询字符串，包含？
- path 路径，包含查询字符串
- query 查询字符串，不包含起始字符串 ？
- hash 散列字符串，包含 #

### 1.10 发送服务器响应流

http.ServerResponse 对象表示响应对象

#### 1.10.1 writeHead

```js
response.writeHead(statusCode, [reasonPhrase], [headers]);
```

- content-type 内容类型
- location 将客户端重定向到另外一个 URL 地址
- content-disposition 指定一个被下载的文件名
- content-length 服务器响应内容的字节数
- set-cookie 在客户端创建 Cookie
- content-encoding 指定服务器响应内容的编码方式
- cache-cache 开始缓存机制
- expires 用于指定缓存过期时间
- etag 指定当服务器响应内容没有变化不重新下载数据

#### 1.10.2 Header

设置、获取和删除 Header

```js
response.setHeader('Content-Type', 'text/html;charset=utf8');
response.getHeader('Content-Type');
response.removeHeader('Content-Type');
response.headersSent 判断响应头是否已经发送
```

#### 1.10.3 headersSent

判断响应头是否已经发送

```js
const http = require("http");
const server = httpt.createServer(function (req, res) {
  console.log(response.headersSent ? "响应头已经发送" : "响应头未发送");
  res.writeHead(200, "ok");
  console.log(response.headersSent ? "响应头已经发送" : "响应头未发送！");
});
```

#### 1.10.4 sendDate

不发送 Date

```js
res.sendDate = false;
```

#### 1.10.5 write

可以使用 write 方法发送响应内容

```js
response.write(chunk, [encoding]);
response.end([chunk], [encoding]);
```

#### 1.10.6 timeout

可以使用 setTimeout 方法设置响应让超时时间，如果在指定时间内不响应，则触发 timeout 事件

```js
response.setTimeout(msecs, [callback]);
response.on("timeout", callback);
```

#### 1.10.7 close

在响应对象的 end 方法被调用之前，如果连接中断，将触发 http.ServerResponse 对象的 close 事件

```js
response.on("close", callback);
```

#### 1.10.8 parser

```
net
onconnection

_http_server.js
连接监听
connectionListenerInternal
socketOnData
onParserExecuteCommon
parserOnIncoming
```

## 2.HTTP 客户端

### 2.1 向其他网站请求数据

```js
const req = http.request(options, callback);
req.on("request", callback);
request.write(chunk, [encoding]);
request.end([chunk], [encoding]);
```

- host 指定目标名或主机名
- hostname 指定目标域名或主机名，如果和 host 都指定了，优先使用 hostname
- port 指定目标服务器的端口号
- localAddress 本地接口
- socketPath 指定 Unix 域端口
- method 指定 HTTP 请求的方式
- path 指定请求路径和查询字符串
- headers 指定客户端请求头对象
- auth 指定认证部分
- agent 用于指定 HTTP 代理，在 Node.js 中，使用 http.Agent 类代表一个 HTTP 代理，默认使用 keep-alive 连接，同时使用 http.Agent 对象来实现所有的 HTTP 客户端请求

```js
const http = require("http");
const options = {
  hostname: "localhost",
  port: 8080,
  path: "/",
  method: "GET",
};
const req = http.request(options, function (res) {
  console.log("状态码：" + res.statusCode);
  console.log("响应头:" + JSON.stringify(res.headers));
  res.setEncoding("utf8");
  res.on("data", function (chunk) {
    console.log("响应内容", chunk);
  });
});
req.end();
```

### 2.2 取消请求

可以使用 abort 方法来终止本次请求

```js
req.abort();
```

### 2.3 监听 error 事件

如果请求过程中出错了，会触发 error 事件

```js
request.on("error", function (err) {});
```

### 2.4 socket

建立连接过程中，为该连接分配端口时，触发 socket 事件

```js
req.on("socket", function (socket) {
  socket.setTimeout(1000);
  socket.on("timeout", function () {
    req.abort();
  });
});
```

### 2.5

可以使用 get 方法向服务器发送数据

```js
http.get(options, callback);
```

### 2.6 addTrailers

可以使用 response 对象的 addTrailers 方法在服务器响应尾部追加一个头信息

```js
const http = require("http");
const path = require("path");
const crypto = require("crypto");

const server = http
  .createServer(function (req, res) {
    res.writeHead(200, {
      "Transfer-Encoding": "chunked",
      Trailer: "Content-MD5",
    });
    const rs = require("fs").createReadStream(
      path.join(__dirname, "msg.txt", {
        highWaterMark: 2,
      })
    );
    const md5 = crypto.createHash("md5");
    rs.on("data", function (data) {
      console.log(data);
      res.write(data);
      md5.update(data);
    });
    rs.on("end", function () {
      res.addTrailers({
        "Content-MD5": md5.digest("hex"),
      });
      res.end();
    });
  })
  .listen(8080);
```

```js
const http = require("http");
const options = {
  hostname: "localhost",
  port: 8080,
  path: "/",
  method: "GET",
};
const req = http.request(options, function (res) {
  console.log("状态码：" + res.statusCode);
  console.log("响应头：" + JSON.stringify(res.headers));
  res.setEncoding("utf8");
  res.on("data", function (chunk) {
    console.log("响应内容", chunk);
  });
  res.on("end", function () {
    console.log("trailer", res.trailers);
  });
});
req.end();
```

### 2.7 制作代理服务器

```js
const http = require("http");
const url = require("url");
const server = http
  .createServer(function (request, response) {
    const { path } = url.parse(request.url);
    const options = {
      host: "localhost",
      port: 9090,
      path: path,
      headers: request.headers,
    };
    const req = http.get(options, function (res) {
      console.log(res);
      response.writeHead(res.statusCode, res.headers);
      res.pipe(response);
    });
    req.on("error", function (err) {
      console.log(err);
    });
    request.pipe(req);
  })
  .listen(8080);
```
