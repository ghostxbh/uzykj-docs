# 重入锁相关

## ReentrantLock

### 简单介绍下 ReentrantLock

![rt](http://file.uzykj.com/rt1.png)

### AQS

AQS实际上封装了对一个双向队列的操作

### TryLock和Lock区别？

TryLock只是CAS设置状态然后去设置独占线程，lock设置独占线程失败可能会Park。

lock是调用try失败后才入AQS的。

### TryLock后还需要Lock吗？

不需要

### 加锁逻辑

1. tryAcquire：尝试CAS获取锁，失败 addWaiter 加入到等待队列
2. addWaiter：入队：
   1. 新建Node，包装本线程
   2. 设置节点关系，CAS设置为队尾（失败呢），头节点是一个虚节点不存储任何数据
3. acquireQueued：对排队的线程进行获取锁操作
   1. 自旋，获取刚才新建的Node的上个节点，如果是头节点就尝试获取锁，如果获取失败，判断是否应当阻塞：
      1. ws（waitStatus）=-1 则阻塞当前线程
      2. ws不为-1 且 > 0 设置前驱为-1
   2. 获取失败

### 解锁过程

解锁不区分公平非公平

1. state-1，如果为0则设置独占锁线程为null
2. 唤醒头节点的下一个节点。**为空则从后向前找**。

### ReentrantLock中阻塞线程如何被唤醒？

由其前一个节点唤醒。

在锁释放成功后，如果当头节点的下一个节点不为空，则直接唤醒这下一个节点; 否则从队尾开始向前找第一个不为空的节点去唤醒。

线程被唤醒后先尝试 tryAcquire 去尝试获取锁，但这里就要分情况讨论了：

	1. 非公平锁： 还是有可能竞争锁失败，失败则放入队尾
	2. 公平锁：一定会被唤醒

### 为什么唤醒时还得从后向前找

处理并发问题。可能有指针断开的时间间隙，所以也许不能遍历到所有节点。

### 如果处于排队等候机制中的线程一直无法获取锁，需要一直等待么？

自旋过程中，如果超时或者异常，节点状态会变为取消，取消节点会从队列中释放。

### ReentrantLock被中断线程为什么会中断两次

线程在等待中被中断，唤醒后还是要抢锁，而中断只是一个记录。

当中断线程被唤醒，并不知道唤醒的原因，所以需要再中断一次。



### 自己写个锁吧

### LOCK如何中断

![lock](http://file.uzykj.com/222196b8c410ff4ffca7131faa19d833.jpg)







## ReentrantReadWriteLock

### RWW 读写锁结构

- RRW由实现两个Lock接口实现：一个readLock一个writeLock组成; 

- Sync继承了AQS，将state这个32位的数据切割为2个16位的部分，高位为读，低位为写。
- 

### RRW 和 ReentrantLock 结构区别

- RRW由两个Lock接口实现了读写锁，自身为ReadWriteLock，而ReentrantLock是自身实现了Lock，**然后调用AQS**去操作那个双端队列的方法。
- status：RRW分高低位，分别代表读锁和写锁的占有量; ReentrantLock只是用来记录重入次数
- RRW 读读可同步，读写，写写都不可以



### ReentrantReadWriteLock 如何处理共享资源原子性

### RRW读状态记录的什么？

### RRW 读写锁获取流程

- 共享锁请求

  1. 根据state低16位，判断没有线程持有写锁
     - 无写锁：是重入或者CAS更新读锁占有数成功就获取到读锁同步代码块
     - 有写锁：
       1. 封装本节点为Shared Node放入AQS队尾，addWaiter
       2. 如果本节点为head后节点，尝试获取锁成功则将当前线程的本节点设置为head（？），然后唤醒后面的线程，自己也进入读锁同步代码块
       3. 否则准备挂起，设置自己的前一个节点为SIGNAL，被唤醒时还会准备去到 2

  

- 独占锁

  1. **读锁被占用**或**占有写锁的非当前线程**，addWaiter
  2. 当前节点在Head节点后，再次尝试占有写锁，失败准备Park

- 释放是会唤醒head后面的下一个SIGNAL节点

- 非公平模式下，写锁必然插队

### RRW用ThreadLocal线程本地对象干什么了？

简单说，是主要是针对读线程，判断当前线程是否有权限 compare n and swap n-1

保存当前线程的重入计数。

因为现在是**并发读**，再用state去保存重入数无法进行判断是否锁释放干净了，因为多个线程的重入数都混在一起了。

所以现在是ThreadLocal计算本线程锁数量，state记录全局。

ReentrantLock没有并发读，所以不用管，直接state搞定。

### RWW 对 ThreadLocal的优化

~~分了两层缓存，减少获取TL的机会。具体逻辑自己查查吧，面试这么难了吗？~~

### RRW的锁降级/锁升级

- 锁降级：

  是指**把持住当前拥有的写锁的同时，再获取到读锁**，**随后释放写锁**的过程。（最后释放读锁）

  主要目的为保证数据可见性，如果不获取读直接释放写，线程2获得了写锁并改变了数据，线程A是无法得到数据的最新变化的。

  RRW是可以实现当前占有写锁的线程，同时去lock读锁的;解锁是释放写锁就转换为读锁了。

- 锁升级：不支持。

  试想多个线程都要升级，会死锁。







### RRW写锁线程饥饿问题

公平模式下没有饥饿问题。

非公平模式下，因为写入线程是要无条件竞争锁的，写多读少情况下，写入线程会因迟迟无法竞争到锁而一直处于等待状态，这也是RRW的问题。

## StampedLock

### stampedLock 介绍，能解决什么问题？

1. 所有获取锁都返回stamp
2. 所有释放锁都需要stamp

- 模式：
  - 读模式
  - 写模式
  - 乐观读模式

StampedLock支持读写互转，主要为了解决RWLock的写锁饥饿问题。

乐观读锁，写锁可以共存，但是可能有数据不一致问题。

所以乐观读必须校验 validate(stamp)。

### StamepedLock基本结构

类似AQS结构，也有state和CLH队列

### StampedLock对比RRW最大问题

StampedLock 无法重入，重入就死锁

## Future模式

### Future和FutureTask

futureTask是future的实现类，主要场景用于异步多线程获取数据，然后阻塞等待全部结果返回。

简单来说Future模式实现是：

1. Future接口内定了get方法;get方法目的是同步等待结果执行完毕。
2. FutureTask实现了Future，其内部：设立一个lock锁 和 一个类变量result作为结果值
   - get方法：拿到lock就去判断任务是否结束，没结束就lock.wait, 结束了就返回result
   - finish方法：接收传参的结果，拿到lock后把接到的传参赋值给result，任务状态结束。
3. TaskService的submit方法接收一个Callable接口（即为task）并将新建的FutureTask返回，开启线程去执行这个Callable的内容，线程结束前调用FutureTask的finish方法。
4. 这样，外部通过submit的返回值可以拿到Future，调用get方法就能等待Callable执行固定的方法完毕再返回结果了。

以上就是最小实现方案，实际JUC的实现和上面的区别在于：

1. state设定了多种任务状态
2. JUC的FutureTask同时实现了Runnable接口，核心的执行方法run在FutureTask内部修改状态，设定返回值。
3. 用AQS实现get中的阻塞操作，因为否则没法实现超时等待。比如第一次get，wait了，第二次又要wait，最后都完事了notifyAll得。没完事就凉了。

---
收录时间: 2021/01/15

<Vssue :title="$title" />