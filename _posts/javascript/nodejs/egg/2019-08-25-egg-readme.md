---
title: Egg 示例
date: 2019-08-25
tags:
    - Egg.js
author: ghostxbh
location: BeiJing
summary: Egg 示例。
---
# Egg 示例

[![star](https://badgen.net/github/stars/ghostxbh/egg-mongodb-demo?icon=github&color=4ab8a1)](https://github.com/ghostxbh/egg-mongodb-demo/stargazers)
[![fork](https://badgen.net/github/forks/ghostxbh/egg-mongodb-demo?icon=github&color=4ab8a1)](https://github.com/ghostxbh/egg-mongodb-demo/members)

**基于macbook环境下**

## 快速开始

<!-- add docs here for user -->

去 [egg docs][egg] 官网查看详情.

### 开发

```bash
$ npm i
$ npm run dev
$ open http://localhost:7001/
```

### 部署

```bash
$ npm start
$ npm stop
```

### npm 脚本

- 使用 `npm run lint` 检查代码风格.
- 使用 `npm test` 进行单元测试.
- 使用 `npm run autod` 自动检测依赖项升级, 请参阅 [autod](https://www.npmjs.com/package/autod) 了解更多详情.

### 导入数据

- [GET] http://127.0.0.1:7001/user/load

[egg]: https://eggjs.org

## Eggjs 操作 mongodb 教程

基于`egg-mongoose`组件操作**mongodb**
+ 连接
    + [连接](files/blogs/blog/javascript/nodejs/egg/egg-mongo-connection.md)
+ 创建 | 删除 数据集
    + [创建数据集](files/blogs/blog/javascript/nodejs/egg/egg-mongo-createdb.md)
    + [删除数据集](files/blogs/blog/javascript/nodejs/egg/egg-mongo-dropdb.md)
+ 增删改查
    + [增加](files/blogs/blog/javascript/nodejs/egg/egg-mongo-create.md)
    + [删除](files/blogs/blog/javascript/nodejs/egg/egg-mongo-delete.md)
    + [修改](files/blogs/blog/javascript/nodejs/egg/egg-mongo-update.md)
    + [查找](files/blogs/blog/javascript/nodejs/egg/egg-mongo-find.md)

---
收录时间: 2019-08-25

<Vssue :title="$title" />
