---
title: 并发概念
date: 2018-09-11
sidebar: 'auto'
categories:
  - Java
tags:
  - Process
  - Thread
author: ghostxbh
location: blog
summary: Java并发中所应用的相关概念
---
# 并发相关概念

## 进程(`Process`)与线程(`Thread`)

### 进程 - `Process`

+ 1.进程是系统资源分配的最小单元。线程是CPU调度的最小单元。
+ 2.一个进程至少包含一个线程，可以包含多个线程。这些线程共享这个进程的资源。
+ 3.每个线程都拥有独立的运行栈和程序计数器，线程切换开销小。
+ 4.多进程指的是操作系统同时运行多个程序，如当前操作系统中同时运行着QQ、IE、微信等程序。
+ 5.多线程指的是同一进程中同时运行多个线程，如迅雷运行时，可以开启多个线程，同时进行多个文件的下载。


---
收录时间: 2018/09/11

<Vssue :title="$title" />
