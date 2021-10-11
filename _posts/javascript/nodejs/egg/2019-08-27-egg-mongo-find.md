---
title: MongoDB 查询文档
date: 2019-08-27
sidebar: 'auto'
categories:
  - Javascript
tags:
  - Egg.js
  - MongoDB
  - Mongoose
author: ghostxbh
location: blog
summary: 使用Egg进行MongoDB 查询文档。
---
## MongoDB 查询文档

### 语法
MongoDB 查询数据使用`find()`函数：
```bash
db.collection.find();
```

`find()` 方法以非结构化的方式来显示所有文档。

如果你需要以易读的方式来读取数据，可以使用 `pretty()` 方法，语法格式如下：
```bash
db.collection.find().pretty();
```
**注意：** 除了 `find()` 方法之外，还有一个 `findOne()` 方法，它只返回一个文档。

`pretty()` 方法以格式化的方式来显示所有文档。

`MongoDB` 与 `RDBMS Where` 语句比较

|操作|格式|范例|RDBMS中的类似语句|
|---|---|---|----------------|
|等于|{\<key>:\<value>}|db.col.find({"by":"w3cschool"}).pretty()|where by = 'w3cschool'|
|小于|{\<key>:{$lt:\<value>}}|db.col.find({"likes":{$lt:50}}).pretty()|where likes \< 50|
|小于或等于|{\<key>:{$lte:\<value>}}|db.col.find({"likes":{$lte:50}}).pretty()|where likes \<= 50|
|大于|{\<key>:{$gt:\<value>}}|db.col.find({"likes":{$gt:50}}).pretty()|where likes > 50|
|大于或等于|{\<key>:{$gte:\<value>}}|db.col.find({"likes":{$gte:50}}).pretty()|where likes >= 50|
|不等于|{\<key>:{$ne:\<value>}}|db.col.find({"likes":{$ne:50}}).pretty()|where likes != 50|

### MongoDB AND 条件
MongoDB 的 `find()` 方法可以传入多个键(key)，每个键(key)以逗号隔开，及常规 SQL 的 AND 条件。

语法格式如下：
```bash
db.collection.find({key1:value1, key2:value2}).pretty()
```

### MongoDB OR 条件
MongoDB OR 条件语句使用了关键字 `$or`,语法格式如下：
```bash
db.collection.find(
   {
      $or: [
	     {key1: value1}, {key2:value2}
      ]
   }
).pretty()
```

### AND 和 OR 联合使用
以下实例演示了 `AND` 和 `OR` 联合使用，类似常规 SQL 语句为： `'where likes>50 AND (by = 'w3cschool' OR title = 'MongoDB 教程')'`
```bash
db.collection.find({"likes": {$gt:50}, $or: [{"by": "table"},{"title": "MongoDB"}]}).pretty()
{
        "_id" : ObjectId("56063f17ade2f21f36b03133"),
        "title" : "MongoDB",
        "description" : "MongoDB 是一个 Nosql 数据库",
        "by" : "table",
        "url" : "http://blog.uzykj.com",
        "tags" : [
                "mongodb",
                "database",
                "NoSQL"
        ],
        "likes" : 100
}
```

### egg操作mongodb
+ [user model](https://github.com/ghostxbh/egg-mongodb-demo/blob/master/app/model/user.js)

+ [mongoose](https://github.com/ghostxbh/egg-mongodb-demo/blob/master/app/service/user.js) 操作

```js
async find(id) {
    const {ctx} = this;
    return await ctx.model.User.find({_id: id});
}
```

### 参考资料
[find](https://www.w3cschool.cn/mongodb/mongodb-query.html)


---
收录时间: 2019-08-27

<Vssue :title="$title" />
