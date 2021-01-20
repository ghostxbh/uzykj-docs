## MongoDB 创建数据库

### 语法
MongoDB 创建数据库的语法格式如下：
```bash
use DATABASE_NAME
```

如果数据库不存在，则创建数据库，否则切换到指定数据库。

实例
以下实例我们创建了数据库 youj:
```bash
> use youj
switched to db youj
> db
youj 
```

如果你想查看所有数据库，可以使用 show dbs 命令：
```bash
> show dbs
local  0.078GB
test   0.078GB
```

可以看到，我们刚创建的数据库 youj 并不在数据库的列表中， 要显示它，我们需要向 youj 数据库插入一些数据。
```bash
> db.youj.insert({"name":"mongodb教程"})
WriteResult({ "nInserted" : 1 })
> show dbs
local   0.078GB
youj  0.078GB
test    0.078GB
```

MongoDB 中默认的数据库为 test，如果你没有创建新的数据库，集合将存放在 test 数据库中。

### egg mongoose
定义好model => [user](https://github.com/ghostxbh/egg-mongodb-demo/blob/master/app/model/user.js)

成功添加一条数据即可以创建数据集

### 参考资料
[创建数据集](https://www.w3cschool.cn/mongodb/mongodb-create-database.html)


---
收录时间: 2019/08/27

<Vssue :title="$title" />
