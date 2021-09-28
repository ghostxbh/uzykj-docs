---
title: Ubuntu安装新版本nodejs的5种姿势
date: 2021-08-17
tags:
    - Nodejs
    - Ubuntu
author: ganiks
location: cnblogs
summary: Ubuntu安装新版本nodejs的5种姿势。
---
# Ubuntu安装新版本nodejs的5种姿势
**引言：**

写这篇文章之前，关于ubuntu14.04（Trusty）默认安装的NodeJS版本是0.10.25百思不解（什么鬼，哪一年的NodeJS）

写这篇文章之时，NodeJS的LTS版本号都已经10.15.0，当然Ubuntu在2018年也都发行ubuntu18.04（我还没打算用）

系统我可以用4年前的，但是node不行

于是每次都要倒腾Node新版本的安装，踩过一些坑

但是本着刨根问底的原则，还是收获不小

PS：

[http://releases.ubuntu.com/](http://releases.ubuntu.com/)

[https://nodejs.org/zh-cn/download/releases/](https://nodejs.org/zh-cn/download/releases/)

没错，14年4月份（ubuntu14.04）发行时，NodeJS刚刚发行了还不是LTS版本的0.10.25（2014-01-23）



## 姿势A：源码编译安装
**【推荐指数：★★★☆☆】**

官网下载源码：http://nodejs.cn/download/  或者你会用wget ***
```shell
cd your-source-code-directory
./configure
make
sudo make install
```

- 姿势优点：`./configure` 可以自定义安装目录，咳咳，没必要哈
- 姿势缺点：make 费时耗力，内存不大够的VPS或者虚拟机同学建议绕开此姿势
- 扪心自问：Linux系统下用户自己安装的软件（如Node、MongoDB），一般都分布在哪些目录？

> 绝大数开源软件都是公布源代码的，源代码一般被打包为tar.gz归档压缩文件，然后由使用者自行编译为二进制可执行文件  
> 兼容性好／可控制性好／开源软件会大量使用其他开源软件的功能，要解决大量的依赖关系  
> `./configure` 检查编译环境／相关库文件／配置参数，生成makefile  
> make　对源代码进行编译，生成可执行文件  
> make install  将生成的可执行文件安装到当前计算机中  


## 姿势B：添加PPA 用Ubuntu的方式安装
**【推荐指数：★★★★★】**
```shell
user@ubuntu-16.4:~$ logout
Connection to 127.0.0.1 closed.
PS D:\user\user_ubuntu_trusty64> vagrant destroy
default: Are you sure you want to destroy the 'default' VM? [y/N] y
==> default: Forcing shutdown of VM...
==> default: Destroying VM and associated drives...
PS D:\user\user_ubuntu_trusty64> vagrant up

#OK, 又是一个干净的环境了
#记得更换/etc/apt/source.list为本地源（如阿里云）

user@ubuntu-16.4:~$ curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -

## Installing the NodeSource Node.js 8.x LTS Carbon repo...

#添加PPA这个过程好漫长……5-10分钟我这里<br><br>#终于添加成功，开始用咱ubuntu的方式安装
user@ubuntu-16.4:~$ sudo apt-get install -y nodejs
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following NEW packages will be installed:
nodejs
0 upgraded, 1 newly installed, 0 to remove and 12 not upgraded.
Need to get 13.6 MB of archives.
After this operation, 64.3 MB of additional disk space will be used.
Get:1 https://deb.nodesource.com/node_8.x/ trusty/main nodejs amd64 8.15.0-1nodesource1 [13.6 MB]
Fetched 13.6 MB in 8min 13s (27.5 kB/s)
Selecting previously unselected package nodejs.
(Reading database ... 63153 files and directories currently installed.)
Preparing to unpack .../nodejs_8.15.0-1nodesource1_amd64.deb ...
Unpacking nodejs (8.15.0-1nodesource1) ...
Processing triggers for man-db (2.6.7.1-1ubuntu1) ...
Setting up nodejs (8.15.0-1nodesource1) ...

#快来看看成果，不错，这个方式很ubuntu
user@ubuntu-16.4:~$ whereis node
node: /usr/bin/node /usr/bin/X11/node /usr/include/node /usr/share/man/man1/node.1.gz
user@ubuntu-16.4:~$ whereis npm
npm: /usr/bin/npm /usr/bin/X11/npm
user@ubuntu-16.4:~$ node -v
v8.15.0
user@ubuntu-16.4:~$ npm -v
6.4.1
```

### 参考文档：

[https://github.com/nodesource/distributions#debinstall](https://github.com/nodesource/distributions#debinstall)

> NodeSource will maintain Ubuntu distributions in active support by Canonical, including LTS and the intermediate releases.
>
> - Ubuntu 14.04 LTS (Trusty Tahr) - not available for Node.js 10 and later
> - Ubuntu 16.04 LTS (Xenial Xerus)
> - Ubuntu 18.04 LTS (Bionic Beaver)
> - Ubuntu 18.10 (Cosmic Cuttlefish)


## 姿势C：用NPM模块【n】更新Node和NPM
**【推荐指数：★★★★★】**
```shell
#一个干净的ubuntu14.04环境
#默认的方式安装nodejs以及npm
user@ubuntu-16.4:~$ sudo apt install nodejs-legacy npm
user@ubuntu-16.4:~$ npm -v
1.3.10
user@ubuntu-16.4:~$ node -v
v0.10.25

#设置npm源为淘宝源
user@ubuntu-16.4:~$ sudo npm config set registry https://registry.npm.taobao.org

#安装npm包：n
user@ubuntu-16.4:~$ sudo npm i -g n
npm http GET https://registry.npm.taobao.org/n
npm http 200 https://registry.npm.taobao.org/n
npm http GET https://registry.npm.taobao.org/n/download/n-2.1.12.tgz
npm http 200 https://registry.npm.taobao.org/n/download/n-2.1.12.tgz
/usr/local/bin/n -> /usr/local/lib/node_modules/n/bin/n
n@2.1.12 /usr/local/lib/node_modules/n

#用包n安装稳定版本的nodejs
user@ubuntu-16.4:~$ sudo n stable

     install : node-v11.6.0
       mkdir : /usr/local/n/versions/node/11.6.0
       fetch : https://nodejs.org/dist/v11.6.0/node-v11.6.0-linux-x64.tar.gz
######################################################################## 100.0%
installed : v11.6.0


#安装成功后查看node版本，发现sudo可以访问到新安装的版本，凭直觉用户貌似需要重新登录一下
user@ubuntu-16.4:~$ node -v
v0.10.25
user@ubuntu-16.4:~$ npm -v
1.3.10
user@ubuntu-16.4:~$ sudo node -v
v11.6.0
user@ubuntu-16.4:~$ sudo npm -v
6.5.0-next.0

#查看node所在，发现模块n安装的node在/usr/local路径下，而老版本的node还在/usr路径下
user@ubuntu-16.4:~$ whereis node
node: /usr/bin/node /usr/bin/X11/node /usr/local/bin/node /usr/share/man/man1/node.1.gz
user@ubuntu-16.4:~$ /usr/bin/node -v
v0.10.25
user@ubuntu-16.4:~$ /usr/local/bin/node -v
v11.6.0

#希望直觉是对的，logout
user@ubuntu-16.4:~$ logout
Connection to 127.0.0.1 closed.

$ D:\user\user_ubuntu_trusty64> vagrant ssh
Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 3.13.0-164-generic x86_64)
Last login: Fri Jan 18 07:24:38 2019 from 10.0.2.2

#重新登录，是新版啦
user@ubuntu-16.4:~$ node -v
v11.6.0
user@ubuntu-16.4:~$ npm -v
6.5.0-next.0

#嗯，但是 /usr/bin/node 老版本还在，只是 $PATH后面的路径 /usr/local/bin/node 覆盖了老版本
user@ubuntu-16.4:~$ /usr/bin/node -v
v0.10.25

#强迫症的我还是卸载掉老版本吧，嘻嘻
user@ubuntu-16.4:~$ sudo apt remove node-legacy npm
```
模块n的用法：https://www.npmjs.com/package/n

在nodejs中文网 有个阿里云镜像的东东， 默认是最新版本的地址：

[https://npm.taobao.org/mirrors/node/v10.15.0/](https://npm.taobao.org/mirrors/node/v10.15.0/)

（如果需要其他版本的所有镜像，去掉url后面的版本号访问即可）
```
docs/                                             26-Dec-2018 05:30                                 -
win-x64/                                          26-Dec-2018 06:43                                 -
win-x86/                                          26-Dec-2018 06:01                                 -
node-v10.15.0-aix-ppc64.tar.gz                    26-Dec-2018 06:17                                 22797819(21.74MB)
node-v10.15.0-darwin-x64.tar.gz                   26-Dec-2018 05:24                                 16354900(15.6MB)
node-v10.15.0-darwin-x64.tar.xz                   26-Dec-2018 05:25                                 11071128(10.56MB)
node-v10.15.0-headers.tar.gz                      26-Dec-2018 05:31                                 446984(436.51kB)
node-v10.15.0-headers.tar.xz                      26-Dec-2018 05:31                                 336760(328.87kB)
node-v10.15.0-linux-arm64.tar.gz                  26-Dec-2018 04:57                                 18598724(17.74MB)
node-v10.15.0-linux-arm64.tar.xz                  26-Dec-2018 04:59                                 11776444(11.23MB)
node-v10.15.0-linux-armv6l.tar.gz                 26-Dec-2018 04:50                                 17537202(16.72MB)
node-v10.15.0-linux-armv6l.tar.xz                 26-Dec-2018 04:51                                 10762604(10.26MB)
node-v10.15.0-linux-armv7l.tar.gz                 26-Dec-2018 04:53                                 17389653(16.58MB)
node-v10.15.0-linux-armv7l.tar.xz                 26-Dec-2018 04:54                                 10696212(10.2MB)
node-v10.15.0-linux-ppc64le.tar.gz                26-Dec-2018 04:51                                 18620944(17.76MB)
node-v10.15.0-linux-ppc64le.tar.xz                26-Dec-2018 04:52                                 11524352(10.99MB)
node-v10.15.0-linux-s390x.tar.gz                  26-Dec-2018 04:54                                 18879786(18.01MB)
node-v10.15.0-linux-s390x.tar.xz                  26-Dec-2018 04:54                                 11475136(10.94MB)
node-v10.15.0-linux-x64.tar.gz                    26-Dec-2018 06:27                                 18630524(17.77MB)
node-v10.15.0-linux-x64.tar.xz                    26-Dec-2018 06:28                                 12307872(11.74MB)
node-v10.15.0-sunos-x64.tar.gz                    26-Dec-2018 04:52                                 19959848(19.04MB)
node-v10.15.0-sunos-x64.tar.xz                    26-Dec-2018 04:53                                 12839268(12.24MB)
node-v10.15.0-win-x64.7z                          26-Dec-2018 06:49                                 9666719(9.22MB)
node-v10.15.0-win-x64.zip                         26-Dec-2018 06:53                                 16252020(15.5MB)
node-v10.15.0-win-x86.7z                          26-Dec-2018 06:01                                 8593771(8.2MB)
node-v10.15.0-win-x86.zip                         26-Dec-2018 06:01                                 14743242(14.06MB)
node-v10.15.0-x64.msi                             26-Dec-2018 06:56                                 17297408(16.5MB)
node-v10.15.0-x86.msi                             26-Dec-2018 06:01                                 15708160(14.98MB)
node-v10.15.0.pkg                                 26-Dec-2018 05:40                                 16615683(15.85MB)
node-v10.15.0.tar.gz                              26-Dec-2018 05:26                                 36300933(34.62MB)
node-v10.15.0.tar.xz                              26-Dec-2018 05:29                                 20217588(19.28MB)
SHASUMS256.txt                                    26-Dec-2018 16:25                                 3347(3.27kB)
SHASUMS256.txt.asc                                26-Dec-2018 16:25                                 3884(3.79kB)
SHASUMS256.txt.sig                                26-Dec-2018 16:25                                 310(310B)

```
下载 “node-v10.15.0-linux-x64.tar.gz” 来研究下

```shell
user@ubuntu-16.4:~$ wget https://npm.taobao.org/mirrors/node/v10.15.0/node-v10.15.0-linux-x64.tar.gz
2019-01-18 10:09:05 (5.77 MB/s) - ‘node-v10.15.0-linux-x64.tar.gz’ saved [18630524/18630524]

user@ubuntu-16.4:~$ ls
node-v10.15.0-linux-x64.tar.gz
user@ubuntu-16.4:~$ tar xzf node-v10.15.0-linux-x64.tar.gz
user@ubuntu-16.4:~$ ls
node-v10.15.0-linux-x64  node-v10.15.0-linux-x64.tar.gz
user@ubuntu-16.4:~$ ls -l node-v10.15.0-linux-x64
total 164
drwxrwxr-x 2 user user  4096 Dec 26 06:27 bin
-rw-rw-r-- 1 user user 52896 Dec 26 06:27 CHANGELOG.md
drwxrwxr-x 3 user user  4096 Dec 26 06:27 include
drwxrwxr-x 3 user user  4096 Dec 26 06:27 lib
-rw-rw-r-- 1 user user 65839 Dec 26 06:27 LICENSE
-rw-rw-r-- 1 user user 25981 Dec 26 06:27 README.md
drwxrwxr-x 5 user user  4096 Dec 26 06:27 share
user@ubuntu-16.4:~$ ls -l node-v10.15.0-linux-x64/bin node-v10.15.0-linux-x64/lib/node_modules/
node-v10.15.0-linux-x64/bin:
total 38284
-rwxrwxr-x 1 user user 39199960 Dec 26 06:26 node
lrwxrwxrwx 1 user user       38 Dec 26 06:27 npm -> ../lib/node_modules/npm/bin/npm-cli.js
lrwxrwxrwx 1 user user       38 Dec 26 06:27 npx -> ../lib/node_modules/npm/bin/npx-cli.js

node-v10.15.0-linux-x64/lib/node_modules/:
total 4
drwxrwxr-x 10 user user 4096 Dec 26 06:27 npm
```

其实里面是在 linux-x64 环境下已经编译过的nodejs，怎么用呢？

## 姿势D：直接下载编译过的包 分别部署到 /usr/local/bin 以及 /usr/local/lib/node_modules
**【推荐指数：★★★☆☆】**
看下这个包的结构，bin、include、lib、share貌似很眼熟啊，跟linux系统中的 /usr 目录下的结构基本一致
```shell
user@ubuntu-16.4:~$ ls -l /usr/
total 48
drwxr-xr-x   2 root root 20480 Jan 18 09:48 bin
drwxr-xr-x   2 root root  4096 Apr 10  2014 games
drwxr-xr-x  33 root root  4096 Jan 18 09:48 include
drwxr-xr-x  69 root root  4096 Jan 18 09:48 lib
drwxr-xr-x  10 root root  4096 Jan 10 20:43 local
drwxr-xr-x   2 root root  4096 Jan 10 21:43 sbin
drwxr-xr-x 123 root root  4096 Jan 18 09:48 share
drwxr-xr-x   5 root root  4096 Jan 10 21:42 src
```

OK，那我的想法：把 node包下的 bin include lib share四个目录分别跟 /usr下对应同名的目录合并

但是注意一点，/usr这种系统目录，以及目录下的子目录都是 `root:root` 的权限，合并目录后要保证这个一致性

之前这么干过，也一直在用，但是网络上没怎么见过这么干的

## 姿势E：直接下载 编译过的包 部署到 `/usr/local/node` 并修改 环境变量 $PATH
**【推荐指数：★☆☆☆☆】**
还是这个node包，另一种使用方式是整个包的使用，而不是像姿势D中那样拆开了
```shell
user@ubuntu-16.4:~$ sudo cp node-v10.15.0-linux-x64 /usr/local/node -r

#用root用户修改系统文件，修改$PATH(永久生效，所有用户生效）
root@ubuntu-16.4:~/node-install# vim /etc/environment
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/usr/local/node/bin"


#修改sudoers配置，否则普通用户没法用 sudo npm
root@ubuntu-16.4:~/node-install# vim /etc/sudoers
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin:/usr/local/node/bin"
```

## 总结：
- 姿势A：【推荐指数：★★★☆☆】源码编译安装（慢哦慢，还要小心你的内存）

- 姿势B：【推荐指数：★★★★★】添加PPA 用Ubuntu的方式安装（不错，这个方式很ubuntu）

- 姿势C：【推荐指数：★★★★★】用NPM模块【n】更新Node和NPM（不错，这个方式很快，多版本管理更方便）

- 姿势D：【推荐指数：★★★☆☆】直接下载编译过的包 分别部署到 `/usr/local/bin` 以及 `/usr/local/lib/node_modules`（还行，有点鸡贼）

- 姿势E：【推荐指数：★☆☆☆☆】直接下载 编译过的包 部署到 `/usr/local/node` 并修改 环境变量 $PATH（感觉像是Mount挂在系统上，而不是Installatioin，不建议）


> 转载自：[Ubuntu安装新版本nodejs的5种姿势](https://www.cnblogs.com/ganiks/p/5-install-position-for-new-release-nodejs-for-ubuntu.html)

---
收录时间: 2021-08-17

<Vssue :title="$title" />
