## 安装工具包

```bash
npm install -g yo generator-code
```

## 创建工程

```
yo code
? What type of extension do you want to create? New Extension (TypeScript)
? What's the name of your extension? vscode-react
? What's the identifier of your extension? vscode-react
? What's the description of your extension? react plugin
? Initialize a git repository? Yes
? Bundle the source code with webpack? Yes
? Which package manager to use? npm
```

## 目录结构

```
.vscode\extensions.json：// 用于存储当前项目所需要的 VS Code 扩展列表。
.vscode\launch.json：// 用于配置 VS Code 的调试器，指定调试目标和启动参数等。
.vscode\settings.json：// 用于存储当前项目的 VS Code 设置，例如缩进、字体、语言等。
.vscode\tasks.json：// 用于配置 VS Code 的任务，例如自动构建、测试、部署等。
package.json：// 用于描述当前项目的元数据信息，例如名称、版本、依赖等。
tsconfig.json：// 用于配置 TypeScript 项目的编译选项，例如目标版本、模块规范等。
.vscodeignore：// 用于指定需要在项目中忽略的文件或文件夹。
webpack.config.js：// 用于配置 Webpack 构建工具的参数，例如入口文件、输出文件、插件等。
vsc-extension-quickstart.md：// 用于指导用户如何快速创建 VS Code 扩展项目。
.gitignore：// 用于指定需要在 Git 版本控制中忽略的文件或文件夹。
README.md：// 用于存储项目的介绍、用法、注意事项等。
CHANGELOG.md：// 用于存储项目版本的变更记录和更新内容。
src\extension.ts：// 扩展的入口文件，用于注册命令、提供功能等。
src\test\runTest.ts：// 用于测试扩展中的功能。
src\test\suite\extension.test.ts：// 扩展功能的单元测试代码。
src\test\suite\index.ts：// 单元测试的入口文件。
.eslintrc.json：// 用于配置 ESLint 检查工具的规则，例如变量命名、代码风格等。
```

## 添加 activitybar 图标

- 要将一个自定义视图添加到活动栏中，需要将它添加到 `viewsContainers` 中
- viewsContainers 中定义了扩展可以添加视图的容器。通常，`activitybar` 是最常用的容器，它包含了常见的操作和功能视图，如`资源管理器`、`搜索`等
