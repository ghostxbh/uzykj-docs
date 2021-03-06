---
title: Hibernate自增策略异常
date: 2018-12-11
tags:
    - Hibernate
author: ghostxbh
location: BeiJing
summary: Hibernate自增策略异常，及解决方案
---
# Hibernate自增策略异常

### 异常信息
```
Apparently hibernate looks for sequence tables for generating the id. Setting the following:

@GeneratedValue(strategy = GenerationType.IDENTITY)
1
on the id, causes it to use the underlying db’s auto increment and not try to generate the id itself, and now it works.

```

### 解决方案

在主键上面添加自增策略

```java
@GeneratedValue(strategy = GenerationType.IDENTITY)
```

即可完美解决

---
收录时间: 2018-12-11

<Vssue :title="$title" />
