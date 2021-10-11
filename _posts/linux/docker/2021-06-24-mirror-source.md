---
title: Docker 替换镜像源
date: 2021-06-24
sidebar: 'auto'
categories:
  - Linux
tags:
  - docker
author: ghostxbh
location: blog
summary: Docker 替换镜像源
---

# Docker 替换镜像源
由于docker默认的源为国外官方源，下载速度较慢，所以在学习和使用中可改为国内的镜像源，这样速度会提高狠多。

### 方法一
编辑`/etc/docker/daemon.json`

```shell
vi /etc/docker/daemon.json
#添加如下网易镜像源
{
    "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
```

### 方法二
编辑`/etc/sysconfig/docker`，在OPTIONS变量后追加参数` --registry-mirror=镜像源地址`

```shell
vi /etc/sysconfig/docker
#编辑OPTIONS，添加中国科技大学的镜像源
OPTIONS='--selinux-enabled --log-driver=journald --registry mirror=https://docker.mirrors.ustc.edu.cn'
```

### 方法三
编辑`/etc/default/docker`,添加`DOCKER_OPTS="--registry-mirror=镜像源"`

```shell
vi /etc/default/docker
#指定镜像源为阿里的镜像源
DOCKER_OPTS="--registry-mirror=https://pee6w651.mirror.aliyuncs.com"
```

其他：docker pull拉取镜像时也可以指定仓库下载地址

### 重启Docker
```
#配置生效
sudo systemctl daemon-reload

#重启docker
systemctl restart docker.service

or

sudo service docker restart

#测试
docker search mysql
```

### Docker国内源

Docker 官方中国区       
https://registry.docker-cn.com

网易        
http://hub-mirror.c.163.com

中国科技大学        
https://docker.mirrors.ustc.edu.cn

阿里云      
https://cr.console.aliyun.com/

### 资源
- [docker docs](https://docs.docker.com/registry/recipes/mirror/)

---
收录时间: 2021-06-24

<Vssue :title="$title" />
