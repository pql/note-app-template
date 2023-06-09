## 1. npm script

### 1.1 快速创建项目

```sh
npm init
npm init -f
npm init -y
```

```sh
npm config set init.author.email "zhufengnodejs@126.com"
npm config set init.author.name "zhufengnodejs"
npm config set init.author.url "http://github.com/zhufengnodejs"
npm config set init.license "MIT"
npm config set init.version "0.0.1"
```

### 1.2 用 npm run 执行命令

- npm run 实际上执行的是 `npm run-script`
- 从`package.json`文件中读取 scripts 对象里面的全部配置
- 以传给`npm run`的第一个参数作为键,在 scripts 对象里面获取对应的值作为接下来要执行的命令,如果没找到直接报错
- 在系统默认的 shell 中执行上述命令，系统默认 shell 通常是 bash，windows 环境下不太一样

```json
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "eslint": "eslint --version"
}
```

```sh
npm run eslint
```
