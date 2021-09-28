---
title: Node.js中的进程管理 
date: 2021-08-30 
tags:
    - Nodejs 
    - process 
    - cluster 
    - child_process 
author: huangtengfei 
location: blog 
summary: 本文主要对Node.js中进程管理相关的东西做一个简单介绍，包括 process 对象、child_process 模块和 cluster 模块，详细的 API 可以查看官方文档。
---

# Node.js中的进程管理

本文主要对 Node.js 中进程管理相关的东西做一个简单介绍，包括 process 对象、child_process 模块和 cluster 模块，详细的 API 可以查看官方文档。

## Process 对象

`process` 是 Node.js 的一个全局对象，可以在任何地方直接使用而不需要 `require` 命令加载。
`process` 对象提供了 当前 node 进程 的命令行参数、标准输入输出、运行环境和运行状态等信息。

### 常用属性

#### argv

`process.argv` 属性返回一个数组，第一个元素是 node，第二个元素是脚本文件名称，其余成员是脚本文件的参数。

```shell
$ node process-2.js one two=three four

0: /usr/local/bin/node
1: /Users/mjr/work/node/process-2.js
2: one
3: two=three
4: four
```

#### env

`process.env` 返回一个对象，包含了当前 Shell 的所有环境变量，比如：

```json
{
  "TERM": "xterm-256color",
  "SHELL": "/bin/zsh",
  "USER": "huangtengfei",
  "PATH": "~/.bin/:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin",
  "PWD": "/Users/huangtengfei",
  "HOME": "/Users/huangtengfei"
}
```

这个属性通常的使用场景是，新建一个 NODE_ENV 变量，用来确定当前所处的开发阶段，生成阶段设为 production， 开发阶段设为 develop ，然后在脚本中读取 `process.env.NODE_ENV`再做相应处理即可。

运行脚本时可以这样改变环境变量：

```shell
$ export NODE_ENV=production && node app.js
# 或者
$ NODE_ENV=production node app.js
```

#### stdin/stdout

`process.stdin` 指向标准输入（键盘到缓冲区里的东西），返回一个可读的流：

```js
process.stdin.setEncoding('utf8');

process.stdin.on('readable', () => {
  var chunk = process.stdin.read();
  if (chunk !== null) {
    process.stdout.write(`data: ${chunk}`);
  }
});

process.stdin.on('end', () => {
  process.stdout.write('end');
});
```

`process.stdout` 指向标准输出（向用户显示内容），返回一个可写的流：

```js
const fs = require('fs');

fs.createReadStream('wow.txt')
  .pipe(process.stdout);
```

### 常用方法

#### `cwd()`

`process.cwd()` 返回运行 Node 的工作目录（绝对路径），比如在目录 `/Users/huangtengfei/abc` 下执行 `node server.js`，那么 `process.cwd()`
返回的就是 `/Users/huangtengfei/abc`。

另一个常用的获取路径的方法是 `__dirname`，它返回的是执行文件时该文件在文件系统中所在的目录。注意 `process.cwd()` 和 `__dirname` 的不同，前者是进程发起时的位置，后者是脚本的位置，两者可能不一致。

#### `on()`

`process` 对象部署了 EventEmitter 接口，可以使用 `process.on()` 方法监听各种事件，并指定回调函数。比如监听到系统发出进程终止信号时关闭服务器然后退出进程：

```js
process.on('SIGTERM', function () {
  server.close(function () {
    process.exit(0);
  });
});
```

#### `exit()`

`process.exit()` 会让 Node 立即终止当前进程（同步），参数为一个退出状态码，0 表示成功，大于 0 的任意整数表示失败。

#### `kill()`

`process.kill()` 用来对特定 id 的进程（process.pid）发送信号，默认为 `SIGINT` 信号。比如杀死当前进程：

```js
process.kill(process.pid, 'SIGTERM');
```

虽然名字叫 kill ，但其实 `process.kill()` 只是负责发送信号，具体发送完信号之后这个怎么处理这个指定进程，取决于信号种类和接收到这个信号之后做了什么操作（比如 `process.exit()`
或者只是 `console.log('Ignored this single')`）。

## Child Process 模块

`child_process` 模块用于创建和控制子进程，其中最核心的是 .spawn() ，其他 API 算是针对特定场景对它的封装。使用前要先 require 进来：

```js
const cp = require('child_process');
```

### exec(command[, options][, callback])

`exec()` 方法用于执行 shell
命令，它的第一个参数是字符串形式的命令，第二个参数（可选）用来指定子进程运行时的定制化操作，第三个参数（可选）用来设置执行完命令的回调函数。比如在一个特定目录 `/Users/huangtengfei/abc` 下执行 `ls -l`
命令：

```js
cp.exec('ls -l', {
  cwd: '/Users/huangtengfei/abc'
}, (error, stdout, stderr) => {
  if (error) {
    console.error(`exec error: ${error}`);
    return;
  }
  console.log(`stdout: ${stdout}`);
  console.log(`stderr: ${stderr}`);
})
```

### spawn(command[, args][, options])

`spawn()` 用来创建一个子进程执行特定命令，与 `exec()` 的区别是它没有回调函数，只能通过监听事件来获取运行结果，它适用于子进程长时间运行的情况，可以实时输出结果。

```js
const ls = cp.spawn('ls', ['-l']);

ls.stdout.on('data', (data) => {
  console.log(`stdout: ${data}`);
});

ls.stderr.on('data', (data) => {
  console.log(`stderr: ${data}`);
});

ls.on('close', (code) => {
  console.log(`child process exited with code ${code}`);
});
```

使用 spawn 可以实现一个简单的守护进程，在工作进程不正常退出时重启工作进程：

```js
/* daemon.js */
function spawn(mainModule) {
  const worker = cp.spawn('node', [mainModule]);
  worker.on('exit', function (code) {
    if (code !== 0) {
      spawn(mainModule);
    }
  });
}

spawn('worker.js');
```

### fork(modulePath[, args][, options])

`fork()` 用来创建一个子进程执行 node 脚本，`fork('./child.js')` 相当于 `spawn('node', ['./child.js'])`，区别在于 fork
会在父子进程之间建立一个通信管道（`fork()` 的返回值），用于进程间通信。对该通信管道对象可以监听 `message` 事件，用来获取子进程返回的信息，也可以向子进程发送信息。

```js
/* main.js */
const proc = cp.fork('./child.js');
proc.on('message', function (msg) {
  console.log(`parent got message: ${msg}`);
});
proc.send({hello: 'world'});

/* child.js */
process.on('message', function (msg) {
  console.log(`child got message: ${msg}`);
});
process.send({foo: 'bar'});
```

## Cluster 模块

Node.js 默认单进程执行，但这样就无法利用多核计算机的资源，cluster 模块的出现就是为了解决这个问题的。在开发服务器程序时，可以通过 cluster 创建一个主进程和多个 worker 进程，让每个 worker
进程运行在一个核上，统一通过主进程监听端口和分发请求。

```js
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  console.log(`Master ${process.pid} is running`);

// Fork workers.
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`worker ${worker.process.pid} died`);
  });
} else {
// Workers can share any TCP connection
// In this case it is an HTTP server
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end('hello world\n');
  }).listen(8000);

  console.log(`Worker ${process.pid} started`);
}
```

### 常用属性和方法

#### isMaster/isWorker

`cluster.isMaster` 用来判断当前进程是否是主进程，`cluster.isWorker` 用来判断当前进程是否是工作进程，两者返回的都是布尔值。

#### workers

`cluster.workers` 是一个包含所有 `worker` 进程的对象，key 为 `worker.id`，value 为 `worker` 进程对象。

```js
// 遍历所有 workers
function eachWorker(callback) {
  for (const id in cluster.workers) {
    callback(cluster.workers[id]);
  }
}

eachWorker((worker) => {
  worker.send('big announcement to all workers');
});
```

#### fork([env])

`cluster.fork()` 方法用来新建一个 `worker` 进程，默认上下文复制主进程，只有主进程可调用。

### 常用事件

#### listening

在工作进程调用 `listen` 方法后，会触发一个 listening 事件，这个事件可以被 `cluster.on('listening')` 监听。

比如每当一个 worker 进程连进来时，输出一条 log 信息：

```js
cluster.on('listening', (worker, address) => {
  console.log(
    `A worker is now connected to ${address.address}:${address.port}`);
});
```

#### exit

在工作进程挂掉时，会触发一个 `exit` 事件，这个事件可以被 `cluster.on('exit')` 监听。

比如自动重启 worker：

```js
cluster.on('exit', (worker, code, signal) => {
  console.log('worker %d died (%s). restarting...',
    worker.process.pid, signal || code);
  cluster.fork();
});
```

### worker 对象

`worker` 对象是 `cluster.fork()` 的返回值，代表一个 `worker` 进程。

#### worker.id

`worker.id` 是当前 `worker` 的唯一标识，也是保存在 `cluster.workers` 中的 key 值。

#### worker.process

所有的 `worker` 进程都是通过 `child_process.fork()` 生成的，这个进程对象保存在 `worker.process` 中。

#### worker.send()

`worker.send()` 用在主进程给子进程发送消息，在子进程中，使用 `process.on()` 监听消息并使用 `process.send()` 发送消息。

```js
if (cluster.isMaster) {
  const worker = cluster.fork();
  worker.send('hi there');
} else if (cluster.isWorker) {
  process.on('message', (msg) => {
    process.send(msg);
  });
}
```

## 参考 
- [Node.js 官方文档](https://nodejs.org/api/process.html)
- [JavaScript 标准教程 - Node.js](http://javascript.ruanyifeng.com/nodejs/process.html)

> 转载自：[Node.js 中的进程管理](http://huangtengfei.com/2017/03/process/) 


---
收录时间: 2021-08-30

<Vssue :title="$title" />
