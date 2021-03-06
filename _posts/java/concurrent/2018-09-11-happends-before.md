---
title: Happends-Before规则
date: 2018-09-11
tags:
author: ghostxbh
location: BeiJing
summary: Happends-Before规则
---
# Happends-Before规则

Java内存模型，如何解决可见性和有序性

Java内存模型规范了JVM如何提供按需禁用缓存和编译优化的方法：volatile、synchronized 和 final 关键字，以及六项 Happends-Before规则。

+ 1、程序次序规则
+ 2、volatile变量规则
+ 3、管程锁定规则
+ 4、线程启动start()规则
+ 5、线程终止join()规则
+ 6、线程中断interrupt()规则
+ 7、对象终结finalize()规则
+ 8、传递性


---
收录时间: 2018/09/11

<Vssue :title="$title" />