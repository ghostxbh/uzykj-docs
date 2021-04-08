---
title: MySQL文章汇总
date: 2021-03-29
tags:
    - MySQL
    - MVCC
author: ghostxbh
location: blog
summary: MySQL文章汇总
---
# MySQL文章汇总

### [MVCC多版本并发控制](https://www.jianshu.com/p/8845ddca3b23)

MVCC，全称`Multi-Version Concurrency Control`，即多版本并发控制。
MVCC是一种并发控制的方法，一般在数据库管理系统中，实现对数据库的并发访问，在编程语言中实现事务内存

MVCC能解决什么问题，好处是？<br/>
数据库并发场景有三种，分别为：<br/>
- 读-读：不存在任何问题，也不需要并发控制
- 读-写：有线程安全问题，可能会造成事务隔离性问题，可能遇到脏读，幻读，不可重复读
- 写-写：有线程安全问题，可能会存在更新丢失问题，比如第一类更新丢失，第二类更新丢失

![MVCC](http://file.uzykj.com/3133209-be5885051c52fb6a.png)


---
收录时间: 2021-03-29

<Vssue :title="$title" />