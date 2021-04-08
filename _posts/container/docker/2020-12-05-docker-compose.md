---
title: 安装docker-compose
date: 2020-12-05
tags:
    - docker-compose
author: ghostxbh
location: blog
summary: 安装docker-compose
---
# 安装docker-compose

## 安装
```shell script
# download
curl -L "https://github.wuyanzheshui.workers.dev/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# permission
chmod +x /usr/local/bin/docker-compose

# soft link
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

## 验证  
```shell script
docker-compose --version
```

---
收录时间: 2020-12-05

<Vssue :title="$title" />
