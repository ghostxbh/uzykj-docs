---
title: 华为鲲鹏上安装rtpengine
date: 2020-11-23
tags:
    - CentOS
    - rtpengine
    - 鲲鹏
author: ghostxbh
location: blog
summary: 华为上安装rtpengine，遇到的问题主要有2点：安装rtpengine很多的依赖不支持arm，需要源代码编译；rtpengine使用内核态运行不了，改为用户态运行
---
# 华为鲲鹏上安装rtpengine
## 前言
华为上安装rtpengine，遇到的问题主要有2点

安装rtpengine很多的依赖不支持arm，需要源代码编译
rtpengine使用内核态运行不了，改为用户态运行

### 安装依赖
```shell script
   yum install pkg-config
   yum install nasm
   yum install libgnomeui-devel
   yum install openssl-devel
   yum install libevent2-devel
   yum install pcre-devel
   yum install xmlrpc-c-devel
   yum install iptables-devel
   yum install epel-release.noarch
   yum install json-glib-devel
   yum install libpcap-devel
   yum install flex
   yum install bison
```
   

### 安装ffmpeg codec libraries
```
从https://www.ffmpeg.org获取源码
从官网或者git clone git://git.videolan.org/x264.git获取x264源码
从 http://www.tortall.net/projects/yasm/releases/ 获取yasm
```

### 安装yasm
```
./configure 
make
make install
```

### 编译安装x264
```
./configure --enable-shared  --enable-pic
make
make install
```

### 编译安装bcg729
```
从git clone https://github.com/BelledonneCommunications/bcg729.git 源码
cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/
make && make install 
export LD_LIBRARY_PATH=/usr/local/lib64:$LD_LIBRARY_PATH
```

### 编译安装libilbc
```
从git clone https://github.com/TimothyGu/libilbc.git 获取源码
执行安装
cmake3 . -DCMAKE_INSTALL_PREFIX=/usr/local/
make && make install 
export LD_LIBRARY_PATH=/usr/local/lib64:$LD_LIBRARY_PATH
```

### 编译安装 amr （两个库）
```
http://sourceforge.net/projects/opencore-amr/下载源码：
				opencore-amr-0.1.5.tar.gz，
				vo-amrwbenc-0.1.3.tar.gz
```
				
### 编译安装libgsm：
```
http://www.quut.com/gsm/下载gsm源码
			编译前需要修改makefile，在第47行加上- fPIC，然后执行编译即可
			cp ../gsm-1.0-pl18/lib/libgsm.a /usr/local/lib64/
			cp inc/* /usr/local/include/
```

### 编译ffmpeg
```
./configure --enable-libx264 --disable-yasm --enable-shared --enable-gpl --enable-nonfree --enable-libilbc --enable-libgsm --enable-libopencore-amrnb --enable-libopencore-amrwb --enable-version3 --enable-libvo-amrwbenc --enable-version3 --extra-libs=-ldl
Make
make install
export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
```

### 编译rtpengine
```
git clone https://github.com/sipwise/rtpengine.git 
cd rtpengine/daemon
make
```

### 编译模块文件（arm用不了内核模块可以省略）
```
  cd rtpengine/kernel-module
  make 
  生成.ko 文件
```

### 编译依赖库文件
```
cd rtpengine/ iptables-extension
make
生成 libxt_RTPENGINE.so，
cp libxt_RTPENGINE.so  /usr/lib64/xtables/ 
用户态运行
/rtpengine -p /var/run/rtpengine.pid --interface=内网ip -n ip:22223 -m 10000 -M 60000 -L 7 --log-facility=local1
执行ng协议，使用内网ip进行交互，监听ip的2223端口，端口范围1万-6万
```

---
收录时间: 2020-11-23

<Vssue :title="$title" />
