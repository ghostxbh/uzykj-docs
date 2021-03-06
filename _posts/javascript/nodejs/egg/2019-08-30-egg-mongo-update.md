---
title: MongoDB 更新文档
date: 2019-08-30
tags:
    - Egg.js
    - MongoDB
    - Mongoose
author: ghostxbh
location: BeiJing
summary: 使用Egg进行MongoDB 更新文档。
---
## MongoDB 更新文档

MongoDB数据更新可以使用`update()`函数。
### 语法
```bash
db.collection.update({<where>},{<document>})
```

### 参数说明
+ **criteria** : `update`的查询条件，类似`sql update`查询内`where`后面的。
+ **objNew** : `update`的对象和一些更新的操作符（如`$,$inc...`）等，也可以理解为`sql update`查询内`set`后面的
+ **upsert** : 这个参数的意思是，如果不存在`update`的记录，是否插入`objNew`,true为插入，默认是false，不插入。
+ **multi** : mongodb默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。

### 示例
```bash
db.userdetails.update(
    {
        "user_id" : "QRSTBWN"
    },
    {
        "user_id" : "QRSTBWN",
        "password" :"NEWPASSWORD",
        "date_of_join" : "17/10/2010",
        "education" :"M.B.A.",
        "profession" : "MARKETING",
        "interest" :"MUSIC",
        "community_name" :[
            "MODERN MUSIC", 
            "CLASSICAL MUSIC",
            "WESTERN MUSIC"
            ],
    }
)
```
### egg操作mongodb
+ [user model](https://github.com/ghostxbh/egg-mongodb-demo/blob/master/app/model/user.js)

+ [mongoose](https://github.com/ghostxbh/egg-mongodb-demo/blob/master/app/service/user.js) 操作

```js
async update(user) {
    const {ctx} = this;
    user.password = ctx.helper.encrypt(user.password);
    return await ctx.model.User.update({'_id':user._id},user);
}
```

### 参考资料
[update](https://www.w3cschool.cn/mongodb/mongodb-update.html)


---
收录时间: 2019-08-30

<Vssue :title="$title" />
