---
title: MySQL事务
date: 2021-01-19
tags:
    - MySQL
    - 事务
author: 编程界的小学生
location: Gitee
---
# 事务

## 1、事务的四大特性

​		**ACID**

- 原子性

> A Atomicity：要么都成功，要么都失败。

- 一致性

> C Consistent：事务开始之前和完成之后的数据都必须保持一致的状态，必须保证数据库的完整性。

- 隔离性

> I Isolation：数据库允许多个并发事务同事对数据进行操作，隔离性保证各个事务相互独立，事务处理时的中间状态对其它事务是不可见的，以此防止出现数据不一致状态。可通过事务隔离级别设置：包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。

- 持久性

> D Durable：一个事务处理结束后，其对数据库的修改就是永久性的，即使宕机了，数据也不会丢失。redolog+binlog

## 2、事务四种隔离级别

- 读未提交

> READ-UNCOMMITTED：一个事务没提交但是他修改的数据却可以被其他事务看到，可能会导致脏读、幻读、不可重复读。 

- 读已提交

> READ-COMMITTED：一个事务没提交，那么他修改的数据其他事务看不到，只能读到已经提交的数据。可能导致幻读和不可重复读。

- 可重复读（innodb默认）

>REPEATABLE-READ：MySQL INNODB默认的隔离级别。一个事务内读到的数据永远是一样的，不管其他事务做了什么更改，是否提交，我都不知道，我只活在我自己的事务里。可以阻止脏读和不可重复读，但是幻读仍有可能发生，比如insert唯一key。

- 串行化

> SERIALIZABLE：让所有事务串行化执行，就好比单线程了，一个一个的排队执行，效率低下，可以避免脏读、幻读、不可重复读，因为单线程了嘛。

## 3、什么是脏写、脏读、不可重复读、幻读

- 脏写

> A和B两个事务都更新同一条数据， 数据原始值为1，A将数据改为2，B将数据改为3，这时候A回滚了，把B写的3也给抹掉了。这就是脏写。
>
> 就是两个事务都更新一个数据，结果其中一个事务回滚了，这波操作把另一个事务刚写入的值也给抹掉了。

- 脏读

> 一个事务中访问到了另外一个事务未提交的数据，读未提交隔离级别会造成脏读。

- 不可重复读

> 同一个事务里，读取同一条记录多次可能读到不一致的结果。比如A和B两个事务，A负责读取两次，B负责修改数据，A第一次读取到1，B给修改成2且提交了。A在读取的时候发现变成了2，同一个事物里读到的结果不一致了。可重复读策略可以避免此类情况。

- 幻读

> 比如范围查询，同一个事物里执行两次 `where id > 10;`第一次结果是100条记录，但是其他事务在执行第二遍的时候insert了一条，那么回到原事务里再次执行的时候出现了101条记录。
>
> 再比如 一个事务insert了一条记录，有个key是唯一的，在提交之前另一个事务也insert了同样的唯一key且提交了，那么刚才那个事务在commit的时候就唯一key冲突了。

## 4、什么是mvcc？原理是什么？

Multi-Version Concurrency Control，多版本并发控制。用于实现提交读和可重复读这两种隔离级别。主要原理是undo log版本链+read view。

### 4.1、什么是undo log版本链

每条数据都有两个隐藏的字段：trx_id和roll_pointer，trx_id是最近一次更新这一条数据的事务 id，roll_pointer指向了这条数据之前改动所生成的undo log。

比如：事务A（事务id=50）插入了一条数据，然后事务B（事务id=58）修改了一下这条数据，将值改为了B，那么此时事务B的roll_pointer会指向事务A的undolog，事务A的roll_pointer会指向null（因为他是insert的，之前这条数据没undolog），如下图：

![undolog版本链.png](http://file.uzykj.com/undolog%E7%89%88%E6%9C%AC%E9%93%BE.png)

接着事务C（事务id=69）又把这条数据的值改为C，此时如下图：

![undolog版本链2.png](http://file.uzykj.com/undolog%E7%89%88%E6%9C%AC%E9%93%BE2.png)

这种链式就叫redolog链，所以每条数据的redolog都有一条版本链。
总结一句话：redo log版本链就是针对每个事务对同条数据的修改都记录下来形成一个链式结构，链由roll_pointer这个隐藏字段来连接。

### 4.2、什么是read view

你执行一个事务的时候就会给你生成一个ReadView，ReadView可以理解成快照，里面包含四个关键信息：

- m_ids：**此时此刻**有哪些事务还是未提交的（也就是说和我并发了，我开启事务的时候，还有哪些事务和我一起并发且未提交的）。
- min_trx_id：m_ids里的最小值。
- max_trx_id：MySQL**下一个**要生成的事务id，比如m_ids里有[3,12]这两个id，那max_trx_id就是13。
- creator_trx_id：本事务id。

原理如下：

去undolog版本链上找，看看trx_id是不是自己或者是不是小于min_trx_id，如果是，则代表这条数据是历史就被修改过的，不是与此次并发的，就可以满足条件，如果大于等于max_trx_id，则代表我此次事务还没执行完，其他事务先我一步把数据更新了，我是快照，我当然不能看到。

举个例子：

假设很早之前事务id=32插入了一条数据且提交了事务。然后现在事务A（trx_id=45）和事务B（trx_id=59），事务A想要读取这行数据，事务B想要更新这行数据，那么事务A和事务B会分别开一个ReadView出来，那么这个事务A的ReadView里的m_ids是[45, 59]，min_trx_id=45，max_trx_id=60，creator_trx_id=45，假设事务B先更新了这行数据然后事务A开始读取，然后事务A查询的时候会发现trx_id大于ReadView的min_trx_id（也就是45），同时小于max_trx_id，说明这行undo log版本链上的数据是被一个跟我并发执行的事务修改的，于是看下trx_id=59是不是在m_ids列表里，结果发现是在的，就确定这行数据是并发执行的，不能读取的，顺着roll_pointer往下继续找undolog日志链，继续走上面的流程，发现trx_id=32这条数据吻合，拿出值。如下图：

![readview.png](http://file.uzykj.com/readview.png)

如果再来个事务C（trx_id=100），那么事务A查询的时候会看事务C的 trx_id是不是满足条件，发现100比max_trx_id都大了，肯定是我的事务都没执行完，其他事务却先一步update了，这数据肯定不能要，就顺着undolog版本链继续往下找。



所以ReadView结合undo log版本链才会有奇效，所以undo log版本链配合read view是mvcc的核心原理。

## 5、RC（Read Committed，读提交）如何实现的

主要原理是基于undolog版本链和ReadView来实现。RC是可以造成不可重复读的问题的，所以比如事务A查询两次，事务B在事务A两次查询期间update了一条且commit了，那么事务A第一次查询的值和第二次查询的值是不同的。但是ReadView不是快照吗？不是一个事务一个ReadView吗 ？怎么还能读到新值？是因为**RC隔离级别不是针对每个事务开启一个ReadView，而是每个数据库操作都开启一个新的ReadView**，所以事务A的两次查询对应两个ReadView，第二个查询的时候对应的ReadView里m_ids已经不包含事务B了，因为他已经提交完了，事务A就认为你是历史数据可以正常读取。

## 6、RR（Repeatable Read，可重复读）如何实现的

主要原理是基于undolog版本链和ReadView来实现。每个事务开启一个ReadView快照，所以不管你外界怎么提交，我只管我此次事务的快照里的内容，不受外界影响，所以避免了脏读 、不可重复读的问题。

## 7、RR能解决幻读吗？

不能完全解决，比如insert同一个唯一key的话还会唯一key冲突，这就是不能解决的体现，但是可以解决同一个事物里读出数据数量不一致的问题。

---
收录时间: 2021-01-19

<Vssue :title="$title" />