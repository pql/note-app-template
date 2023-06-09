## 1.配置IP地址
### 1.1 ifconfig临时配置IP
- 查看与配置网络状态
- 临时设置eth0网卡的IP地址与子网掩码
```sh
[root@localhost cron.daily]# ifconfig
eth0      Link encap:Ethernet  HWaddr 00:0C:29:E5:3C:11  
          inet addr:172.18.0.252  Bcast:172.18.3.255  Mask:255.255.252.0
```
```sh
ifconfig eth0 172.18.0.254 netmask 255.355.255.0
```
### 1.2 setup永久配置IP
```sh
setup
service network restart
```
## 2.网络配置文件
### 2.1 网卡配置
/etc/sysconfig/network-scripts/ifcfg-eth0
```sh
vi /etc/sysconfig/network-scripts/ifcfg-eth0

DEVICE=eth0 网卡设备号
BOOTPROTO=none 是否自动获取IP地址(none、static、dhcp)
HWADDR=00:0c:29:e5:3c:11 MAC地址
ONBOOT=yes 是否随网络服务启动，eth0生效
TYPE=Ethernet 类型为以太网
NM_CONTROLLED=yes 是否可以由Network Manager图形管理工具托管
UUID="825f66ab-edd7-4076-a256-7a68fb94bf43" 唯一识别码
USERCTL=no 不允许非root用户控制此网
IPV6INIT=no 不启用IPV6
IPADDR=172.18.0.240 IP地址
NETMASK=255.255.252.0 子网掩码
DNS2=8.8.8.8 DNS服务器
GATEWAY=172.18.0.1 网关
DNS1=8.8.8.8 DNS1服务器
```
- 复制的虚拟机能共存于同一个局域网？Mac地址是否会相同？IP地址会相同？
    - 能共存于同一局域网，Mac地址不同，IP地址不同。
    - 对于复制的虚拟机，在开机时，VMware自动为其分配了不同的Mac地址以及IP地址。
- 为什么拷贝的CentOS系统网络配置文件中的UUID与原系统相同？
    - UUID（Universally Unique Identifier）是系统层面的全局唯一标识符号，Mac地址以及IP地址是网络层面的标识号；
    - 两台不同的Linux系统拥有相同的UUID并不影响系统的使用以及系统之间的通信
```sh
可输入如下命令获得新UUID号
# uuidgen ens33
```
### 2.2 主机名
/etc/sysconfig/network
```sh
NETWORKING=yes 网络功能是否起作用
HOSTNAME=localhost.localdomain 主机名

hostname zhufengjiagou
service network restart
```
### 2.3 DNS配置文件
```sh
# cat /etc/resolv.conf
nameserver 8.8.8.8 DNS 服务器
search localhost
nameserver 8.8.8.8
```
## 3.查看网络环境
### 3.1 ifconfig
- 查看与配置网络状态命令
- ifconfig看不到网关和DNS `ipconfig/all`
```
   物理地址. . . . . . . . . . . . . : 14-4F-8A-98-F2-EC
   IPv4 地址 . . . . . . . . . . . . : 192.171.207.104(首选)
   子网掩码  . . . . . . . . . . . . : 255.255.255.0
   默认网关. . . . . . . . . . . . . : 192.171.207.1
   DHCP 服务器 . . . . . . . . . . . : 192.171.207.1
   DNS 服务器  . . . . . . . . . . . : 192.171.207.1
```
### 3.2 关闭和启动网卡
- 禁用该网卡设备 `ifdown 网卡设备名`
- 启动该网卡设备 `ifup 网卡设备名`
### 3.3 查询网络状态
- netstat 选项

| 选项 | 含义 |
| --- | --- |
| -t | 列出TCP协议端口 |
| -u | 列出UDP协议端口 |
| -n | 不使用同域名与服务名，而使用IP地址和端口号 |
| -l | 仅列出在监听状态网络服务 |
| -a | 列出所有的网络连接 |
```sh
netstat -tlun
netstat -an | more
netstat -unt | grep ESTABLISHED
```
### 3.4 netstat -rn
- -r:列出路由列表，功能和 `route` 命令一致
```sh
# netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
192.171.207.0   0.0.0.0         255.255.255.0   U         0 0          0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U         0 0          0 eth0
0.0.0.0         192.171.207.1   0.0.0.0         UG        0 0          0 eth0

# route -n
给路由添加默认网关地址 192.171.207.1
route add default gw 192.171.207.2
route del default gw 192.171.207.2
```
### 3.5 域名解析命令
- nslookup [主机名或IP]
- 进行域名与IP地址解析
- 查看本机的DNS服务器
```sh
# nslookup www.baidu.com
Server:        192.171.207.1
Address:    192.171.207.1#53

Name:    www.baidu.com
Address: 61.135.169.125
```
查看当前的DNS服务器
```sh
[root@192-171-207-101-static ~]# nslookup
> server
Default server: 192.171.207.1
Address: 192.171.207.1#53
```
## 4.网络测试命令
### 4.1 ping
- ping [选项] ip或域名
- 测试指定IP或域名的网络状况
- 选项
    -c 次数指定ping包的次数
```sh
ping www.baidu.com -c 3
```
### 4.2 traceroute [选项] IP或域名
- 路由跟踪命令
- 选项
    - -n使用IP，不使用域名，速度更快
```sh
root@192-171-207-101-static ~]# traceroute www.baidu.com
traceroute to www.baidu.com (61.135.169.125), 30 hops max, 60 byte packets
 1  192-171-207-1-static.bbn.ken-tennwireless.com (192.171.207.1)  0.434 ms  0.323 ms  0.359 ms
 2  localhost (192.168.0.1)  0.948 ms  0.922 ms  1.023 ms
 3  111.196.181.1 (111.196.181.1)  6.849 ms  6.829 ms  9.585 ms
 4  123.126.25.209 (123.126.25.209)  12.284 ms  12.405 ms  12.471 ms
 5  125.33.185.165 (125.33.185.165)  11.276 ms  11.253 ms  11.384 ms
 6  bt-227-030.bta.net.cn (202.106.227.30)  11.580 ms  15.564 ms  15.909 ms
 7  123.125.248.106 (123.125.248.106)  57.920 ms * 123.125.248.110 (123.125.248.110)  13.546 ms
```
### 4.3 wget
- 下载命令
```sh
wget http://www.baidu.com
```
### 4.4 tcpdump命令
- `tcpdump -i eth0 -nnX port 21`
- 选项
    - -i指定网卡接口
    - -nn将数据包中的域名与服务转为IP和端口
    - -X 以十六进制和ASCII码显示数据包内容
    - port 指定监听的端口
## 5.远程登录
### 5.1 SSH协议原理
#### 5.1.1 对称加密算法
- 采用单密钥系统的加密方法，同一个密钥可以同时用作信息的加密和解密，这种加密被称为对称加密。
- 非对称加密算法，需要公钥和私钥
#### 5.1.2 SSH安全外壳协议
- ssh 用户名@ip
- 远程管理指定Linux服务器
```sh
[root@192-171-207-101-static ~]# ssh root@192.171.207.101
The authenticity of host '192.171.207.101 (192.171.207.101)' can't be established.
RSA key fingerprint is a4:97:52:eb:0a:0b:35:a0:98:7d:4f:c8:3b:dc:f9:0a.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.171.207.101' (RSA) to the list of known hosts.
```
/root/.ssh/known_hosts
```sh
192.171.207.101 ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAomDpQxV3RmjJyKkf7elMTInbdm+/ZLnFpfbAryi5PSb2ewfYbwRaBcVl1lBta6yjFuz0J12p9qy90DBhadvoBsfwTB8lQhmlT8B2eCcHr0bfLa1IdKMcjImxRJiD4v0emCGFquHnHIr41vs8uxQ2Ek28mH/1JC0e/+VPEvylBB4+Kk2789ACdAlmhGTtlu7zgeUoLaWQSl1/6g7zfSLIz+/U8qGiRSPaGT+M40oqx/PZdoGOMTRhHgNIR5qgvcNaJXhlZGYT42fLFSmtzUHJ030hP7JGZ99oXS20/mnc8qvonC9itp0+K/nCj5g6uR/gPFb5B0NmTZCM2/gcLkHumw==
```
#### 5.1.3 scp
- scp是 secure copy的缩写，scp是linux系统下基于ssh登陆进行安全的远程文件拷贝命令
- linux的scp命令可以在linux服务器之间复制文件和目录
- 命令格式scp[参数][原路径][目标路径]
| 参数 | 含义 |
| --- | --- |
| -r | 递归复制整个目录 |
| -v | 详细方式显示输出 |
##### 5.1.3.1 从本地服务器复制到远程服务器
```sh
scp local_file remote_username@remote_ip:remote_folder  
scp -r local_folder remote_username@remote_ip:remote_folder  
```
##### 5.1.3.2 从远程服务器复制到本地服务器
```sh
scp remote_username@remote_folder  local_file
scp -r remote_username@remote_ip:remote_folder  local_folder
```
## 6.附录
### 6.1 搭建FTP服务器
#### 6.1.1 查询是否安装了vsftpd服务
```sh
rpm -q vsftpd
```
#### 6.1.2 安装vsftpd
```sh
yum install -y vsftpd
```
#### 6.1.3 修改vsftpd配置文件
- `vi /etc/vsftpd/vsftpd.conf` 修改vsftpd 配置文件
```sh
anoonymous_enable=NO 是否允许匿名用户登录
local_enable=YES 允许本地用户登录
Write_enable=YES 是否可以写入
chroot_local_user=YES #是否将所有用户限制在主目录，YES为启用，NO禁用
chroot_list_enable=YES #是否启动限制用户的名单
chroot_list_file=/etc/vsftpd/chroot_list #是否限制在主目录下的用户名单
```
#### 6.1.4 设置用户可以访问home文件夹
```sh
getsebool -a|grep ftp #查看selinux配置
setsebool -P ftp_home_dir 1 #更改设置(-P 是开机自动使用，无需每次开机都输入该命令)
service vsftpd restart #重启vsftpd
```
```sh
vi /etc/selinux/config
SELINUX=disabled
```
#### 6.1.5 启动服务
```sh
chmod -R 777 /home/zhangsan2
chkconfig vsftpd on
service iptables stop
service vsftpd restart
```
#### 6.1.6 创建用户
```sh
adduser lisi
passwd zhaoliu 设置密码 zhaoliu
```
#### 6.1.7 CMD中的FTP命令
| 命令 | 含义 |
| --- | --- |
| ftp 192.168.1.3 | 登录ftp |
| dir | 显示远程主机目录 |
| help[cmd] | 显示ftp内部命令cmd的帮助信息 |
| get remote-file[local-file] | 将远程主机的文件remote-file传至本地硬盘的local-file(本地文件夹) |
| put local-file[remote-file] | 将本地文件local-file传送至远程主机 |
| quit | 同bye,退出ftp会话 |