## 1.珠峰聊天室接口文档
-   用户
    - [post/login登录](http://www.zhufengpeixun.com/grow/html/38.chat-api-1.html#/login)
    - [post/validate验证用户身份](http://www.zhufengpeixun.com/grow/html/38.chat-api-1.html#/validate)
-   房间
    - [post/addRoom添加房间](http://www.zhufengpeixun.com/grow/html/%E6%B7%BB%E5%8A%A0%E6%88%BF%E9%97%B4)
    - [get/getAllRooms查看房间列表](http://www.zhufengpeixun.com/grow/html/%E6%9F%A5%E7%9C%8B%E6%88%BF%E9%97%B4%E5%88%97%E8%A1%A8)
-   消息
    - [addMessage添加消息](http://www.zhufengpeixun.com/grow/html/%E6%B7%BB%E5%8A%A0%E6%B6%88%E6%81%AF)
    - [getAllMessages获得所有消息](http://www.zhufengpeixun.com/grow/html/%E8%8E%B7%E5%BE%97%E6%89%80%E6%9C%89%E6%B6%88%E6%81%AF)
## 2.用户
### 2.1 登录
#### 2.1.1 请求路径
/login
#### 2.1.2 请求方法
POST
#### 2.1.3 请求参数
放在请求体里
```json
{
    "email": "zfpx@110.com"
}
```
### 2.2 验证用户身份
#### 2.2.1 请求路径
/validate
#### 2.2.2 请求方法
POST
#### 2.2.3 请求参数
放在请求体里
```json
{
    "token": "xxxxxx",
}
```
## 3.房间
### 3.1 房间
#### 3.1.1 请求路径
/addRoom
#### 3.1.2 请求方法
POST
#### 3.1.3 请求参数
放在请求体里
```json
{
    "name": "森钱吧"
}
```
### 3.2 获取所有的房间
#### 3.2.1 请求路径
/getAllRooms
#### 3.2.2 请求方法
GET
#### 3.2.3 请求参数
无
## 4.消息
### 4.1 添加消息
#### 4.1.1 消息类型
addMessage
#### 4.1.2 请求方法
websocket
#### 4.1.3 请求参数
```json
{
    "content": "你好",
    "user": "",
    "room": ""
}
```
### 4.2 查看所有的消息列表
#### 4.2.1 消息类型
getAllMessages
#### 4.2.2 请求方法
websocket
#### 4.2.3 请求参数
无