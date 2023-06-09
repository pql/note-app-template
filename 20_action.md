## 1.静态资源服务器命令行工具
1. [创建一个服务器](https://gitee.com/zhufengnodejs/zf-server/commit/f364e32c0d3a1b65a671af946fd13a5363032f65)
2. [实现压缩](https://gitee.com/zhufengnodejs/zf-server/commit/246b6868a77f721816b5a35fdb7fd2f53d5e303f)
3. [实现缓存](https://gitee.com/zhufengnodejs/zf-server/commit/81c2515ebfd475d88d521c68769c498369afb6f6)
4. [实现断点续传](https://gitee.com/zhufengnodejs/zf-server/commit/89656a75d7e79aa42b5f0528aee45d5286fc502f)
5. [发布命令行工具](https://gitee.com/zhufengnodejs/zf-server/commit/e3897cb4b92b1e9b8fc22cec47edc75eaee25e5a)

[静态资源服务器](https://gitee.com/zhufengnodejs/zf-server)

### 1.1 -r/--range
- 该选项指定下载字节的范围，常应用于分块下载文件
- range的表示方式有多种，如100-500，则指定从100开始的400个字节数据；-500表示最后的500个字节；5000-表示从第5000个字节开始的所有字节
- 另外还可以同时指定多个字节块，中间用“，”分开
- 服务器告诉客户端可以使用range response.setHeader('Accept-Ranges', 'bytes')
- Server 通过请求头中的 Range: bytes=0-xxx来判断是否是做Range请求，如果这个值存在而且有效，则只返回请求的那部分文件内容，响应的状态码变成206，如果无效，则返回416状态码，表明Request Range Not Satisfiable
```js
curl -r 0-1024 -o music.mp3
```
## 2.多语言切换
可以通过Accept-Language检查浏览器的语言
- 请求头格式Accept-Language: Accept-Language:zh-CN,zh;q=0.9
- 响应头格式 Content-Language: zh-CN
```js
const http = require('http');
const pack = {
    en: {
        title: 'hello'
    },
    cn: {
        title: '欢迎'
    }
}
function request(req, res) {
    const acceptLanguage = req.headers['accept-language'];
    let lan = 'en';
    if(acceptLanguage) {
        lan = acceptLanguage.split(',').map(item => {
            let values = item.split(';');
            return {
                name: values[0].split('-')[0],
                q: isNaN(values[1])?1:parseInt(values[1])
            }
        }).sort((lan1, lan2) => lan1.q - lan2.q).shift().name;
    }
    res.end(pack[lan]?pack[lan].title:pack['en'].title);
}
const server = http.createServer();
server.on('request', request);
server.listen(8080);
```
## 3.图片防盗链
- 从一个网站跳转，或者网页引用到某个资源文件时，HTTP请求中带有Referer表示来源网页的URL
- 通过检查请求头中的Referer来判断来源网页的域名
- 如果来源域名不在白名单内，则返回错误提示
- 用浏览器直接访问图片地址是没有referer的
```js
const http = require('http');
const url = require('url');
const path = require('path');
const fs = require('fs');
const root = path.join(__dirname, 'public');

function request(req, res){
    const refer = req.headers['referer'] || req.headers['referrer'];
    if (refer) {
        const referHost = getHostName(refer);
        const host = removePort(req.headers['host']);
        if (referHost != host) {
            sendForbidden(req, res);
        } else {
            serve(req, res);
        }
    } else {
        serve(req, res);
    }
}

function getHostName(urlAddr) {
    const { host } = url.parse(urlAddr);
    return removePort(host);
}

function removePort(host) {
    return host.split(':')[0];
}

function sendForbidden(req, res) {
    res.end('防盗链');
}

function server(req, res) {
    const { pathname } = url.parse(req.url);
    const filePath = path.join(root, pathname);
    console.log(req.url, filePath);

    fs.stat(filePath, (err, stat) => {
        if(err) {
            res.end('Not Found');
        } else {
            fs.createReadStream(filePath).pipe(res);
        }
    });
}

const server = http.createServer();
server.on('request', request);
server.listen(8080);
//-H "referer: http://xxx.xxx.xxx.xxx"   http://localhost:8080/mv.jpg
```

## 4.代理服务器
代理（英语：Proxy）,也称网络代理，是一种特殊的网络服务，允许一个网络终端（一般为客户端）通过这个服务与另一个网络终端（一般为服务器）进行非直接的连接。一些网关、路由器等网络设备具备网络代理功能。一般认为代理服务有利于保障网络终端的隐私或安全，防止攻击。
```js
npm install http-proxy --save
```
- web 代理普通的HTTP请求
- listen port
- close 关闭内置的服务
[https://www.npmjs.com/package/http-proxy](https://www.npmjs.com/package/http-proxy)
```js
const httpProxy = require('http-proxy');
const http = require('http');
const proxy = httpProxy.createProxyServer();

http.createServer(function(req, res) {
    proxy.web(req, res, {
        target: 'http://localhost:8080'
    });
    proxy.on('error', function(err) {
        console.log('出错了');
        res.end(err.toString());
    })
}).listen(8080);
```
## 5.虚拟主机
通过Host实现多个网站共用一个端口，多个网站公用一个服务器
```js
const http = requrie('http');
const httpProxy = require('http-proxy');
const proxy = http.createProxyServer();

const hosts = {
    'a.zfpx.cn': 'http://localhost:8000',
    'b.zfpx.cn': 'http://localhost:9000'
}
const server = http.createServer(function(req, res){
    const host = req.headers['host'];
    host = host.split(':')[0];
    const target = hosts[host];
    proxy.web(req, res, {
        target
    });
}).listen(80);
```
## 6.User-Agent
UserAgent中文名为用户代理，简称UA，它是一个特殊字符串头，使得服务器能够识别客户使用的操作系统及版本、CPU类型、浏览器及版本、浏览器渲染引擎、浏览器语言、浏览器插件等。
- 请求头 User-Agent:Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.94 Safari/537.36
- `user-agent-parser`