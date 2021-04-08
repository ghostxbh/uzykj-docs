---
title: 线程相关的常用方法
date: 2018-09-11
tags:
    - Thread
author: ghostxbh
location: blog
summary: Java并发中：Thread类、Object类、Lock类、Condition类的常用方法。
---
# 线程相关的常用方法

## Thread类中的方法：

+ `isAlive()`:判断线程是否还活着。活着的概念是指是否消亡了，对于运行态、就绪态、睡眠态的线程都是活着的状态。
+ `currentThread()`:返回值为`Thread`，返回当前线程对象。
+ `getName()`:获取当前线程的线程名称。
+ `setName()`:设置线程名称。给线程命名还可以使用构造方法`Thread(String thread_name)`或`Thread(Runnable r,String thread_name)`。
+ `getPriority()`：获取线程优先级。优先级范围值为1-10(默认值为5)，相邻值之间的差距对cpu调度的影响很小。一般使用3个字段**MIN_PRIORITY**、**NORM_PRIORITY**、**MAX_PRIORITY**分别表示1、5、10三个优先级，这三个优先级可较大地区分cpu的调度。
+ `setPriority()`:设置线程优先级。
+ `run()`：封装的是线程开启后要执行的任务代码。如果`run()`中没有任何代码，则线程不做任何事情。
+ `start()`：开启线程并让线程开始执行`run()`中的任务。
+ `toString()`:返回线程的名称、优先级和线程组。
+ `sleep(long millis)`：让线程睡眠多少毫秒。
+ `join(t1)`:将线程t1合并到当前线程，并等待线程t1执行完毕后才继续执行当前线程。即让t1线程强制插队到当前线程的前面并等待t1完成。
+ `yield()`:将当前正在执行的线程退让出去，以让就绪队列中的其他线程有更大的几率被cpu调度。即强制自己放弃cpu，并将自己放入就绪队列。由于自己也在就绪队列中，所以即使此刻自己放弃了cpu，下一次还是可能会立即被cpu选中调度。但毕竟给了机会给其它就绪态线程，所以其他就绪态线程被选中的几率要更大一些。

## Object类中的方法：

+ `wait()`:线程进入某个线程池中并进入睡眠态。等待`notify()`或`notifyAll()`的唤醒。
+ `notify()`:从某个线程池中随机唤醒一个睡眠态的线程。
+ `notifyAll()`:唤醒某个线程池中所有的睡眠态线程。

这里的某个线程池是由锁对象决定的。持有相同锁对象的线程属于同一个线程池。见后文。

一般来说，`wait()`和唤醒的`notify()`或`notifyAll()`是成对出现的，否则很容易出现**死锁**。

## java.util.concurrent.locks包中的类和它们的方法：

+ Lock类中：

    + `lock()`:获取锁(互斥锁)。
    + `unlock()`:释放锁。
    + `newCondition()`:创建关联此`lock`对象的`Condition`对象。

+ Condition类中：

    + `await()`:和`wait()`一样。
    + `signal()`:和`notify()`一样。
    + `signalAll()`:和`notifyAll()`一样。
    
    
## sleep and wait

### `sleep()`和`wait()`的区别：

+ 所属类不同：`sleep()`在`Thread`类中，`wait()`则是在`Object`中;
+ `sleep()`可以指定睡眠时间，`wait()`虽然也可以指定睡眠时间，但大多数时候都不会去指定;
+ `sleep()`不会抛异常，而`wait()`会抛异常;
+ `sleep()`可以在任何地方使用，而`wait()`必须在同步代码块或同步函数中使用;
+ 最大的区别是`sleep()`睡眠时不会释放锁，不会进入特定的线程池，在睡眠时间结束后自动苏醒并继续往下执行任务，
  而`wait()`睡眠时会释放锁，进入线程池，等待`notify()`或`notifyAll()`的唤醒。



---
收录时间: 2018/09/11

<Vssue :title="$title" />