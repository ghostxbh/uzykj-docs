# 主从集群

## 1、Redis如何实现高可用？

采取主从复制，用slave来做从节点，读写分离。主节点挂了的话从节点可以切换为主继续提供工作。哨兵方式的话可以自动故障切换。

## 2、主从复制有哪几种常见的方式？

- 同步阻塞

> 优点：数据强一致性。（但是会破坏可用性，也就是CAP的A）
>
> 缺点：效率低，同步阻塞。
>
> ![5](http://file.uzykj.com/redis_5.png)

- 异步非阻塞（Redis默认采取的此种方式，效率高）

> 优点：效率高，异步非阻塞。
>
> 缺点：会丢失数据，满足了CAP的A，舍弃了CAP的C。
>
> ![6](http://file.uzykj.com/redis_6.png)

- 同步阻塞MQ（大数据hive采取的就是这种方式，他会保证最终一致性。）

> 优点：效率相对较高、能保证数据最终一致性。
>
> 缺点：没发现啥缺点。非要说缺点那就是有可能取到不一致的数据，因为不是强一致性。为什么Redis不采取这个？因为Redis要高效率，不想融入太多组件（MQ）进来。
>
> ![7](http://file.uzykj.com/redis_7.png)

## 3、主从复制的完整过程

- slave启动，这时候仅仅保存了master的信息，比如master的host和ip，复制流程还未开始。
- slave节点内部有个定时任务，每秒检查是否有新的master节点要连接和复制，若有，则跟master节点简历socket网络链接。
- slave发送ping命令给master。
- 进行权限认证，若master设置了密码（requirepass），那么slave节点必须发送masteraut的命令过去进行认证。
- master节点第一次执行全量复制，将所有数据发送给slave节点。
- master后续持续写命令，异步复制给slave。

## 4、主从复制核心原理

- 当启动一个slave节点的时候他会发送 PSYNC命令给master节点
- 如果slave是第一次连接到master，那么会触发一次`full resynchronization`进行全量复制。开始`full resynchronization`的时候，master会启动一个后台线程负责生成rdb文件，与此同时还会将客户端收到的所有写命令缓存到内存中。rdb文件生成完成后，master会将这个rdb发送给slave，slave会先把master传来的rdb写入磁盘然后清空自己的旧数据然后 从本地磁盘load到内存中进行同步数据，然后master会将内存中缓存的写命令发送给slave，slave也会同步这些数据。

- 若slave不是第一次连接到master，而是因为某种原因（比如网络故障）断开了进行重新链接的，那么master仅仅会复制给slave断开这段时间缺少的那部分数据，不会全量复制。这个称为断点续传。

### 4.1、追问：怎么判断Slave和Master是不是第一次链接的？

master节点会在内存中创建一个backlog，然后master和slave都会保存一个replica offset和一个master  run id，offset就是保存在backlog中的。 若master和slave的网络链接断开了，那么slave会让master从上次的replica offset开始继续复制。**若没找到对应的offset，则进行一次全量复制。**

### 4.2、追问：什么是主从复制的断点续传？

redis2.8开始支持的主从复制断点续传，若主从复制过程中出现了网络故障导致网络链接断开了，那么slave重新连接到master的时候可以接着上次复制的地方继续复制下去，而不是从头开始复制。

### 4.3、追问：怎么实现的断点续传？

master节点会在内存中创建一个backlog，然后master和slave都会保存一个replica offset和一个master id，offset就是保存在backlog中的。 若master和slave的网络链接断开了，那么slave会让master从上次的replica offset开始继续复制。若没找到对应的offset，则进行一次全量复制。

### 4.4、追问：什么是backlog？

backlog是一个环形缓冲区，整个master进程中只会存在一个，所有的slave公用，默认大小是1MB，在master给slave复制数据的时候，也会将增量数据在backlog中同步写一份，backlog主要用于做全量复制中断的时候的增量复制的。

### 4.5、追问：replica offset是干嘛的？

master和slave都会维护一个offset，master和slave都会不断累加offset，然后slave每秒都上报自己的offset给master，同时master也会保存每个slave的offset，然后slave上报offset给master的时候，master发现offset不一致就能发现数据不一致的情况了。 

## 5、主从复制无磁盘化复制

master在内存中直接创建rdb文件然后发送给slave，不会在本地落盘。此种方式不安全，因为没了持久化操作，但效率高，因为减少了一次磁盘IO操作，直接网络传输给slave。配置参数如下：

```properties
# 默认是no，先落盘，再进行网络传输。
repl-diskless-sync no
# 等待一定时长再开始复制，因为要等更多slave重新连接过来
repl-diskless-sync-delay
```

## 6、主从复制过期key处理

首先slave是不会过期key的，只会等master过期key，若master过期了一个key或者lru淘汰了一个key，那么master会模拟一条del命令发送给slave。

## 7、聊聊sentinel哨兵？

有哨兵之前都是手动进行故障转移。sentinel可以将之前纯人工进行故障操作的步骤自动化。哨兵主要包含以下功能：

- 集群监控，负责监控redis的master和slave进程是否正常工作
- 消息通知，若某个redis实例有故障，那么哨兵负责发送消息作为报警通知给管理员
- 故障转移，若master节点挂了，会自动转移到slave节点 上
- 配置中心，若故障转移发生了，会通知客户端新的master地址

哨兵也需要单独部署，但是他不负责读写请求，只是个看门狗，负责监控Redis集群和自动故障转移。

## 8、哨兵之间是怎么发现彼此的 ？

是通过PSUBSCRIBE这个Redis内置的发布订阅命令来实现的，他们共同监听`__sentinel__:hello`这个channel，每隔两秒钟，每个哨兵都会往自己监控的某个master+slaves对应的`__sentinel__:hello channel`里发送一个消息，内容是自己的host、ip和runid还有对这个master的监控配置，每个哨兵也会去监听自己监控的每个master+slaves对应的`__sentinel__:hello channel`，然后去感知到同样在监听这个master+slaves的其他哨兵的存在，每个哨兵还会跟其他哨兵交换对master的监控配置，互相进行监控配置的同步。

## 9、什么是脑裂？如何解决？

脑裂：某个master所在机器突然脱离了正常的网络，跟其他slave机器不能连接，但是实际上master还运行着，此时哨兵可能就会认为master宕机了，然后开启选举，将其他slave切换成了master，这个时候，集群里就会有两个master，也就是所谓的脑裂。

**解决方案**

```properties
# 表示连接到master的最少slave数量
min-replicas-to-write 3
# 表示slave连接到master的最大延迟时间
min-replicas-max-lag 10
```

按照上面的配置，要求至少3个slave节点，且数据复制和同步的延迟不能超过10秒，否则的话master就会拒绝写请求，配置了这两个参数之后，如果发生集群脑裂，原先的master节点接收到客户端的写入请求会拒绝，就可以减少数据同步之后的数据丢失（最多丢失10s）。

## 10、主从会造成数据不一致吗？如何解决？

会造成，如下两个可能：

- 因为master -> slave的复制是异步的，所以可能有部分数据还没复制到slave，master就宕机了，此时这些部分数据就丢失了。
- 脑裂的情况也会造成数据不一致。此时虽然某个slave被切换成了master，但是可能client还没来得及切换到新的master，还继续写向旧master的数据可能也丢失了，因此旧master再次恢复的时候，会被作为一个slave挂到新的master上去，自己的数据会清空，重新从新的master复制数据

**解决方案**

```properties
# 表示连接到master的最少slave数量
min-replicas-to-write 3
# 表示slave连接到master的最大延迟时间
min-replicas-max-lag 10
```

按照上面的配置，要求至少3个slave节点，且数据复制和同步的延迟不能超过10秒，否则的话master就会拒绝写请求，配置了这两个参数之后，如果发生集群脑裂，原先的master节点接收到客户端的写入请求会拒绝，就可以减少数据同步之后的数据丢失（最多丢失10s）。**是的，没有根治，只是减少了丢失量。若要当作db来用，需要零丢失的话，可以引入mq当中间件。**

## 11、什么是主观宕机（sdown）？什么是客观宕机（odown）？

- sdown是主观宕机，就一个哨兵如果自己觉得一个master宕机了，那么就是主观宕机

- odown是客观宕机，如果quorum数量的哨兵都觉得一个master宕机了，那么就是客观宕机

## 12、sentinel是怎么选举新Master的？

基于raft协议，quorum数一般为`(n/2)+1`，具体流程如下

- sentinel发现master下线，修改其状态为sdown。

- sentinel和其他sentinel确认master是否down掉，超过quorum指定的哨兵进程都认为sdown之后，就变为odown。

- 对我们的当前epoch进行自增， 并尝试在这个epoch中当选，也就是说首先发现master down掉的sentinel有优先权当选为leader。

- 如果当选失败，那么在设定的故障迁移超时时间的两倍之后，重新尝试当选。如果当选成功，那么执行以下步骤。

- 选出一个从服务器，并将它升级为主服务器。

- leader选出一个slave作为master，发送slaveof no one命令（slaveof no one 顾名思义了，就是我不是任何人的slave）。

- 通过发布与订阅功能，将更新后的配置传播给所有其他Sentinel，其他Sentinel对它们自己的配置进行更新。

- 并通过给其他slave发送slaveof master命令告知其他slave新的master。

- 当所有从服务器都已经开始复制新的主服务器时，领头Sentinel终止这次故障迁移操作。

### 12.1、追问：为什么quorum数一般为`(n/2)+1`？

我是这么理解的哈：

- 全部哨兵同意才能升级的话，也就是比如3个Redis、3个Redis哨兵必须保证3个哨兵都同意A节点升级为Master才行的话，那很可能第一台哨兵由于自身网络问题导致连接失败，而其他两个都是正常的，那就不升级了？不公平也不合适，因为那个哨兵明明是自身网络问题。
- 一个哨兵同意就升级的话，也就是比如3个Redis、3个Redis哨兵只要保证又1个哨兵都同意A节点升级为Master就可以的话，那也不靠谱，很可能那节点已经挂了，但是哨兵A可能缓存或者其他问题返回正常，但是其他两个哨兵明显发现问题连接不上了等，少数服从多数是最保险最靠谱的。**最主要的是一个哨兵同意就可以的话会很容易产生脑裂问题。**

## 13、master选举的时候会选择哪个slave？

- 跟master断开连接的时长，如果一个slave跟master断开连接已经超过了`down-after-milliseconds`配置的10倍，那么slave就被认为不适合选举为master
- 按照slave优先级进行排序，slave priority越低，优先级就越高

- 如果slave priority相同，那么看replica offset，哪个slave复制了越多的数据，offset越靠后，优先级就越高

- 如果上面两个条件都相同，那么选择一个run id比较小的那个slave

## 14、Redis集群的数据是怎么分布的？

Redis集群并没有采取hash取模和一致性hash算法，而是采取的槽slot的方式进行分布数据的。Redis一共分为16384个slot，接收到命令后具体流程如下：

- Redis会将这16384个slot平均分配到当前Redis的master实例上，比如一共四个Master，那么每个Master的slot数就是16384/4=4096个。当然slave节点是同步的主节点，也是4096个。
- 接收到请求后，先将key按照CRC16算法得出一个值再 % 16384，这样就找到了这个key是在哪个hash槽上的，也就定位到了redis实例。
- 但Redis并没办法直接去那个Redis实例去执行命令，他会看如果对应的master就是自己当前链接的的话，那么就直接处理掉了。
- 若不是当前自己本地的话，也就是计算出来是hash槽在其他master上，那么就会给客户端返回一个moved error错误，告诉你，你需要到xxx实例上去执行这条命令，然后客户端就精确定位到哪个master的信息去执行了 。

### 14.1、什么是slot？

Redis集群有16384个哈希槽，每个key通过CRC16校验后对16384取模来决定放置哪个槽，集群的每个节点负责一部分hash槽。

### 14.2、Redis集群最大节点个数是多少？

16384个。

## 15、Redis集群是读写分离吗？

cluster默认是不支持slave节点读或者写的，跟我们手动基于哨兵搭建的主从架构不一样的，需要手动带上readonly这个指令，这个时候才能在slave node进行读取。默认的话就是读和写都到master上去执行的，redis cluster模式下就不建议做物理的读写分离了，建议通过master的水平扩容，来横向扩展读写吞吐量，还有支撑更多的海量数据。

## 16、Redis集群如何做高可用的？

Redis集群的高可用并不像哨兵那样需要单独部署哨兵程序，默认也不会像主从复制那样读写分离，Redis集群的slave节点仅仅做数据备份以及高可用，比如现在有5个master，每个master都有1个slave，然后新增了3个slave作为冗余，那么现在有的master就有2个slave了，也就是有的master出现了salve冗余。如果某个master的slave挂了，那么redis cluster会自动迁移一个冗余的slave给那个master，所以集群部署的时候最好多冗余出几个slave做高可用。

## 17、Redis集群如何扩一个节点？如何摘一个节点？

**扩容：**

- 启动新Redis实例当作master节点加入老集群
- reshard一些数据过去，也就是把一部分hash slot从一些node上迁移到这个新节点中，让集群16384个slot重新平均分配
- 启动新Redis实例当作slave节点加入老集群（为了高可用）

**缩容：**

- 先用reshard将数据都移除到其他节点
- 确保清空了一个master的hash slot时，redis cluster就会自动将其slave挂载到其他master上去
- 这个时候就只要删除掉master就可以了。

## 18、Redis集群Slave是如何进行升级为Master的？

和哨兵的类似的流程。

- 判断节点宕机，如果一个节点认为另外一个节点宕机，那么就是pfail（主观宕机），如果多个节点都认为另外一个节点宕机了，那么就是fail（客观宕机），跟哨兵的原理几乎一样，sdown，odown，在cluster-node-timeout内，某个节点一直没有返回pong，那么就被认为pfail，如果一个节点认为某个节点pfail了，那么会在gossip ping消息中，ping给其他节点，如果超过半数的节点都认为pfail了，那么就会变成fail。
- 从节点过滤，对宕机的master node，从其所有的slave node中，选择一个切换成master node，检查每个slave node与master node断开连接的时间，如果超过了`cluster-node-timeout * cluster-slave-validity-factor`，那么就没有资格切换成master，这个也是跟哨兵是一样的，从节点超时过滤的步骤。
- 从节点选举，每个从节点，都根据自己对master复制数据的offset，来设置一个选举时间，offset越大（复制数据越多）的从节点，选举时间越靠前，优先进行选举，所有的master node开始slave选举投票，给要进行选举的slave进行投票，如果大部分`master node（N/2 + 1）`都投票给了某个从节点，那么选举通过，那个从节点可以切换成master，从节点执行主备切换，从节点切换为主节点。

## 19、Redis集群搭建方案有哪几种？

- 官方
- 推特的twemproxy
- codis

---
收录时间: 2021/01/11

<Vssue :title="$title" />