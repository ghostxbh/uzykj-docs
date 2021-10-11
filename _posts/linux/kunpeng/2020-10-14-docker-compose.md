---
title: 安装docker-compose
date: 2020-10-14
sidebar: 'auto'
categories:
  - Linux
tags:
  - docker
  - docker-compose
  - 鲲鹏
author: ghostxbh
location: blog
summary: 根据docker官网安装docker-compose即可(docker支持ARM)。
---
# 安装docker-compose

### 安装docker
根据docker官网安装即可(docker支持ARM)

安装docker-compose

docker-compose上的发行包没有找到arm的 所以选择使用pip安装

```shell script
yum update
yum install -y libffi libffi-devel openssl-devel python3 python3-pip python3-devel
pip3 install docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose -v
```

### 卸载docker-compose
```shell script
pip3 uninstall docker-compose
```

---
收录时间: 2020-10-14

<Vssue :title="$title" />
