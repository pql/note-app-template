## 1.软件包管理
- RPM是RedHat Package Manager（RedHat软件包管理工具）类似Windows里面的"添加/删除程序"
### 1.1 软件包的分类
- 源码包(需要经过编译，把人所编写的源代码编译成机器语言才能运行)
    - 优点
        - 开源免费
        - 可以自由配置功能
        - 编译安装更适合自己系统，更稳定
        - 卸载方便
    - 缺点
        - 安装过程比较复杂
        - 编译过程比较长
        - 安装过程一旦报错，非常难以排查
- 二进制包(把源代码包经过编译生成0/1二进制，PRM包、系统默认的安装包)
    - 优点
        - 包管理系统比较简单，只要通过简单的命令就可以实现包的安装、升级、查询和卸载
        - 安装速度比源码包快很多
    - 缺点
        - 经过编译则不能看到源代码
        - 功能选择不灵活
        - 依赖性比较麻烦
- 脚本安装包(就是把复杂的安装过程写成了脚本，可以一键安装，本质上安装的还是源代码包和二进制包)
    - 优点是安装简单
    - 缺点是失去了自定义性
## 2. YUM在线管理
- yum = Yellow dog Updater, Modified主要功能是更方便的添加/删除/更新RPM包.它能自动解决包的倚赖性问题.
- 这是rpm包的在线管理命令
- 将所有的软件名放到官方服务器上，当进行YUM在线安装时，可以自动解决依赖性问题
- /etc/yum.repos.d/
    - CentOS-Base.repo
    - epel.repo
### 2.1 CentOS-Base.repo
```sh
[base]
name=CentOS-$releasever
enabled=1
failovermethod=priority
baseurl=http://mirrors.cloud.aliyuncs.com/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=http://mirrors.cloud.aliyuncs.com/centos/RPM-GPG-KEY-CentOS-7
```
| 字段 | 含义 |
| --- |  --- |
| base | 容器名称，一定要放在[]中 |
| name | 容器说明，可以自己随便写 |
| mirrorlist | 镜像站点，可以注释掉 |
| baseurl | YUM源服务器的地址，默认是CentOS官方的YUM源 |
| enable | 此容器是否生效 不写或者写成enable=1表示生效，写成enable=0表示不生效 |
| gpgcheck | 如果是1就是指 RPM的数字证书生效,如果是0则表示不生效 |
| gpgkey | 数字证书的公钥文件保存位置，不用改 |

使用阿里云镜像
```sh
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
yum makecache
yum -y update //升级所有包同时也升级软件和系统内核
```
## 3. YUM命令
- yum安装只需要写包名即可

| 命令 | 含义 |
| --- | --- |
| yum list | 查询所有可用软件包列表 |
| yum search 关键字 | 搜索服务器上所有和关键字相关的包 |
| yum -y install 包名 | -y 自动回答yes install安装 |
| yum -y update 包名 | 	-y 自动回答yes update升级 |
| yum -y remove 包名 | -y 自动回答yes remove 卸载,卸载有依赖性，所以尽量不要卸载 |
| yum grouplist | 列出所有可用的软件组列表 |
| yum groupinstall 软件组名 | 安装指定的组，组名可以用grouplist查询 |
| yum groupremove 软件组名 | 卸载指定软件组 |

```sh
yum -y install gcc  //安装C语言安装包
```
## 4. 常用软件安装
### 4.1 nginx
```sh
yum install nginx  -y
```
```sh
whereis nginx //查看安装位置
```
启动服务
```sh
/bin/systemctl start nginx.service
/bin/systemctl stop nginx.service
```
```sh
curl http://115.29.148.6/
```
### 4.2 mongodb
#### 4.2.1 添加安装源
- vim /etc/yum.repos.d/mongodb-org-3.4.repo
添加以下内容：
```sh
[mongodb-org-3.4]  
name=MongoDB Repository  
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.4/x86_64/  
gpgcheck=1  
enabled=1  
gpgkey=https://www.mongodb.org/static/pgp/server-3.4.asc
```
- 这里可以修改 gpgcheck=0, 省去gpg验证
- yum makecache 就是把服务器的包信息下载到本地电脑缓存起来
#### 4.2.2 更新缓存
```sh
yum makecache
```
#### 4.2.3 安装
```sh
yum -y install mongodb-org
```
#### 4.2.4 修改配置文件
```sh
whereis mongod
vi /etc/mongod.conf
```
/etc/mongod.conf
```sh
net:
  port: 27017
#  bindIp: 127.0.0.1 
```
#### 4.2.5 启动服务
```sh
systemctl start mongod.service
systemctl stop mongod.service
systemctl status mongod.service
systemctl restart mongod.service
```
#### 4.2.6 远程连接
```sh
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
mongo 115.29.148.6
```
### 4.3 redis
#### 4.3.1 安装软件
```sh
yum install redis -y
```
#### 4.3.2 启动服务
```sh
systemctl start redis.service
systemctl stop redis.service
systemctl status redis.service
systemctl restart redis.service
```
### 4.4 mysql
#### 4.4.1 查看最新的安装包
- [https://dev.mysql.com/downloads/repo/yum/](https://dev.mysql.com/downloads/repo/yum/)
#### 4.4.2 下载MySQL源安装包
- wget [http://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm](http://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm)
#### 4.4.3 安装源
- yum -y install mysql57-community-release-el7-11.noarch.rpm
- yum repolist enabled | grep mysql.*
#### 4.4.4 安装MYSQL服务器 
- yum install mysql-community-server -y
```sh
/var/cache/yum/x86_64/7/mysql57-community/packages
https://mirrors.ustc.edu.cn/mysql-ftp/Downloads/MySQL-5.7/
wget https://img.zhufengpeixun.com/mysql5.7-centos7.zip
```
#### 4.4.5 启动服务器 
```sh
systemctl start mysqld.service
systemctl stop mysqld.service
systemctl status mysqld.service
systemctl restart mysqld.service
```
#### 4.4.6 初始化数据库密码
- grep "password" /var/log/mysqld.log
- mysql -uroot -p
- ALTER USER 'root'@'localhost' IDENTIFIED BY 'abcd1#EFG';
- SHOW VARIABLES LIKE 'validate_password%';
#### 4.4.7 支持远程访问
- GRANT ALL PRIVILEGES ON . TO 'root'@'%' IDENTIFIED BY 'abcd1#EFG' WITH GRANT OPTION;
- FLUSH PRIVILEGES;
#### 4.4.8 开机自动访问
- systemctl enable mysqld
- systemctl daemon-reload
#### 4.4.9 远程访问
- C:\program1\mysql-5.7.31-winx64\bin\mysqld MySQL
```sh
mysql -h115.29.148.6 -uroot -p
```