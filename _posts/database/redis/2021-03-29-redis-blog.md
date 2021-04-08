---
title: Redis文章汇总
date: 2021-03-29
tags:
    - Redis
    - BitMap
author: ghostxbh
location: blog
summary: Redis文章汇总
---
# Redis文章汇总

### [Redis中bitmap的妙用](https://segmentfault.com/a/1190000008188655)

Redis bitmap就是通过一个bit位来表示某个元素对应的值或者状态,其中的key就是对应元素本身。
我们知道8个bit可以组成一个Byte，所以bitmap本身会极大的节省储存空间。

使用场景：<br/>
- 用户签到
- 统计活跃用户
- 用户在线状态


---
收录时间: 2021-03-29

<Vssue :title="$title" />