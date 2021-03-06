---
title: MongoDB 删除数据库
date: 2019-08-29
tags:
    - Egg.js
    - MongoDB
    - Mongoose
author: ghostxbh
location: BeiJing
summary: 使用Egg进行MongoDB 删除数据库。
---
## MongoDB 删除数据库

### 语法
MongoDB 删除数据库的语法格式如下：
```bash
db.dropDatabase();
```
删除当前数据库，默认为 test，你可以使用 db 命令查看当前数据库名。

### 实例
以下实例我们删除了数据库 youj。

> 首先，查看所有数据库：

```bash
> show dbs
local   0.078GB
youj  0.078GB
test    0.078GB
```

> 接下来我们切换到数据库 youj：

```bash
> use youj
switched to db youj
```

> 执行删除命令：

```bash
> db.dropDatabase()
{ "dropped" : "youj", "ok" : 1 }
```

> 最后，我们再通过 show dbs 命令数据库是否删除成功：

```bash
> show dbs
local  0.078GB
test   0.078GB
```

### 删除集合
集合删除语法格式如下：
```bash
db.collection.drop()
```

### 参考资料
[删除数据集](https://www.w3cschool.cn/mongodb/mongodb-dropdatabase.html)


---
收录时间: 2019-08-29

<Vssue :title="$title" />
