---
title: MySQL面试
date: 2021-01-19
sidebar: 'auto'
categories:
  - Database
tags:
  - MySQL
  - 面试
author: ghostxbh
location: blog
summary: MySQL面试收集。
---
# 面试收集

## 1、为什么表数据删掉一半，表文件大小不变？

背景：100GB的日志表，三个月前的数据都迁移走了，可以直接delete掉，然后delete后发现表大小还是100GB。

原因：删除了那么多数据，其实只是代表那些数据所在的数据页变成可复用的，实际磁盘空间并没有减少。所以只是标记了下数据页变为可用。

解决：重建表，命令：`alter table A engine=InnoDB;`

## 2、为什么我只查一行的语句，也执行这么慢？

背景：`select * from t where id=1;`就这SQL很慢，为啥？

分析：

- 大概率是表被锁住了，用`show processlist;`来分析下是不是存在`Waiting for table metadata lock`的。
- 如果不是锁表的话，那么看下是不是在等flush，在表里执行`select * from information_schema.processlist where id=1;`，看看这个线程的状态是 不是`Waiting for table flush`，这个状态表示的是，现在有一个线程正要对表 t 做 flush 操作。MySQL 里面对表做 flush 操作的用法，一般有以下两个：
  `flush tables t with read lock;`
  `flush tables with read lock;`
- 检查下是不是存在行锁：`select * from t sys.innodb_lock_waits where locked_table='`test`.`t`'\G`

## 3、怎么最快地复制一张表？

- mysqldump 方法
- 导出csv，然后load data导入新表

> ```mysql
> select * from db1.t where a>900 into outfile '/server_tmp/t.csv';
> load data infile '/server_tmp/t.csv' into table db2.t;
> ```

## 4、MySQL数据库cpu飙升到500%的话他怎么处理？

-  top命令找到占用cpu最高的进程id。
- 如果是 mysqld 造成的，`show processlist;`看看是不是有消耗资源的 sql 在运行。
- 如果有的话先kill掉这些线程，保证线上稳定运行，然后再分析SQL如何优化。

## 5、如果某个表有近千万数据，CRUD比较慢，如何优化？

**分库分表**

某个表有近千万数据，可以考虑优化表结构，分表（水平分表，垂直分表），当然，你这样回答，需要准备好面试官问你的分库分表相关问题呀，如

- 分表方案（水平分表，垂直分表，切分规则hash等）
- 分库分表中间件（Mycat，sharding-jdbc等）
- 分库分表一些问题（事务问题、分布式id、跨节点Join等问题）
- 解决方案（分布式事务等）

**索引优化**

除了分库分表，优化表结构，当然还有所以索引优化等方案。

## 6、数据库自增主键可能遇到什么问题？

- 使用自增主键对数据库做分库分表，可能出现诸如主键重复等的问题
- 自增主键会产生表锁，从而引发问题
- 自增主键可能用完问题

## 7、百万级别或以上的数据，你是如何删除的？

- 我们想要删除百万数据的时候可以先删除索引
- 然后批量删除其中无用数据
- 删除完成后重新创建索引。

> `alter table T engine=InnoDB;`

## 8、关心过业务系统里面的sql耗时吗？统计过慢查询吗？对慢查询都怎么优化过？

- 我们平时写Sql时，都要养成用explain分析的习惯。
- 慢查询的统计，运维会定期统计给我们
- 我们业务系统也有监听器统计耗时的SQL到日志文件

**优化慢查询：**

- 先确定是不是网络、cpu、io、锁、大事务的问题，如果不是的话在分析SQL语句索引方面。

- 分析语句，是否加载了不必要的字段/数据。
- 分析SQl执行句话，是否命中索引等。
- 如果SQL很复杂，优化SQL结构
- 如果表数据量太大，考虑分表

## 9、你进行SQL优化的时候一般步骤是什么？

- 避免返回不必要的数据
- 加索引，explain进行分析
- 适当分批量进行
- 分库分表
- 读写分离

## 10、深分页很慢，怎么解决的呢？

利用子查询优化超多分页场景。先快速定位需要获取的id段，然后再关联。这样可以防止回表。

比如：

```mysql
-- 原来SQL
SELECT * from employee WHERE 条件 LIMIT 1000000,10 
-- 优化成如下
SELECT a.* FROM employee a, (SELECT id FROM employee WHERE 条件 LIMIT 1000000,10 ) b WHERE a.id=b.
```

## 11、主键自增ID还是UUID之间选择哪个？为什么？

自增ID，因为主键是聚簇索引，也就是一颗B+树，索引字段在页里是从小到大排序的，所以自增可以避免页分裂，uuid的话会频繁页分裂。因为每次插入uuid都可能比之前的小，就需要插入到前面去，就造成了页分裂，多个数据页之间的数据来回挪动。

## 12、如何排查项目中可优化的SQL？

- 1.把慢SQL的时间设置为0，慢SQL=0也就是记录了全部SQL到slowlog里。**调整慢SQL的时间参数不需要重启MySQL服务。**
- 2.然后用**工具（pt-query=digest）**分析哪些SQL最慢，哪些SQL执行次数最多等指标，项目上线前2天可以这么配置，分析完后再把慢日志调回原来数值，否则一直写slowlog也会对mysql造成性能压力（10%以内）。

- 3.然后调优那些单次执行时间最长的（几乎都是索引有问题，所以调优）、总访问次数最多的SQL（因为这种SQL肯定是系统核心功能的SQL，做到极致，比如发现某种配置类的SQL，内容长期不更新的，就可以放到缓存里等）。

---
收录时间: 2021-01-19

<Vssue :title="$title" />
