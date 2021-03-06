---
title: MongoDB 删除文档
date: 2019-08-30
tags:
    - Egg.js
    - MongoDB
    - Mongoose
author: ghostxbh
location: BeiJing
summary: 使用Egg进行MongoDB 删除文档。
---
## MongoDB 删除文档
MongoDB `remove()`函数是用来移除集合中的数据。

MongoDB数据更新可以使用`update()`函数。
在执行`remove()`函数前先执行`find()`命令来判断执行的条件是否正确，
这是一个比较好的习惯。

### 语法
`remove()` 方法的基本语法格式如下所示：
```bash
db.collection.remove(
   <query>,
   <justOne>
)
```
如果你的 MongoDB 是 2.6 版本以后的，语法格式如下：
```bash
db.collection.remove(
   <query>,
   {
     justOne: <boolean>,
     writeConcern: <document>
   }
)
```
### 参数说明：

query :（可选）删除的文档的条件。
justOne : （可选）如果设为 true 或 1，则只删除一个文档。
writeConcern :（可选）抛出异常的级别。
### 实例
```bash
db.col.remove({'_id':'56066169ade2f21f36b03137'})
WriteResult({ "nRemoved" : 1 })    # 删除了一条数据
```

### egg操作mongodb
+ [user model](https://github.com/ghostxbh/egg-mongodb-demo/blob/master/app/model/user.js)

+ [mongoose](https://github.com/ghostxbh/egg-mongodb-demo/blob/master/app/service/user.js) 操作
```js
async delete(id) {
    const {ctx} = this;
    return await ctx.model.User.delete({_id: id});
}
```

### 参考资料
[delete](https://www.w3cschool.cn/mongodb/mongodb-remove.html)


---
收录时间: 2019-08-30

<Vssue :title="$title" />
