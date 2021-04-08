---
title: JVM汇总
date: 2021-03-01
tags:
author: ghostxbh
location: blog
summary: JVM汇总，JVM相关资料，GC参数、Parallel参数、CMS参数、G1参数。
---
# JVM汇总

- 1、[ClassLoader类加载器](2021-01-08-classloader.md)

- 2、[Java内存区域](2021-01-08-memoryarea.md)

- 3、[GC垃圾回收](2021-01-08-gc.md)

- 4、[实战调优](2021-01-08-tuning.md)

- 5、[调优总结](2020-11-18-tuningsum.md)

## GC常用参数

- -Xmn：年轻代  
- -Xms：最小堆 
- -Xmx ：最大堆
- -Xss：栈空间
- -XX:+UseTLAB：使用TLAB，默认打开
- -XX:+PrintTLAB：打印TLAB的使用情况
- -XX:TLABSize：设置TLAB大小
- -XX:+DisableExplictGC：禁用System.gc()不管用 ，防止FGC
- -XX:+PrintGC：打印GC日志
- -XX:+PrintGCDetails：打印GC详细日志信息
- -XX:+PrintHeapAtGC：打印GC前后的详细堆栈信息
- -XX:+PrintGCTimeStamps：打印时间戳
- -XX:+PrintGCApplicationConcurrentTime：打印应用程序时间
- -XX:+PrintGCApplicationStoppedTime：打印暂停时长
- -XX:+PrintReferenceGC：记录回收了多少种不同引用类型的引用
- -verbose:class：类加载详细过程
- -XX:+PrintVMOptions：jvm参数
- -XX:+PrintFlagsFinal：-XX:+PrintFlagsInitial 必须会用
- -Xloggc:opt/log/gc.log：gc日志的路径以及文件名称
- -XX:MaxTenuringThreshold：升代年龄，最大值15

## Parallel常用参数

- -XX:SurvivorRatio：年轻代中eden和from/to的比值。比如设置3就是eden:survivor=3:2，也就是from和to各占1，eden占用3
- -XX:PreTenureSizeThreshold：大对象到底多大
- -XX:MaxTenuringThreshold：升代年龄，最大值15
- -XX:+ParallelGCThreads：并行收集器的线程数，同样适用于CMS，一般设为和CPU核数相同
- -XX:+UseAdaptiveSizePolicy：自动选择各区大小比例

## CMS常用参数

- -XX:+UseConcMarkSweepGC：设置年老代为并发收集
- -XX:ParallelCMSThreads：CMS线程数量
- -XX:CMSInitiatingOccupancyFraction：使用多少比例的老年代后开始CMS收集，默认是68%(近似值)，如果频繁发生SerialOld卡顿，应该调小，（频繁CMS回收）
- -XX:+UseCMSCompactAtFullCollection：在FGC时进行压缩
- -XX:CMSFullGCsBeforeCompaction：多少次FGC之后进行压缩
- -XX:+CMSClassUnloadingEnabled：年老代启用CMS，但默认是不会回收永久代(Perm)的。此处对Perm区启用类回收，防止Perm区内存满。
- -XX:CMSInitiatingPermOccupancyFraction：达到什么比例时进行Perm回收
- -XX:MaxGCPauseMillis：停顿时间，是一个建议时间，GC会尝试用各种手段达到这个时间，比如减小年轻代
- GCTimeRatio：设置GC时间占用程序运行时间的百分比

## G1常用参数

- -XX:+UseG1GC：开启G1
- -XX:MaxGCPauseMillis：建议值，G1会尝试调整Young区的块数来达到这个值
- -XX:GCPauseIntervalMillis：GC的间隔时间
- -XX:+G1HeapRegionSize：分区大小，建议逐渐增大该值，1 2 4 8 16 32。 随着size增加，垃圾的存活时间更长，GC间隔更长，但每次GC的时间也会更长 ZGC做了改进（动态区块大小）
- G1NewSizePercent：新生代最小比例，默认为5%
- G1MaxNewSizePercent：新生代最大比例，默认为60%
- GCTimeRatio：GC时间建议比例，G1会根据这个值调整堆空间
- ConcGCThreads：线程数量
- InitiatingHeapOccupancyPercent：启动G1的堆空间占用比例

## 资料来源
- [Java超神之路-JVM](https://gitee.com/geekerdream/java-legendary/blob/master/%E9%9D%A2%E8%AF%95%E9%A2%98/jvm/%E8%B6%85%E7%A5%9E%E4%B9%8B%E8%B7%AF-JVM.md#)

- [新一代垃圾回收器ZGC的探索与实践](https://tech.meituan.com/2020/08/06/new-zgc-practice-in-meituan.html)

- [Java中9种常见的CMS GC问题分析与解决](https://mp.weixin.qq.com/s/RFwXYdzeRkTG5uaebVoLQw)

- [一文看懂JVM内存布局及GC原理](https://mp.weixin.qq.com/s/9xGsz5TpTSN0LxeOdNV8zA)

---
收录时间: 2021-03-01

<Vssue :title="$title" />