## 1. Node.js 安装配置

本章节我们将向大家介绍在 window、Mac 和 Linux 上安装 Node.js 的方法。

- 偶数位为稳定版本，奇数位为非稳定版本
- 稳定版本中已经发布的 API 是不会改变的

### 1.1 打开官网首页

首页会推荐你合适的版本[https://nodejs.org/en/](https://nodejs.org/en/)
![](/public/images/WX20220509-111828.png)

### 1.2 如果推荐的版本不合适可以进入下载页面

[https://nodejs.org/en/download/](https://nodejs.org/en/download/)
![](/public/images/WX20220509-111828.png)
根据不同平台系统选择你需要的 Node.js 安装包。注意：linux 上安装 Node.js 需要安装 Python 2.6 或 2.7，不建议安装 3.0 以上版本。

## 2.windows

### 2.1 步骤 1 : 双击下载后的安装包 node-v4.2.1-x64.msi 运行安装程序：

### 2.2 步骤 2 : 勾选接受协议选项，点击 next（下一步） 按钮 :

### 2.3 步骤 3 : Node.js 默认安装目录为 "C:\Program Files\nodejs\" , 你可以修改目录，并点击 next（下一步）

### 2.4 步骤 4 : 点击树形图标来选择你需要的安装模式 , 然后点击下一步 next（下一步）

### 2.5 步骤 6 :点击 Install（安装） 开始安装 Node.js。你也可以点击 Back（返回）来修改先前的配置。 然后并点击 next（下一步）：

### 2.6 点击 Finish（完成）按钮退出安装向导。

### 2.7 检测 PATH 环境变量是否配置了 Node.js

- 点击开始菜单,点击运行
- 输入 `cmd`
- 输入命令`path`输出结果

如果有`node`的路径的话就表示配置正确，可以在命令行下执行`node`命令 检查 node.js 版本 `node -v`

### 2.8 如果没有的话就需要手工再次配置环境变量

1. 打开资源管理器
2. 在计算机上点击右键，显示菜单后点击属性
3. 选择高级系统设置
4. 选择高级页签下的环境变量
5. 在用户变量中找到 path,如果没有就新建一个
6. 在 path 的最前面添加 node.js 的安装路径，如 `C:\Program Files\nodejs`

## 3. MAC 安装

### 3.1 安装包安装

下载 Mac 安装后结束后，单击下载的文件，运行它，会出现一个向导对话框。 单击 continue 按钮开始安装，紧接着向导会向你询问系统用户密码，输入密码后就开始安装。不一会儿就会看见一个提示 Node 已经被安装到计算机上的确认窗口

### 3.2 homebrew 安装

1. 先安装 homebrew 打开网站 [http://brew.sh/](http://brew.sh/)
2. 在 terminal 下安装 `Homebrew`

```sh
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

homebrew 依赖 ruby,如果安装出错检查一下 ruby 的版本以及路径

```sh
ruby -v
```

3. 通过 homebrew 安装 node.js

```sh
brew install node
```

### 3.3 n 模块安装

切换版本或升级 node 可以安装 `n` 模块

```sh
npm install n -g
```

直接输入`n`即可上下切换不同的版本

- [n 源码](https://github.com/tj/n)
- [n 的 npm 安装包](https://www.npmjs.com/package/n)

## 4.源代码安装

### 4.1 安装依赖库

Node 依赖一些第三方代码库，但幸运的是大多数第三方库已经随 Node 一起发行，如果想从源码编译，需要以下两项工具

- python(2.4 及以上版本)
- libssl-dev 如果计划使用 SSL/TLS 加密，则必须安装它。libssl 是 openssl 工具中用到的库，在 linux 和 UNIX 系统中，通常可以用你喜欢的包管理器安装 libssl,而在 Mac OS X 系统中已经预置了。

### 4.2 下载源代码

选择好版本后，你就可以到 nodejs.org 网站上复制对应的 tar 压缩包进行下载，比如你用的 mac 或 linux,可以输入以下命令下载

```sh
wget https://nodejs.org/dist/v8.9.4/node-v8.9.4.tar.gz
```

```sh
curl -O https://nodejs.org/dist/v8.9.4/node-v8.9.4.tar.gz
```

### 4.3 编译源码

对 tar 压缩包进行解压缩

- x extract 解包
- f file 要解包的是个文件
- z gzip 这个包是压缩过的，需要解压缩
- v verbose 把解包过程告诉你

```sh
tar xfz node-v8.9.4.tar.gz
```

进入源代码目录

```sh
cd node-v8.9.4
```

对其进行配置

```sh
./configure
```

现在就开始编译了

```sh
make
```

编译生成 Node 可执行文件后，就可以按以下的指令开始安装

```sh
make install
```

以上指令会将 Node 可执行文件复制到/user/local/bin/node 目录下
执行以上指令如果遇到权限问题，可以以 root 用户权限执行

```sh
sudo make install
```
