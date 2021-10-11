---
title: 面试汇总
date: 2021-02-20
sidebar: 'auto'
categories:
  - Java
tags:
  - 面试
  - Alibaba
author: ghostxbh
location: blog
summary: 面试相关汇总，此篇为面试题汇总，没有答案。
---
# 面试汇总

## 阿里新零售面试题

- 1.写出两个线程死锁的场景
- 2.编写代码将36进制数"ZA123DS"转换为10进制数
- 3.A线程打印"A" B线程打印"B" C线程打印"C" 启动三个线程交替打印ABCABCABC... 

自我介绍，项目业务，框架技术介绍；

- 使用Kafka时业务中如何保证了幂等性？（缓存+数据库）
- 使用redis时，它时如何清理过期数据的？（定期+惰性+数据淘汰机制）
- Mysql-InnoDB的索引数据结构；
- 覆盖索引和回表介绍；
- 线程池ThreadPoolExecutor的参数以及运行机制介绍，最大线程数，拒绝策略；
- 根据个人观点说明Java内存模型的安全性问题；
- volatile关键字，synchronized关键字。
- synchronized锁升级过程。
- 为什么要使用自旋锁？锁升级的好处？

---
收录时间：2021-02-20

<Vssue :title="$title" />
