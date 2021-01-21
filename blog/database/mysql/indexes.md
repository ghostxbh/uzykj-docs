# Mysql索引

## 介绍
**索引**是帮助`MySQL`高效获取数据的排好序的数据结构

### 聚簇索引
也译作聚集索引，它将数据存储与索引放到了一块，找到索引也就找到了数据

### 非聚簇索引
也译作非聚集索引，它将数据存储于索引分开结构，索引结构的叶子节点指向了数据的对应行，`myisam`通过`key_buffer`把索引先缓存到内存中，当需要访问数据时（通过索引访问数据），在内存中直接搜索索引，然后通过索引找到磁盘相应数据，这也就是为什么索引不在`key buffer`命中时，速度慢的原因

## 索引数据结构
- 二叉树
- 红黑树
- Hash表
- B-Tree

> [MySQL索引总结](https://zhuanlan.zhihu.com/p/29118331)

> [深入理解 MySQL 索引](https://www.infoq.cn/article/ojkwyykjoyc2ygb0sj2c)

> [MySQL索引原理及慢查询优化](https://tech.meituan.com/2014/06/30/mysql-index.html)


---
收录时间: 2021/01/19

<Vssue :title="$title" />

