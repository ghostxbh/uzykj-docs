---
title: MongoDB 连接
date: 2019-08-28
tags:
    - Egg.js
    - MongoDB
    - Mongoose
author: ghostxbh
location: BeiJing
summary: 使用Egg进行MongoDB 连接。
---
## MongoDB 连接

### 启动 MongoDB服务
只需要在MongoDB安装目录的bin目录下执行`'mongod'`即可。

执行启动操作后，mongodb在输出一些必要信息后不会输出任何信息，之后就等待连接的建立，当连接被建立后，就会开始打印日志信息。

你可以使用`MongoDB shell` 来连接 `MongoDB` 服务器。你也可以使用PHP来连接mongodb。本教程我们会使用 MongoDB shell来连接Mongodb服务，之后的章节我们将会介绍如何通过php 来连接MongoDB服务。

默认情况下，MongoDB的启动端口为**27017**。比MongoDB启动端口大1000的端口为MongoDB的web用户界面，你可以再浏览器中输入http://localhost:28017 来访问MongoDB的web用户界面。

MongoDB连接命令格式
使用用户名和密码连接到MongoDB服务器，你必须使用 `'username:password@hostname/dbname'` 格式，`'username'`为用户名，`'password'` 为密码。

使用用户名和密码连接登陆到默认数据库：

```bash
mongodb://mongo_admin:AxB6_w3r@localhost/
```
 
以上命令中，用户 mongo_admin使用密码AxB6_w3r连接到本地的MongoDB服务上。

使用用户名和密码连接登陆到指定数据库：

连接到指定数据库的格式如下：
```bash
mongodb://mongo_admin:AxB6_w3r@localhost/w3r
``` 
 
使用用户名fred，密码foobar登录localhost的admin数据库。
```bash
mongodb://fred:foobar@localhost
```
 
使用用户名fred，密码foobar登录localhost的baz数据库
```bash
mongodb://fred:foobar@localhost/baz
```

连接 replica pair, 服务器1为example1.com服务器2为example2。
```bash
mongodb://example1.com:27017,example2.com:27017
```

连接 replica set 三台服务器 (端口 27017, 27018, 和27019):
```bash
mongodb://localhost,localhost:27018,localhost:27019
```

连接 replica set 三台服务器, 写入操作应用在主服务器 并且分布查询到从服务器。
```bash
mongodb://host1,host2,host3/?slaveOk=true
```
 
直接连接第一个服务器，无论是replica set一部分或者主服务器或者从服务器。
```bash
mongodb://host1,host2,host3/?connect=direct;slaveOk=true
```
 
当你的连接服务器有优先级，还需要列出所有服务器，你可以使用上述连接方式。

安全模式连接到localhost:
```bash
mongodb://localhost/?safe=true
```

以安全模式连接到replica set，并且等待至少两个复制服务器成功写入，超时时间设置为2秒。
```bash
mongodb://host1,host2,host3/?safe=true;w=2;wtimeoutMS=2000
```

参数选项说明
标准格式：
```bash
mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]
```

标准的连接格式包含了多个选项(options)，如下所示：

|选项|描述|
|---|---|
|replicaSet=name|验证replica set的名称。 Impliesconnect=replicaSet.|
|slaveOk=true/false|1、true:在connect=direct模式下，驱动会连接第一台机器，即使这台服务器不是主。在connect=replicaSet模式下，驱动会发送所有的写请求到主并且把读取操作分布在其他从服务器。|
| |2、false: 在 connect=direct模式下，驱动会自动找寻主服务器. 在connect=replicaSet 模式下，驱动仅仅连接主服务器，并且所有的读写命令都连接到主服务器。|
|safe=true/false|1、true: 在执行更新操作之后，驱动都会发送getLastError命令来确保更新成功。(还要参考 wtimeoutMS).|
| |2、false: 在每次更新之后，驱动不会发送getLastError来确保更新成功。|
|w=n|驱动添加 { w : n } 到getLastError命令. 应用于safe=true。|
|wtimeoutMS=ms|驱动添加 { wtimeout : ms } 到 getlasterror 命令. 应用于 safe=true.|
|fsync=true/false|1、true: 驱动添加 { fsync : true } 到 getlasterror 命令.应用于 safe=true.|
| |2、false: 驱动不会添加到getLastError命令中。
|journal=true/false|如果设置wie true, 同步到 journal (在提交到数据库前写入到实体中). 应用于 safe=true|
|connectTimeoutMS=ms|可以打开连接的时间。|
|socketTimeoutMS=ms|发送和接受sockets的时间。|

### egg mongoose配置
config/plugin.js
```js
exports.mongoose = {
    enable: true,
    package: 'egg-mongoose',
};
```

config/config.default.js
```js
module.exports = appInfo => {
    /**
     * built-in config
     * @type {Egg.EggAppConfig}
     **/
    const config = exports = {};
    // mongoose
    config.mongoose = {
        url: 'mongodb://127.0.0.1/apiExample'
    };
}
```

### 参考资料
[连接](https://www.w3cschool.cn/mongodb/mongodb-connections.html)


---
收录时间: 2019-08-28

<Vssue :title="$title" />
