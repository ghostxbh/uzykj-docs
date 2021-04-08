---
title: Vue.js开发环境
date: 2019-06-21
tags:
    - Vue.js
    - Vue-CLI
author: ghostxbh
location: blog
summary: 搭建Vue.js的开发环境，及使用Vue-CLI脚手架命令。
---
# 开发环境

- 官方推荐命令，Vue 不支持 IE8 及以下版本，因为 Vue 使用了 IE8 无法模拟的 ECMAScript 5 特性。
- 必须有[nodejs](http://nodejs.cn/download/)环境
- [nodejs-linux](http://blog.uzykj.com/content?cid=17#)

### CLI
```shell
npm install -global vue-cli / cnpm install -global vue-cli
```

### 创建&启动项目（webpack创建）
```shell
#创建
vue init webpack vuedemo

#进入项目路径
cd vuedemo

#安装依赖
npm install / cnpm install

#启动
npm run dev
```

### 2.创建&启动项目（webpack-simple创建）
```shell
#创建
vue init webpack-simple vuedemo

#进入项目路径
cd vuedemo

#安装依赖
npm install / cnpm install

#启动
npm run dev
```

### vue-cli 2.X升级3.X
```shell
npm uninstall -global vue-cli
```
**升级3.X之前需要卸载2.X**

### 安装vue-cli 3.X(以下3种命令都可以安装)
```shell
npm install -g @vue/cli 
cnpm install -g @vue/cli 
yarn global add @vue/cli
```

### 创建&启动项目
```shell
#创建项目
vue create vuedome01

#进入项目路径
cd vuedemo01

#启动项目
npm run serve

#编译项目
npm run build
```
创建项目可能会遇到选择，直接选择默认就行`default`

![default](http://file.uzykj.com/vue-cli-log.png)

### vue图形化界面创建项目
- 1.命令窗口运行
```shell
vue ui
```

- 2.进入ui界面（建议使用chrome浏览器）

![manager](http://file.uzykj.com/vue-ui.png)

- 3.点击创建项目

![config](http://file.uzykj.com/vue-ui-config.png)

### cnpm的安装(淘宝镜像)
```shell
#地址：http://npm.taobao.org/	
#安装cnpm:

npm install -g cnpm --registry=https://registry.npm.taobao.org
```

### 参考资料
[vuejs安装](https://cn.vuejs.org/v2/guide/installation.html)

---
收录时间: 2019-06-21

<Vssue :title="$title" />


