# Redis类型

<img src="http://file.uzykj.com/48040a58-df90-9fa5-232c-4f915d4598d0.png" width=50%>

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

# redis shell
localhost:0>SET test 我是一个value
"OK"
```

- 对象缓存
```shell
# SET object:1 value(json数据)

# redis shell
localhost:0>SET object:1 \"{\"name\": \"Xiaoming\", \"age\": 18}\"
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

<img src="http://file.uzykj.com/828c9f8a-8f0a-588e-22cd-95df1c8ec11f.png" width=50%>

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

<img src="http://file.uzykj.com/279e7f90-e21f-662e-3fc8-1f3cf66ad2eb.png" width=50%>

转换

<img src="http://file.uzykj.com/a11169e9-4cde-32bf-5b9c-189a5499ed7a.png" width=50%>


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
    + 以用户ID为key
    + 商品ID为field
    + 商品数量为value

<img src="http://file.uzykj.com/7868a794-72c1-240e-48e0-78e069a271d5.png" width=50%>
    
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

<img src="http://file.uzykj.com/44c33cce-d4a3-f898-8e9e-2223d0ffd765.png" width=50%>

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

<img src="http://file.uzykj.com/a8cddafa-6e44-ab6f-4dda-b60188611817.png" width=50%>

- 常用数据结构

栈：      Stack = LPUSH + LPOP -> FILO(先进后出)

队列：    Queue = LPUSH + RPOP

阻塞队列： Blocking Queue = LPUSH + BRPOP

- 微博、公众号消息
userA 关注 美团技术、 阿里技术 大V微博/公众号

1、美团技术 发博文，博文ID: 8080
```
LPUSH msg:{userAID} 8080

# redis shell
localhost:0>LPUSH msg:A001 8080
"1"
```

2、阿里技术 发博文，博文ID: 7070
```
LPUSH msg:{userAID} 7070

# redis shell
localhost:0>LPUSH msg:A001 7070
"2"
```

3、获取最新博文
```
LRANGE msg:{userAID} 0 5

# redis shell
localhost:0>LRANGE msg:A001 0 5
 1)  "7072"
 2)  "7071"
 3)  "8082"
 4)  "8081"
 5)  "7070"
 6)  "8080"
```
`LRANGE`拿出来的数据，偏移量0到5的列表

符合先进后出，按最后添加的时间排序。

如果使用MySQL则需要`order by createTime DESC`,降低了查询性能

## set

### 常用操作
```
// 往集合key添加元素，member存在则忽略，不存在则新建
SADD key member [member...]

// 从集合key中删除元素
SREM key member [member...]

// 获取集合key中所有的元素
SMEMBERS key

// 获取集合key中的元素个数
SCARD key

// 判断集合key中是否存在member元素
SISMEMBER key member

// 从集合key中选出count个元素，count个元素不被删除
SRANDMEMBER key [count]

// 从集合key中选出count个元素，count个元素被删除
SPOP key [count]
```

### 运算操作
```
// 交集运算
SINTER key [key...]

// 将交集储存到新集合destination中
SINTERSTORE destination key [key...]

// 并集运算
SUNION key [key...]

// 将并集储存到新集合destination中
SUNIONSTORE destination key [key...]

// 差集运算
SDIFF key [key...]

// 将差集储存到新集合destination中
SDIFFSTORE destination key [key...]
```

### 应用场景
- 微信抽奖小程序

<img src="http://file.uzykj.com/5b294b9a-bd6e-65c6-3160-474b8a520181.png" width=50%>

1、点击参与抽奖的用户加入到集合
```
// 添加参与抽奖的用户ID
SADD key {用户ID...}

# redis shell
localhost:0>SADD lottery 1001 1002 1003
"3"
```

2、获取所有参与抽奖的用户
```
SMEMBERS key

# redis shell
localhost:0>SMEMBERS lottery
 1)  "1001"
 2)  "1002"
 3)  "1003"
```

3、抽取count名幸苦参与者
```
SRANDMENBER key count / SPOP key count

# redis shell
# 保留元素
localhost:0>SRANDMEMBER lottery 1
 1)  "1002"

# 删除元素
localhost:0>SPOP lottery 2
 1)  "1002"
 2)  "1001"
```

- 微信、微博点赞、收藏、标签
1、点赞
```
SADD like:{文章ID} 用户ID

localhost:0>SADD like:A001 1001
"1"
```




---
收录时间: 2021/01/21

<Vssue :title="$title" />
