---
title: MongoDB 插入文档
date: 2019-08-27
tags:
    - Egg.js
    - MongoDB
    - Mongoose
author: ghostxbh
location: BeiJing
summary: 使用Egg进行MongoDB创建文档。
---
## MongoDB 插入文档

文档的数据结构和JSON基本一样。

所有存储在集合中的数据都是BSON格式。

BSON是一种类json的一种二进制形式的存储格式,简称Binary JSON。

### 插入文档
MongoDB 使用 `insert()` 或 `save()` 方法向集合中插入文档，语法如下：

```bash
db.COLLECTION_NAME.insert(document)
```

### 实例
以下文档可以存储在 MongoDB 的 w3cschool.cn数据库 的 col集合中：

``` bash
db.col.insert({
    "name":"小玉",
    "password":"xiaoyu123",
    "age":19,
    "gender":"2",
    "phone":"17786187724",
    "email":"xiaoyu@163.com"
  })
```
### egg操作mongodb
+ [user model](https://github.com/ghostxbh/egg-mongodb-demo/blob/master/app/model/user.js)

+ [mongoose](https://github.com/ghostxbh/egg-mongodb-demo/blob/master/app/service/user.js) 操作
```js
async create(user) {
    const {ctx} = this;
    user.password = ctx.helper.encrypt(user.password);
    return await ctx.model.User.create(user);
}
```

### 参考资料
[create](https://www.w3cschool.cn/mongodb/mongodb-insert.html)


---
收录时间: 2019-08-27

<Vssue :title="$title" />
