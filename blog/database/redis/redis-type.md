# Redis类型

![p](http://file.uzykj.com/48040a58-df90-9fa5-232c-4f915d4598d0.png)

- string
- list
- hash
- set
- zset


## string

### 常用操作
```
// 存入字符串k-v
SET key value

// 批量存入字符串k-v
MSET key value [key value...]

// 存入一个不存在的字符串k-v
SETNX key value

// 获取一个字符串键值
GET key

// 批量获取字符串键值
MGET key [key...]

// 删除一个/多个键
DEL key [key...]

// 设置一个键的过期时间(秒)
EXPIRE key seconds
```

### 原子加减
```
// 储存key的数字加一
INCR key 

// 储存key的数字减一
DECR key

// 储存key的值加上increment（增量）
INCRBY key increment

// 储存key的值加上decrement（减量）
DECRBY key decrement
```

### 应用场景

- 单值缓存
```shell
# SET key value

localhost:0>SET test 我是一个value
"OK"
```

- 对象缓存
```shell
# SET object:1 value(json数据)

# redis shell
localhost:0>SET object:1 "{\"name\": \"Xiaoming\", \"age\": 18}\"
"OK"
```

- 分布式锁
```shell
// 线程A获取锁成功 
SETNX product:1 true

// 线程B获取锁失败
SETNX product:1 false
```
开启多个线程下，争抢锁，
线程A抢到了锁，
线程B...获取锁就会失败

```shell
// 执行业务。。。

// 线程A释放锁
DEL product:1
```
执行完业务流程后，需要释放掉这个锁
```shell
SETNX product:1 ture ex 10 nx
```
防止程序意外终止导致死锁

- 计数器
![p](http://file.uzykj.com/828c9f8a-8f0a-588e-22cd-95df1c8ec11f.png)

```shell script
INCR article:readcount:{文章ID}

GET article:readcount:{文章ID}
```

- WEB集群Session共享

`Spring Session` + `Redis`实现`session`共享

- 分布式系统全局序列号
```
// redis 批量生成序列号提升性能
INCRBY orderId 1000
```
分库分表时，使用redis中使用`INCR`做自增序列，不会存在重复ID

服务集群大时，redis的`INCR`压力也大，可使用分批次拿取区间自增序列

```
A server 1-1000
B server 1001-2000
C server 2001-3000
...
```
各自序列在服务的内存中做递增逻辑

减小对redis的`INCR`的频繁操作

## hash

### 常用操作
```
// 储存一个哈希表key的键值
HSET key field value

// 储存一个不存在的哈希表key的键值
HSETNX key field value

// 在一个key的哈希表中储存多个键值
HMSET key field value [field value...]

// 获取哈希表key的field对应键值
HGET key field

// 获取哈希表key的多个field对应键值
HMGET key field [field...]

// 删除哈希表key的field键值
HDEL key field [field...]

// 返回哈希表key的field的数量
HLEN key

// 返回哈希表key所有的键值
HGETALL key

// 为哈希表key的field键加上increment增量
HINCRBY key field increment
```

### 应用场景
- 对象缓存
MySQL表
![p](http://file.uzykj.com/279e7f90-e21f-662e-3fc8-1f3cf66ad2eb.png)
转换
![p](http://file.uzykj.com/a11169e9-4cde-32bf-5b9c-189a5499ed7a.png)

```
HMSET user {userId}:name 姓名 {userId}:balance 余额

HMSET user 10:name zhangsan 10:balance 1000

HMGET user 10:name 10:balance

# redis shell
localhost:0>HMSET user 10:name zhangsan 10:balance 1000
"OK"
localhost:0>HMGET user 10:name 10:balance
 1)  "zhangsan"
 2)  "1000"
```

- 电商购物车
![p](http://file.uzykj.com/7868a794-72c1-240e-48e0-78e069a271d5.png)
    + 以用户ID为key
    + 商品ID为field
    + 商品数量为value
    
```
// 添加一个商品A
HSET cart:{用户ID} {商品ID} 1

# redis shell
# 添加商品A
localhost:0>HSET cart:101 2010 1
"1"

# 添加商品B
localhost:0>HSET cart:101 2011 1
"1"

// 增加数量
HINCRBY cart:{用户ID} {商品ID}

# redis shell
localhost:0>HINCRBY cart:101 2010 1
"2"

// 删除商品
HDEL cart:{用户ID} {商品ID}

# redis shell
localhost:0>HDEL cart:101 2010
"1"

// 商品总数
HLEN cart:{用户ID}

# redis shell
localhost:0>HLEN cart:101
"1"

// 所有商品
HGETALL cart:{用户ID}

# redis shell
localhost:0>HGETALL cart:101
 1)  "2010"
 2)  "2"
 3)  "2011"
 4)  "1"
```

### 优缺点
- 优点
    + 同类数据归类存储，方便数据管理
    + 相比string操作消耗内存与CPU更小
    + 相比string更节省空间
    
- 缺点
    + 过期功能只能使用在key上，不能作用在field上
    + Redis不适合在集群架构上大规模使用

![p](http://file.uzykj.com/44c33cce-d4a3-f898-8e9e-2223d0ffd765.png)

## list

### 常见用法
```
// 将一个或多个值value插入key列表的表头(最左边)
LPUSH key value [value...]

// 将一个或多个值value插入key列表的表尾(最右边)
RPUSH key value [value...]

// 移除并返回key列表的头元素
LPOP key

// 移除并返回key列表的尾元素
RPOP key

// 返回列表key中的区间元素，偏移量按照start与stop指定
LRANGE key start stop

// 从key列表的表头弹出一个元素，若列表没有元素，则阻塞timeout秒，若timeout=0，则一直阻塞
BLPOP key [key...] timeout

// 从key列表的表尾弹出一个元素，若列表没有元素，则阻塞timeout秒，若timeout=0，则一直阻塞
BRPOP key [key...] timeout
```

### 应用场景
![p](http://file.uzykj.com/a8cddafa-6e44-ab6f-4dda-b60188611817.png)

- 常用数据结构
栈：      Stack = LPUSH + LPOP -> FILO(先进后出)
队列：    Queue = LPUSH + RPOP
阻塞队列： Blocking Queue = LPUSH + BRPOP


---
收录时间: 2021/01/21

<Vssue :title="$title" />
