---
title: 引入和编译
date: 2022-03-01
sidebar: 'auto'
categories:
  - Javascript
tags:
  - TypeScript
  - TS
  - 编译
  - 引入
author: tasaid
location: blog
summary: 如果我们想快速测试一个文件，可以使用 ts-node，ts-node 可以让我们通过命令直接执行 *.ts 文件
---


# 引入和编译
## 快速使用
安装：
```shell
$ npm i -g typescript
```

编译：
```shell
$ tsc helloworld.ts
```

如果我们想快速测试一个文件，可以使用 `ts-node`，`ts-node` 可以让我们通过命令直接执行 `*.ts` 文件：
```shell
$ npm i -g ts-node
# 执行当前文件夹下 demo.ts 文件

$ ts-node demo.ts
# 输出的执行结果
```

关于 `tsc`，一般来说，全局安装的方案并不是很棒，我们可以把 typescript 安装到本地项目目录中：
```shell
$ npm i  --save-dev typescript
```
这样在 `package.json` 的 `scripts` 中可以引用 `tsc` 命令：
```json
{
    "scripts": {
      "build": "tsc"
    },
    "devDependencies": {
      "typescript": "^2.4.2"
    }
}
```

使用 `npm run build` 命令即可启用 `tsc` 命令编译本地目录，typescript 会去查找目录下的 `tsconfig.json` 配置文件。

引入 TypeScript 非常简单，TypeScript 的文件后缀为 ts，迁移 TypeScript 只需要将项目中，业务代码的 `*.js` 修改为 `*.ts` 即可。不过在此之后你会看到大量的报错，然后就是按照 TypeScript 的规则，解决这些报错即可:)

![引入和编译-类型报错](https://static.tasaid.com/blogs/e1c340f248b8dad18c05d28c7a462680.png)

## tsconfig.json
`tsconfig.json` 是 TypeScript 的编译选项文件，通过配置它来定制 TypeScript 的编译细节。

直接调用 `tsc`，编译器会从当前目录开始去查找 `tsconfig.json` 文件，逐级向上搜索父目录。
调用 `tsc -p`，可以指定一个包含 `tsconfig.json` 文件的目录进行编译。如果没有找到 `tsconfig.json` 文件，TypeScript 会编译每个文件并在对应文件的同级目录产出。
> 如果你要编译的是一个 Node 项目，请先安装 Node 编译依赖： `npm i @types/node --save-dev`，否则会出现 Node 内置模块无法找到的情况。

一个 tsconfig.json 文件描述：
```
{
	// 编译选项
	"compilerOptions": {
		// 输出目录
		"outDir": "./output",
		// 是否包含可以用于 debug 的 sourceMap
		"sourceMap": true,
		// 以严格模式解析
		"strict": true,
		// 采用的模块系统
		"module": "esnext",
		// 如何处理模块
		"moduleResolution": "node",
		// 编译输出目标 ES 版本
		"target": "es5",
		// 允许从没有设置默认导出的模块中默认导入
		"allowSyntheticDefaultImports": true,
		// 将每个文件作为单独的模块
		"isolatedModules": false,
		// 启用装饰器
		"experimentalDecorators": true,
		// 启用设计类型元数据（用于反射）
		"emitDecoratorMetadata": true,
		// 在表达式和声明上有隐含的any类型时报错
		"noImplicitAny": false,
		// 不是函数的所有返回路径都有返回值时报错。
		"noImplicitReturns": true,
		// 从 tslib 导入外部帮助库: 比如__extends，__rest等
		"importHelpers": true,
		// 编译过程中打印文件名
		"listFiles": true,
		// 移除注释
		"removeComments": true,
		"suppressImplicitAnyIndexErrors": true,
		// 允许编译javascript文件
		"allowJs": true,
		// 解析非相对模块名的基准目录
		"baseUrl": "./",
		// 指定特殊模块的路径
		"paths": {
			"jquery": [
				"node_modules/jquery/dist/jquery"
			]
		},
		// typescript 语法检测支持的版本库，注意不是 polyfill！只是为了有对应版本的代码特性提示！
		"lib": [
			"es2015",
			"es2015.promise"
		]
	}
}
```

完整 `tsconfig` 配置选项的可以参考 [这里](https://www.tslang.cn/docs/handbook/compiler-options.html) ，或者 `tsconfig` 的 [json-schema](http://json.schemastore.org/tsconfig) 。

注意： TypeScript 不会做 Polyfill，例如从 `es6` 编译到 `es5`，TypeScript 编译后不会处理 `es6` 的那些新增的对象的方法，如果需要 `polyfill` 需要自行处理！

完整的编译选项请参阅 [TypeScript 中文网](https://www.tslang.cn) 和 [TypeScript 官网](https://www.typescriptlang.org/) 。

## 编译
大多数前端已经使用各种各样的构建工具，完整构建工具集成列表参见 这里

对于 Node 项目，建议搭配 `gulp` 使用。不过个人更喜欢通过 `npm scripts` 脚本组合命令，然后直接使用 `tsc` 编译，例如我自己的编译方案。

项目目录为：
```
|---output # 编译输出
|---client
|---server # node ts 文件目录
    |--tsconfig.json
|---package.json
```

`package.json` 中 `scripts` 脚本如下：
```
"scripts": {
    "dev": "nodemon --ext ts --watch server --exec \"npm run clean && npm run build:ts && npm run server\"",
    "server": "cross-env NODE_ENV=development node ./output/app.js",
    "clean": "rm -rf ./output.server",
    "build:ts": "tsc -p ./server",
    "build": "npm run clean && npm run build:ts"
}
```

- 使用 `nodemon` 监听整个 server 目录文件改动并执行脚本
- `npm run clean` 用于清空编译输出目录
- `npm run build:ts` 用于编译 server 目录下的 TypeScript 文件
- `npm run server` 用于启动编译后的 node 服务器。

平常开发只需要 `npm run dev`，生产使用 `npm run build` 产出文件即可。

## visual studio code 集成和 debug
### 编译任务
Visual Studio Code 编译 TypeScript 非常简单，根据上面我自己的组合命令，只需要在 vs code 的任务中加入编译脚本即可（`npm run build-ts`）：

`./.vscode/task.json` 如下：
```
{
	"version": "2.0.0",
	"tasks": [{
		// npm 命令
		"type": "npm",
		// npm run 的脚本
		"script": "build-ts",
		// 标签
		"label": "build-typescript",
		// 默认任务
		"group": {
			"kind": "build",
			"isDefault": true
		}
	}]
}
```

按 `control+shift+B` 即可编译。

![编译 Code](https://static.tasaid.com/blogs/ca484d61d09a03e92c816b2539c39767.jpeg)

### debug
在 `./.vscode/launch.json` 中加入如下代码即可调试，记得要在 `tsconfig.json` 里打开 `sourceMap` 选项：
```json
{
	"version": "0.2.0",
	"configurations": [{
		// 调试前运行的任务 (task)，就是上面编译任务中的 label
		"preLaunchTask": "build-typescript",
		// 调试任务名称
		"name": "server debug",
		"env": {
			// 传递的参数
			"NODE_ENV": "development"
		},
		// 调试的 node 入口文件，注意 tsconfig.json 里面要打开 sourceMap
		"program": "${workspaceRoot}/output/app.js"
	}]
}
```

然后在 vs code 中给代码打上断点，按 `F5`，一步步调试即可。

![vscode debug](https://static.tasaid.com/blogs/b3aaefb3dcfacd5a6efd35abd0d5a50a.gif)

## 结语

- 使用 `ts-node` 可以让我们直接运行 `*.ts` 文件（不过只建议临时运行代码或特殊应用场景使用）
- 使用 typescript 产出的 `tsc` 命令来编译 `*.ts` 文件到 `*.js`，然后我们运行编译出来的 `*.js` 文件即可
- 使用 `tsconfig.json` 配置 TypeScript 的编译选项


---
收录时间: 2022-03-01

<Vssue :title="$title" />