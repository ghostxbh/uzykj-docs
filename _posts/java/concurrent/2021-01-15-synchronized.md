---
title: Synchronized相关
date: 2021-01-15
sidebar: 'auto'
categories:
  - Java
tags:
  - Synchronized
author: ghostxbh
location: blog
summary: 介绍Java中Synchronized锁，Synchronized锁的功能及其使用。
---
# Synchronized相关

## 一、锁

### 我们是否可以Synchronized(Integer)

不能，Integer具有享元模式，所以看似不同对象，实际都是同一个。

### 既然synchronized是对应monitorenter和exit，那当enter和exit之间的代码发生异常，会解锁吗？

会解锁。实际上可以通过javap -v 得到的字节码文件看出，有两个monitor exit，synchronized隐式的加了exception table，也即try catch。

### java怎么保存的哪些线程持有锁，哪些线程在等待？有没有其他实现方式？

java使用对象头的方式保存的对象锁信息。在偏向锁时对象头会保存线程ID，轻量级锁时对象头保存栈中的LockRecord，重量级锁时对象头保存   指向重量级锁的指针ObjectMonitor。

![header](http://file.uzykj.com/header.png)

实际是要保存一个对应 对象和对象持有者的关系。可以考虑使用同步map，缺点是map可能会很大，性能也不好。

### 上锁后的对象如何保存hashCode？

本身对象头的构造是保存对象的hashCode的，而系统hashcode只能调用一次。

准备加偏向锁时，这一部分会被cas由0替换为偏向的线程ID，所以如果对象计算过hashCode之后就无法进入偏向锁了;偏向锁过程中计算hashCode也会造成锁升级为重量级锁。

轻量级锁在栈中的Lock record指向锁对象，lock record存在一个Displaced Mark Word，就是替换下来的原对象mark word。（重入时为null）

重量级锁的mark word是一个指向堆中monitor对象的指针，这个Object monitor对象中有hash code。

这个hashcode都是原始系统hashCode，并不是用户自定义的。

### 为什么Synchronized是非公平锁

在重量级锁中，线程刚刚进入ContentionList时是会自旋尝试获取锁的，这对已经进入list的线程就不公平。

### synchronized 锁重入是如何记录的？

添加一个displaced mark word 为null的 lock record记录进栈。

重量级锁的monitor记录重入次数。

### 锁能否降级？

可以理解为不行，但实际有可能。当STW阶段，仅能被 VMThread 访问而没有其他 JavaThread 访问的对象。

### synchronized和ReentrantLock区别

1. synchronized是JVM的实现，ReentrantLock是jdk的实现
2. syn是非公平的，rlock可以设置
3. syn无法设置锁条件（比如锁时间），rlock可以
4. ReentrantLock可以灵活tryLock
5. 锁释放不同 隐式/显式
6. 细微获取锁的细节（谁先获取锁的问题，见重量级锁流程）

### 锁消除

锁消除是易于逃逸分析进行的。

如果一个锁对象确定不会逃逸出线程，便可以进行锁消除。



## 二、偏向锁

### epoch是什么？

每个class都会有一个epoch字段，相应的每个对象头也有一个epoch，对象头中的初始值来自于class中的epoch值。每次发生批量重偏向，class中的epoch就+1。

一旦下次获得锁对象发现对象中的epoch值不等于class中的epoch值，就算当前该对象已经偏向其他线程，也就省了次撤销，而是CAS修改threadId。

当然，如果重偏向不成功会升级轻量级。

### 偏向锁加锁/释放流程

- 加锁
  1. 当偏向锁首次进入一个匿名可偏向对象，会在当前线程栈中找到地址**最高**的可用lock record（此处和网上大多文章不同），displaced mark word和mark word会用cas将线程ID存入，同时Lock record的obj指向锁对象。
  2. 如果是锁重入，就添加一个dmw为空的lock record，其中obj还指向对象

- 释放
  1. **释放**时，**从低到高**找一个相关的lock record施放掉

### 偏向锁的撤销（revoke是指获取锁时不满足偏向）

撤销的场景1：锁已经偏向线程A，这时B线程尝试获得锁

当其他线程进入同步块时，发现对象头有偏向的线程了，则会准备撤销偏向锁;

而当前这个竞争的线程在safepoint中会查看那个threadId是不是不存活或者不在同步快中了（遍历栈中是否有相关lock record），存活就升级持有锁的那个线程的锁为轻量级，（？如果不存活则先改mark word为无锁再升级轻量级，）



### 什么情况才会使用偏向锁

默认需全部满足：

1. 看mark word状态，没有获取过hash code且可偏向。
2. 系统开启偏向
3. 系统启动后4s时间，规避jvm启动时的大量竞争，所以不要在系统启动5秒前new对象进行测试。

### 批量重偏向和批量撤销的场景和意义

1. 批量重偏向

   每个class维护了一个偏向锁撤销计数器，当class对象发生偏向操作，该值+1,当到达阀值（20），会触发批量重偏向。epoch值+1,并会遍历jvm所有线程栈，找到所有处于加锁状态的偏向锁，修改epoch为新值。

   **下次获得锁**后，只要发现**epoch和class的epoch不相等**，就不再进入偏向锁撤销，而是直接通过CAS将其threadId改为本threadId。

   都撤销20次了说明这个对象不适合再偏向于原线程，所以撤销20次就要尝试偏向于其他线程。比如线程A新建了某个类的很多对象，然后线程B再去给对象加锁。

2. 批量撤销 

   比如有一个类的达到了批量重偏向阀值（DEF40），说明多线程竞争，会直接标记改class为不可偏向，下次进入该对象同步快直接进入轻量级锁流程。并且，撤销当前正在使用的锁对象的偏向锁。

   在对某个竞争较高的对象处理时，直接就不使用偏向锁了，都已经撤销40次了说明这个对象并不适合使用偏向锁。比如生产消费模型的那个队列对象。

### 对象使用偏向锁一次之后下次还能再偏向吗？

可以;一个对象偏向过之后，有另一个线程再次尝试获得偏向锁并且之前的线程还在占用，才会撤销升级为轻量级锁;否则只要是匿名可偏向对象，就可以修改成功。

同时还有批量重偏向机制。

### 偏向锁升级为轻量级锁过程

1. 当偏向锁 cas 匿名可偏向对象的mark word 0-> threadId失败
2. 当偏向锁发现threadId并非当前线程，则会在safe point时查看原持有对象的线程是否依然存活，存活则准备升级轻量级锁，不存活就走撤销流程。
3. 原偏向线程获得轻量级锁。在safepoint中，直接修改当前偏向的线程栈中最高位lockrecord dmw设置为无锁状态，其他所有相关lock record的dmw都设置为null。
4. **将对象头指向最高位 lock record。**
5. 当前线程？？？

## 三、轻量级锁

### 轻量级锁和其加锁、释放流程

1. 加锁

   （这里和网上不一样，没有自旋;主要为了没有线程争用时不创建monitor对象）

   在执行同步块之前，JVM会在当前线程栈帧中插入一个Lock record，其中就有一个指向锁对象的指针。

   通过CAS指令将LockRecord的地址存进对象头，失败并且非锁重入，说明发生竞争，进入 **锁膨胀** 升级重量级锁。

2. 释放

   遍历线程栈，找到所有LockRecord，并且这个lockRecord的指向是当前的对象

   CAS替换回原始mark word，否则还是要 **锁膨胀**

### 轻量级/无锁状态膨胀为重量级过程

1. 轻量级膨胀
   1. 分配 ObjectMonitor对象并初始化
   2. 设置当前状态为膨胀中
   3. displaced mark word写入objectMonitor对象头、owner为lock record;
   4. 设置对象mark word为1。的monitor对象
2. 无锁膨胀重量级类似，但是没有状态设置，且owner=null

## 四、重量级锁

### 聊聊重量级锁中，对象头的markword指向的monitor对象

![subway2](http://file.uzykj.com/e240de28-f45b-464b-201a-03dc51ca67db.png)

Obj对象加重量级锁时，会创建一个monitor对象，Obj对象的对象头会指向这个monitor对象;

monitor对象主要内部构造是contentionList（cxq）、entryList、waitSet三个链表，还有 owner表示持有锁的线程。

### 重量级锁及其加锁/施放流程

首先，线程**自旋**尝试获取锁。

- 没有得到锁

  1. 会被封装为一个ObjectWaiter 对象放入contentionList队首，对象头中mark word CAS替换为指向这个对象的指针;

  2. park当前线程。

- 得到锁的情况：

  1. owner被设置为这个得到锁的线程
  2. 将锁对象的mark word设置为这个monitor对象，修改锁标志为10
  3. 当前线程wait时会将当前线程放入waitSet;
  4. waitSet中的元素被被notify之后又会加回到entryList

- 释放锁时

  当持有锁的线程释放锁前，正常情况下，entryList为空cxq不为空时，会将cxq的元素按原顺序加入到entryList，并唤醒第一个entryList中的线程并onDeck作为继承人。（后来的线程先获取到锁）



### 为什么要用一个contentionList和一个EntryList实现？

这里详细说一下，所有请求锁的线程都会被放入contentionList（链表list），而锁的候选线程存放进EntryList，而且只有解锁之后发现EntryList为空，才会从contentionList获取线程进入EntryList。

1. 并发情况下，contentionList会被大量并发线程访问，为了降低尾部元素竞争，需要引入entryList。

2. 唤醒waitSet也会进入EntryList而非contentionList，否则也太不公平了

当然三个list/set中的线程都处于被 mutex lock 中。



### onDeck候选的意义

保证只有一个线程正在竞争锁资源



### 偏向锁会直接变成重量级锁吗

会，偏向过程中计算hashCode可能;

调用notify wait也会



### 附录

![lk](http://file.uzykj.com/lock_rt.png)

---
收录时间: 2021/01/15

<Vssue :title="$title" />
