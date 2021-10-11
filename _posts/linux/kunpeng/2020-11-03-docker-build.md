---
title: 使用docker构建多平台镜像
date: 2020-11-03
sidebar: 'auto'
categories:
  - Linux
tags:
  - docker
  - 鲲鹏
author: ghostxbh
location: blog
summary: 由于鲲鹏的处理器是arm的，很多开源软件如redis最新版本都没有arm的镜像，所以在鲲鹏上运行这些开源软件会有问题。
---
# 使用docker构建多平台镜像
## 前言
最近在玩鲲鹏，由于鲲鹏的处理器是arm的，很多开源软件如redis最新版本都没有arm的镜像，所以在鲲鹏上运行这些开源软件会有问题

解决办法也很简单，把redis源代码拉下来，使用原本的Dockerfile在arm的基础上在打包镜像就好了

## arm和x86的区别
ARM和X86的区别
ARM属于精简指令集（RISC）和X86属于复杂指令集（CISC)

精简指令集（RISC）和 复杂指令集（CISC)的区别 打个比方：

比如说我们要命令一个人吃饭，那么我们应该怎么命令呢？ 我们可以直接对他下达“吃饭”的命令，也可以命令他“先拿勺子，然后舀起一勺饭，然后张嘴，然后送到嘴里，最后咽下去”。 从这里可以看到，对于命令别人做事这样一件事情，不同的人有不同的理解 有人认为，如果我首先给接受命令的人以足够的训练，让他掌握各种复杂技能（即在硬件中实现对应的复杂功能）

那么以后就可以用非常简单的命令让他去做很复杂的事情——比如只要说一句“吃饭”，他就会吃饭

但是也有人认为这样会让事情变的太复杂，毕竟接受命令的人要做的事情很复杂，如果你这时候想让他吃菜怎么办？ 难道继续训练他吃菜的方法？我们为什么不可以把事情分为许多非常基本的步骤，这样只需要接受命令的人懂得很少的基本技能，就可以完成同样的工作 无非是下达命令的人稍微累一点——比如现在我要他吃菜，只需要把刚刚吃饭命令里的“舀起一勺饭”改成“舀起一勺菜”，问题就解决了，多么简单。这就是“复杂指令集”和“精简指令集”的逻辑区别。
使用docker buildx构建多平台镜像
linux下主要平台信息 linux/amd64 目前最主流的 X86_64 linux/arm64 linux/arm linux/arm/v6 linux/arm/v7
可以看到linux下有很多主要的平台，那上面的redis来举例，如果我要把redis移植到不同的平台难道还要每次都进行编译？答案当然不是

既然我面临这样的问题，肯定其他人也面临这样的问题。

我们看下nginx 官方所提供的docker镜像支持很多版本 在这里插入图片描述
<img src="http://file.uzykj.com/892fd8da-f9ea-7ce5-ea78-51cce95dcb90.png" heigth=50% widt=50%>

## Docker Buildx
Docker Buildx 是一个CLI插件，扩展了docker命令，并完全支持 Moby BuildKit 构建器工具包提供的功能. 它提供了与 docker build 相同的用户体验，并具有许多新功能，例如：创建范围内的构建器实例和同时针对多个节点进行构建。

### 安装
直接安装 Docker v19.03 版本，该版本已包含 Docker Buildx 组件，因为目前还是实验功能，默认没有开启。

### 开启
```
$ vim ~/.docker/config.json
{
    "experimental": "enabled"
}
```
配置新增Docker配置文件，并重启Docker服务（systemctl daemon-reload systemctl restart docker）

环境变量配置开启方法
```
$ export DOCKER_CLI_EXPERIMENTAL=enabled
```

### 创建并使用
- 创建并使用
```
docker buildx create --use --name testbuilder
```

- 查看
```
docker buildx ls
```
在这里插入图片描述
<img src="http://file.uzykj.com/3cfe189b-7207-24aa-f4f6-09bfe162d033.png" heigth=50% widt=50%>

### 安装qemu-user-static
```shell script
$ uname -m
x86_64
```

```
$ docker run --rm -t arm64v8/ubuntu uname -m
standard_init_linux.go:211: exec user process caused "exec format error"

$ docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
```

- 如果要使用其他平台请运行相应平台命令  如下
```
$ docker run --rm -t arm64v8/ubuntu uname -m
aarch64
$ docker run --rm -t arm32v6/alpine uname -m
armv7l

$ docker run --rm -t ppc64le/debian uname -m
ppc64le

$ docker run --rm -t s390x/ubuntu uname -m
s390x

$ docker run --rm -t arm64v8/fedora uname -m
aarch64

$ docker run --rm -t arm32v7/centos uname -m
armv7l

$ docker run --rm -t ppc64le/busybox uname -m
ppc64le
```

- 打包镜像
注意你的Dockerfile的基础镜像需要支持你的打包平台镜像
```
FROM node:latest
```

你要打包成arm和x86的那么，node基础镜像必须支持arm和x86
构建好的镜像不会保存本地，需要推送到镜像仓库，我这里使用docker hub
```
docker login
docker buildx build -t daxion/buildx:opensips --platform=linux/arm,linux/arm64,linux/amd64 --push .
```
在这里插入图片描述 在push的时候我们可能会等很久出现这样的错误 failed to solve: rpc error: code = Unknown desc = server message: insufficient_scope: authorization failed
<img src="http://file.uzykj.com/979374b9-f178-5157-eccb-72ef6f265ded.png" heigth=50% widt=50%>

我们这里只需要指定下name=docker仓库的地址，私库更换下网址和端口就行就了

```
docker buildx build -t daxion/buildx:opensips --platform=linux/arm,linux/arm64,linux/amd64 . --push --output type=image,name=docker.io:443,push=true
```
最后成功截图 在这里插入图片描述
<img src="http://file.uzykj.com/439193d3-7007-af8f-5770-63afb6271e55.png" heigth=50% widt=50%>

---
收录时间: 2020-11-03

<Vssue :title="$title" />
