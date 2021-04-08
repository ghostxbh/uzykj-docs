---
title: Centos7上使用内核态安装rtpengine
date: 2020-11-23
tags:
    - CentOS
    - Linux
    - rtpengine
author: ghostxbh
location: blog
summary: Centos7上使用内核态安装rtpengine。
---
# Centos7上使用内核态安装rtpengine

### 更新系统并安装最新的内核
```shell script
yum -y update
yum -y install kernel-devel
yum -y update kernel
reboot
```

### 安装iptables
```shell script
systemctl stop firewalld
systemctl disable firewalld
yum -y install iptables-services iptables-devel
systemctl enable iptables.service
systemctl start iptables.service
iptables -F
service iptables save

# 检查iptables是否处于活动状态并正在运行
systemctl status iptables.service
```

### 安装编译RTPEngine所需的软件包
```shell script
yum -y install epel-release
yum -y install glib glib-devel gcc zlib zlib-devel openssl openssl-devel pcre pcre-devel libcurl libcurl-devel xmlrpc-c xmlrpc-c-devel
yum -y install libevent-devel glib2-devel json-c-devel json-glib json-glib-devel gperf libpcap-devel git perl-IPC-Cmd libiptcdata-devel libiptcdata-devel hiredis hiredis-devel redis iptables-devel libwebsockets-devel
yum -y install spandsp spandsp-devel
rpm --import http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro
yum -y install http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-5.el7.nux.noarch.rpm
yum repolist
yum -y install ffmpeg ffmpeg-devel
yum -y install bcg729 bcg729-devel

# 由于MariaDB是Centos 7上MySQL的默认替代品，我们将使用它代替真正的MySQL。如果您需要MySQL，请安装MySQL数据包或Maria
yum -y install mariadb-devel
```

### 从GitHub存储库获取RTPEngine源代码：
```shell script
cd /usr/src
git clone https://github.com/sipwise/rtpengine.git rtpengine
cd rtpengine
# 出现的问题
# Can't locate IPC/Cmd.pm 

# 解决办法
yum install perl-IPC-Cmd -y
rtpengine media_player.c:4:19: fatal error: mysql.h: No such file or directory

# 解决办法
yum -y install mysql mysql-server mysql-devel
```

### 编译并安装守护程序
```shell script
cd /usr/src/rtpengine/daemon/
make
cp -fr rtpengine /usr/sbin/rtpengine
cp -fr rtpengine /usr/local/bin/rtpengine
```

### 编译并安装iptables扩展
```shell script
cd /usr/src/rtpengine/iptables-extension
make all
cp -fr libxt_RTPENGINE.so /usr/lib64/xtables/.
```

### 编译并安装内核模块xt_RTPENGINE
```shell script
cd /usr/src/rtpengine/kernel-module
make
cp -fr xt_RTPENGINE.ko /lib/modules/`uname -r`/extra/xt_RTPENGINE.ko
depmod -a
modprobe -v xt_RTPENGINE
```

### 检查内核模块是否正确加载：
```shell script
lsmod | grep xt_RTPENGINE

# 在引导时进行加载模块：
echo "# load xt_RTPENGINE module"  >> /etc/modules-load.d/rtpengine.conf
echo "xt_RTPENGINE" >> /etc/modules-load.d/rtpengine.conf

# 检查RTPEngine是否可访问：
ls -l /proc/rtpengine/control | grep root

# 应该看到：
–w–w --- 1 root root 0 Oct 16 10:32 / proc / rtpengine / control
ls -l /proc/rtpengine/list
-r–r–r 1 root root 0 Oct 16 10:32 / proc / rtpengine / list
```

### 安装配置文件
```shell script
# 获取您的外部IP（例如，使用“ ip addr”）

# 在以下命令中，将111.111.111.111更改为您的外部IP

echo "OPTIONS=\"-i 111.111.111.111 -n 127.0.0.1:2223 -m 10000 -M 60000 -L 4 --log-facility=local1 --table=8 --delete-delay=0 --timeout=60 --silent-timeout=600 --final-timeout=7200 –offer-timeout=60 --num-threads=12 --tos=184 –no-fallback\"" > /etc/sysconfig/rtpengine

```

### 配置RTPEngine日志
```shell script
echo "local1.*      -/var/log/rtpengine/rtpengine.log;clean_template" >> /etc/rsyslog.conf
mkdir -p /var/log/rtpengine
touch /var/log/rtpengine/rtpengine.log
```

### 配置logrotate
```shell script
echo "/var/log/rtpengine/rtpengine.log {
daily
rotate 4
missingok
dateext
copytruncate
compress
}" > /etc/logrotate.d/rtpengine
```

### 禁用记录到/ var / log / messages
```shell script
perl -pi.bak -e 's#(\s+)\/var\/log\/messages#;local1.none$1/var/log/messages#' /etc/rsyslog.conf
systemctl restart rsyslog
```


### 添加数据包转发
```shell script
echo 'add 8' > /proc/rtpengine/control

```
### 添加iptables规则以将传入数据包转发到xt_RTPENGINE模块
```shell script
iptables -I INPUT -p udp -j RTPENGINE --id 8
ip6tables -I INPUT -p udp -j RTPENGINE --id 8
service iptables save
service ip6tables save
```

### 配置RTPEngine服务
```shell script
echo "[Unit]
Description=Kernel based rtp proxy
After=syslog.target
After=network-online.target
After=iptables.service
Requires=network-online.target

[Service]
Type=forking
PIDFile=/var/run/rtpengine.pid
EnvironmentFile=-/etc/sysconfig/rtpengine
ExecStart=/usr/local/bin/rtpengine -p /var/run/rtpengine.pid \$OPTIONS

Restart=on-abort

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/rtpengine.service

```
### 启用并启动RTPEengine
```shell script
systemctl enable rtpengine.service
mkdir -p /var/spool/rtpengine
systemctl start rtpengine
```

### 检查状态
```shell script
systemctl status rtpengine.service
```

### 检查g729是否正确安装
```shell script
rtpengine --codecs | grep "G729"
```

### 关于systemd / journald的一些说明
```
Centos 7带有诸如systemd / journald之类的丑陋可恶性，它被硬编码到系统中并且无法关闭。当执行诸如写入日志之类的简单任务时，结果是占用大量CPU。

在低日志级别运行RTPEngine非常重要。我们将日志级别4（-L 4 in /etc/sysconfig/rtpengine）设置为WARNING级别。强烈建议不要使用5-7级。在高呼叫负载下，RTPEngine会生成大量的NOTICE / INFO / DEBUG消息，并且日记处理会立即吞噬您的CPU。

在我们的服务器上，我们执行以下步骤以最小化这种怪异的影响：

在/etc/rsyslog.conf中，在“ $ ModLoad imuxsock”和“ $ ModLoad imjournal”之后添加：

$SystemLogRateLimitInterval 0
$SystemLogRateLimitBurst    0
$IMUXSockRateLimitInterval 0
$IMJournalRatelimitInterval 0
在/etc/systemd/journald.conf中，将当前文件添加/更改为：

RateLimitInterval=0
RateLimitBurst=0
Storage=volatile
Compress=no
RateLimitInterval=0
MaxRetentionSec=5s
执行：

mv /var/log/journal /var/log/journal-disabled
systemctl restart systemd-journald
systemctl restart rsyslog

# FAQ 安装途中如果出现问题，可以在rtpengine代码块执行以下命令，重新安装

清理脏数据

git clean -f -x -d
make clean
```

---
收录时间: 2020-11-23

<Vssue :title="$title" />
