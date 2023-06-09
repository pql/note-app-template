## 1.服务器部署步骤
1. 购买自己的域名
2. 域名备案
3. 购买服务器
4. 配置服务器应用环境
5. 安装配置服务器
6. 项目远程部署和发布于更新
### 1.2 购买域名
- [腾讯云](https://dnspod.cloud.tencent.com/)
- [阿里云](https://wanwang.aliyun.com/)
- [百度云](https://cloud.baidu.com/product/bcd.html)
- [爱名网](https://www.22.cn/)
- [godaddy](https://sg.godaddy.com/)
### 1.3 云主机
- [阿里云ECS](https://www.aliyun.com/)
- [亚马逊AWS](https://aws.amazon.com/cn/)
- [百度云](https://cloud.baidu.com/)
### 1.4 购买阿里云
- [选择配置](https://ecs-buy.aliyun.com/wizard/#/postpay/cn-beijing)
- 镜像 Ubuntu 16.04 64位
### 1.5 备案
- [阿里云备案](https://beian.aliyun.com/)
- [备案服务号管理](https://bsn.console.aliyun.com/#/bsnManagement)
## 2.安装软件
### 2.1 连接服务器
```sh
ssh root@47.92.243.226
```
### 2.2 升级系统
```sh
apt-get update
apt-get upgrade
```
### 2.3 安装工具包
```sh
apt-get install wget curl git
```
### 2.4 安装 node
- 使用[nvm](https://github.com/creationix/nvm/blob/master/README.md)安装 node
```sh
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | sh
source /root/.bashrc
nvm install stable
node -v
npm i cnpm -g
npm i nrm -g
```
### 2.5 nginx
Nginx 是一个高性能的HTTP和反向代理服务器
#### 2.5.1 安装
```sh
apt-get install nginx
```
### 2.5.2 nginx命令
| 名称 | 命令 |
| --- | --- |
| 启动 nginx | nginx -c /etc/nginx/nginx.conf |
| 关闭 nginx | nginx -s stop |
| 重读配置文件 | nginx -s reload kill -HUP nginx |
| 常用命令 | service nginx {start、stop、status、restart、reload、configtest} |
### 2.6 mysql
安装 mysql
```sh
apt install mysql-server -y
apt install mysql-client
apt install libmysqlclient-dev
```
- [root用户重置密码问题](https://www.cnblogs.com/roadofstudy/p/7446690.html)
## 3.部署项目
### 3.1 部署后端项目
```sh
git clone https://gitee.com/zhufengpeixun/2018projects1.git
cd 2018projects1/zhufeng-cms
yarn

wget http://7xil5b.com1.z0.glb.clouddn.com/zhufeng-cms.sql
mysql -uroot -proot
CREATE DATABASE IF NOT EXISTS zhufengcms default charset utf8 COLLATE utf8_general_ci;
use zhufengcms;
source zhufengcms.sql;
yarn start
```
### 3.1 部署前端项目
```sh
cd 2018projects1/zhufeng-cms-front
yarn
yarn build
```
/etc/nginx/nginx.conf
```
include /etc/nginx/conf.d /*.conf;
include /etc/nginx/sites-enabled/*;
```
include /etc/nginx/sites-enabled/default
```
root /var/www/html;
```
```sh
scp 1.txt root@47.92.243.226:/root

server {
    listen 80;
    server_name 27.92.243.226;

    root /var/www/html;

    location / {
        try_files $uri/ @router;
        index index.html;
    }

    location /api {
        rewrite ^/api/(.*)$ /$1 break;
        proxy_pass http://127.0.0.1:7001;
    }

    location @router {
        rewrite ^.*$ /index.html break;
    }
}
```
## 参考
- [yarn](https://yarn.bootcss.com/docs/usage/)
- [deployment](https://eggjs.org/zh-cn/core/deployment.html)
- [deploy](https://umijs.org/zh/guide/deploy.html)