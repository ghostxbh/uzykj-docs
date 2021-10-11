---
title: sudo权限报错
date: 2021-06-28
sidebar: 'auto'
categories:
  - Linux
tags:
author: z-sm
location: Linux社区
summary: 解决linux下sudo更改文件权限报错 xxx is not in the sudoers file. This incident will be reported.
---

# 解决linux下sudo更改文件权限报错 xxx is not in the sudoers file. This incident will be reported.

Linux中普通用户用 **sudo** 执行命令时报`xxx is not in the sudoers file.This incident will be reported`错误，

解决方法就是在/etc/sudoers文件里给该用户添加权限。如下：

### 1.切换到root用户下
方法为直接在命令行输入：su，然后输入密码（即你的登录密码，且密码默认不可见）。
```shell
[guest@xxx1xx ~]$ su
Password:
```

### 2./etc/sudoers文件默认是只读的，对root来说也是，因此需先添加sudoers文件的写权限,命令是:
即执行操作：
```shell
[root@xxx1xx ~]$ chmod u+w /etc/sudoers
```

### 3.编辑sudoers文件
即执行：
```shell
[root@xxx1xx ~]$ vi /etc/sudoers

#找到这行 root ALL=(ALL) ALL, 在他下面添加：
# (这里的user是你的用户名)
user ALL=(ALL) ALL 
```

可以sudoers添加下面四行中任意一条        
user ALL=(ALL) ALL       
%user ALL=(ALL) ALL      
user ALL=(ALL) NOPASSWD: ALL     
%user ALL=(ALL) NOPASSWD: ALL

解释：  
第一行: 允许用户user执行sudo命令(需要输入密码).        
第二行: 允许用户组user里面的用户执行sudo命令(需要输入密码).      
第三行: 允许用户user执行sudo命令, 并且在执行的时候不输入密码.      
第四行: 允许用户组user里面的用户执行sudo命令, 并且在执行的时候不输入密码.        

### 4.撤销sudoers文件写权限
```shell
[root@xxx1xx ~]$ chmod u-w /etc/sudoers
```

> 原文链接: [Ubuntu报“xxx is not in the sudoers file.This incident will be reported” 错误解决方法](http://www.linuxidc.com/Linux/2016-07/133066.htm)

---
收录时间: 2021-06-28

<Vssue :title="$title" />
