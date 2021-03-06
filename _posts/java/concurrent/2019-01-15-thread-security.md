---
title: 多线程安全问题
date: 2019-01-15
tags:
    - Thread
author: ghostxbh
location: BeiJing
summary: 线程安全问题是指多线程同时执行时，对同一资源的并发操作会导致资源数据的混乱。
---
# 多线程安全问题

线程安全问题是指多线程同时执行时，对同一资源的并发操作会导致资源数据的混乱。

### 1.1 示例代码
```java
public class RunnableTest implements Runnable {
    private int num;

    public RunnableTest(int num){
        this.num = num;
    }
    
    private void sale() {
        if (num > 0) {
            num--;
            System.out.println(Thread.currentThread().getName() + "----" + getRemain());
        }
    }

    private int getRemain(){
        return num;
    }

    @Override
    public void run() {
        while (true){
            sale();
        }
    }

    public static void main(String[] args) {
        RunnableTest rt = new RunnableTest(10);
        Thread thread1 = new Thread(rt);
        Thread thread2 = new Thread(rt);
        Thread thread3 = new Thread(rt);

        thread1.start();
        thread2.start();
        thread3.start();
    }
}
```

### 1.2 输出
```
Thread-0----8
Thread-0----6
Thread-1----8
Thread-0----5
Thread-0----4
Thread-0----3
Thread-0----2
Thread-0----1
Thread-0----0
Thread-2----7
```
### 1.3 执行过程

此处代码执行过程如下图所示

![线程安全](http://file.uzykj.com/thread_security.png)

共开启了3个线程执行任务(不考虑main主线程)，每一个线程都有3个任务：

+ ①判断if条件`if(num>0)`;
+ ②数量自减`num--`;
+ ③获取剩余数量`return num`;
+ ④打印返回的`num`数量`System.out.println(Thread.currentThread().getName()+"-------"+remain())`。
这三个任务的共同点也是关键点在于它们都操作同一个资源`RunnableTest`对象中的`num`，这是多线程出现安全问题的本质，也是分析多线程执行过程的切入点。

当main线程开启t1-t3这3个线程时，它们首先进入就绪队列等待被CPU随机选中。
+ (1).假如t1被先选中，分配的时间片执行到任务②就结束了，于是t1进入就绪队列等待被CPU随机选中，此时票数`num`自减后为9；
+ (2).当t3被CPU选中时，t3所读取到的`num`也为9，假如t3分配到的时间片在执行到任务②也结束了，此时票数`num`自减后为8；
+ (3).同理t2被选中执行到任务②结束后，`num`为7；
+ (4).此时t3又被选中了，于是可以执行任务③，甚至是任务④，假设执行完任务④时间片才结束，于是t3的打印语句打印出来的`num`结果为7；
+ (5).t1又被选中了，于是任务④打印出来的`num`也为7。

显然，上面的代码有几个问题：
+ (1)有些数量没有自减了但是其他线程并没有自减；
+ (2)有的数量重复了。这就是线程安全问题。


---
收录时间: 2018/09/13

<Vssue :title="$title" />