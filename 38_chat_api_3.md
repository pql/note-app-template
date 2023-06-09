## 1.初始化项目
```sh
egg-init --type simple chat-api
```
## 2.实现后端的登录接口
### 2.1 app\model\user.js
```js
module.exports = app => {
    const mongoose = app.mongoose;
    const Schema = mongoose.Schema;

    const UserSchema = new Schema({
        email: { type: String }
    });

    return mongoose.model('User', UserSchema);
}
```
### 2.2 app\controller\user.js
app/controller/user.js
```js
const Controller = require('./base');

class UserController extends Controller {
    async login() {
        const {ctx, app} = this;
        let body = ctx.request.body;
        let oldUser = await ctx.model.User.findOne({email: body.email});
        if(oldUser) {
            ctx.session.user = oldUser;
            this.success(oldUser);
        } else {
            let newUser = new ctx.model.User(body);
            await newUser.save();
            ctx.session.user = newUser;
            this.success(newUser);
        }
    }
}
```
### 2.3 app\controller\base.js
app/controller/base.js
```js
const Controller = require('egg').Controller;
class BaseController extends Controller {
    success(data) {
        this.ctx.body = {
            code: 0,
            data
        }
    }
    error(error) {
        this.ctx.body = {
            code: 1,
            error
        }
    }
}
module.exports = BaseController;
```
## 3.房间功能
### 3.1 app/router.js
```js
module.exports = app => {
    const { router, controller } = app;
    router.post('/login', controller.user.login);
    router.get('/rooms', controller.rooms.list);
    router.post('/rooms', controller.rooms.create);
};
```
### 3.2 config/config.default.js
config/config.default.js
```js
config.mongoose = {
    client: {
        url: 'mongodb://127.0.0.1/chat',
        options: {}，
    }，
}
config.security = {
    csrf: false,
    domainWhiteList: ['http://127.0.0.1:8000']
}
```
### 3.3 config/plugin.js
config/plugin.js
```js
exports.mongoose = {
    enable: true,
    package: 'egg-mongoose',
};
exports.cors = {
    enable: true,
    package: 'egg-cors',
}
```
### 3.4 app/controller/rooms.js
app/controller/rooms.js
```js
const Controller = require('./base');
class RoomController extends Controller {
    async list() {
        const {ctx, app} = this;
        let {keyword=''} = ctx.query;
        let query={};
        if(keyword) {
            query['name'] = new RegExp(keyword);
        }
        let list = await app.model.Room.find(query);
        this.success(list);
    }
    async create() {
        const {ctx, app} = this;
        let body = ctx.request.body;
        let oldRoom = await app.model.Room.findOne({name: body.name});
        if(oldRoom) {
            this.error('房间名已经存在');
        } else {
            let newRoom = new app.model.Room(body);
            await newRoom.save();
            this.success(newRoom);
        }
    }
}
module.exports = RoomController;
```
### 3.5 app/model/room.js
app/model/room.js
```js
module.exports = app => {
    const mongoose = app.mongoose;
    const Schema = mongoose.Schema;

    const RoomSchema = new Schema({
        name: { type: String }
    });

    return mongoose.model('Room', RoomSchema);
}
```
## 4.聊天功能
### 4.1 app/controller/user.js
```js
const Controller = require('./base');
const gravatar = require('gravatar');
class UserController extends Controller {
    async login() {
        const {ctx, app} = this;
        let user = ctx.request.body;
        let doc = await ctx.model.User.findOne({email: user.email});
        if(!doc) {
            user.name = user.email.split('@')[0];
            user.avatar = gravatar.url(user.email);
            doc = new ctx.model.User(user);
            await doc.save();
        }
        let token = app.jwt.sign(doc.toJSON(), app.config.jwt.secret);
        this.success(token);
    }
}
```
### 4.2 app/model/room.js
app/model/room.js
```js
module.exports = app => {
    const mongoose = app.mongoose;
    const Schema = mongoose.Schema;
    const ObjectId = Schema.Types.ObjectId;
    const RoomSchema = new Schema({
        name: {type: String},
        createAt: {type: Date, default: Date.now},
        creator: {type: ObjectId, ref: 'User'}
    });

    return mongoose.model('Room', RoomSchema);
}
```
### 4.3 app/model/user.js
app/model/user.js
```js
module.exports = app => {
    const mongoose = app.mongoose;
    const Schema = mongoose.Schema;
    const ObjectId = Schema.Types.ObjectId;
    const UserSchema = new Schema({
        name: String,
        avatar: String,
        email: {type: String},
        room: {type: ObjectId, ref: 'Room'},
        socket: {type: String},
        online: {type: Boolean, default: false}
    });

    return mongoose.model('User', UserSchema);
}
```
### 4.4 app/router.js
app/router.js
```js
module.exports = app => {
    const { router, controller, io } = app;
    router.post('/login', controller.user.login);
    router.post('/rooms', controller.rooms.list);
    router.post('/rooms', controller.rooms.create);
    io.route('getRoom', io.controller.messages.getRoom);
    io.route('addMessage', io.controller.message.addMessage);
};
```
### 4.5 config/config.default.js
config/config.default.js
```js
'use strict';
module.exports = appInfo => {
    const config = exports = {}；
    // use for cookie sign key, should change to your own and keep security
    config.keys = appInfo.name + '_1533636529759_8753';

    // add your config here
    config.middleware = [];

    config.mongoose = {
        client: {
            url: 'mongodb://127.0.0.1/chat',
            options: {},
        },
    };

    config.security = {
        csrf: false,
        domainWhiteList: ['http://127.0.0.1:8000']
    }
    config.io = {
        init: {wsEngine: 'ws'}, // default: ws
        namespace: {
            '/': {
                connectionMiddleware: ['connect'],
                packetMiddleware: [],
            }
        },
        redis: {
            host: '127.0.0.1',
            port: 6379
        },
    };
    config.jwt = {
        secret: 'zfpx'
    }
    return config;
};

```
### 4.6 config/plugin.js
config/plugin.js
```js
exports.io = {
    enable: true,
    package: 'egg-socket.io',
};
exports.jwt = {
    enable: true,
    package: 'egg-jwt'
}
```
### 4.7 package.json
package.json
```js
"scripts": {
    "start": "egg-scripts start --daemon --title=egg-server-chat-api --sticky",
    "stop": "egg-scripts stop --title=egg-server-chat-api",
    "dev": "egg-bin dev --sticky",
    "debug": "egg-bin debug",
    "test": "npm run lint -- --fix && npm run test-local",
    "test-local": "egg-bin test",
    "cov": "egg-bin cov",
    "lint": "eslint .",
    "ci": "npm run lint && npm run cov",
    "autod": "autod"
}
```
### 4.8 app/io/controller/messages.js
app/io/controller/messages.js
```js
const {Controller} = require('egg');
class MessageController extends Controller {
    async addMessage() {
        let {app, ctx} = this;
        let message = ctx.args[0]; // {user, room, content}
        let newMessage = new app.model.Message(message);
        await newMessage.save();
        let doc = await app.model.Message.findById(newMessage._id).populate('user');
        app.io.to(message.room).emit('messageAdded', doc.toJSON());
    }
    async getRoom() {
        let {app, ctx} = this;
        let room = ctx.args[0];
        let users = await app.model.User.find({room,online: true});
        let messages = await app.model.Message.find({room}).sort({createAt: -1}).populate('user').sort({createAt: -1}).limit(20);
        ctx.socket.emit('room', {users, messages: messages.reverse()});
    }
}
module.exports = MessageController;
```
### 4.9 app/io/middleware/connect.js
app/io/middleware/connect.js
```js
// {app_root}/app/io/middleware/packet.js
const SYSTEM = {
    name: '系统',
    email: 'admin@126.com',
    avatar: 'http://www.gravatar.com/avatar/1e6fd8e56879c84999cd481255530592'
}
module.exports = app => {
    return async (ctx, next) => {
        const { app, socket, query: {room, token } } = ctx;
        if(token) {
            let decodeUser=app.jwt.verify(token,app.config.jwt.secret);
            if (decodeUser) {
                let oldUser=await app.model.User.findById(decodeUser._id);
                let oldSocket=oldUser.socket;
                 if (oldSocket) {
                    app.io.of('/').adapter.remoteDisconnect(oldSocket, true, err => {
                        app.logger.error(err);
                    });
                } 
                oldUser.room=room;
                oldUser.online=true;
                oldUser.socket=socket.id;
                await oldUser.save();
                socket.join(room);
                socket.broadcast.to(room).emit('online',oldUser.toJSON());
                socket.broadcast.to(room).emit('messageAdded',{
                    user: SYSTEM,
                    content:`用户${oldUser.name}加入聊天室`
                });
                await next();
                socket.leave(room);
                socket.broadcast.to(room).emit('offline',oldUser._id);
                socket.broadcast.to(room).emit('messageAdded',{
                    user: SYSTEM,
                    content:`用户${oldUser.name}离开了聊天室`
                });
                oldUser.room=null;
                oldUser.online=null;
                oldUser.socket=null;
                await oldUser.save();
            }else {
                socket.emit('needLogin','你需要先登录');
            }
        } else {
            socket.emit('needLogin', '你需要先登录');
        }
    };
};
```
### 4.10 app/model/message.js
app/model/message.js
```js
module.exports = app => {
    const mongoose = app.mongoose;
    const Schema = mongoose.Schema;
    const ObjectId = Schema.Types.ObjectId;
    const RoomSchema = new Schema({
        user: {type: ObjectId, ref: 'User'},
        content: String,
        room: {type: ObjectId, ref: 'room'},
        createAt: {type: Date, default: Date.now}
    });
    return mongoose.model('Message', RoomSchema);
}
```
### 4.11 server/app.js
```js
var express = require('express');
var app = express();
var path = require('path');
var server = require('http').createServer(app);
app.use(express.static(path.join(__dirname, 'public')));
var io = require('socket.io')(server);
var port = process.env.PORT || 3000;

server.listen(port, () => {
    console.log('Server listening at port $d', port);
});
```
### 4.12 server/public/index.html
server/public/index.html
```html
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>Document</title>
    </head>
    <body>
        <button onclick="send()">发送</button>
        <script src="/socket.io/socket.io.js"></script>
        <script>
            let socket = io('http://localhost:7001/',{query:{room:'default'}});
            socket.on('connect', function () {
                console.log('连接成功');
                socket.emit('getAllMessages');
            });
            socket.on('messageAdded', function (message) {
                console.log(message);
            });
            socket.on('allMessages', function (messages) {
                console.log(messages);
            });
            socket.on('message', function (message) {
                console.log(message);
            });
            socket.on('needLogin', function (message) {
                console.log(message);
            });

            function send() {
                socket.emit('addMessage', { content: '你好' });
            }
        </script>
    </body>
</html>
```
## 参考
-[chat-api](https://gitee.com/zhufengpeixun/chat-api)