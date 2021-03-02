# 安装部署Redis

## 安装
```shell script
# 搜索
docker search redis

# 拉取
docker pull redis

# 运行，--requirepass为设置密码
docker run -d --name redis -p 6379:6379 --requirepass "xxxxx"
```


## 部署

### 部署redis主、从服务：

- 创建对应的目录：
```shell script
mkdir -p /opt/redis-sentinel/redis/conf

mkdir -p /opt/redis-sentinel/redis/data

mkdir -p /opt/redis-sentinel/sentinel
```


- 创建redis配置文件：

`/opt/redis-sentinel/redis/conf`下增加`redis-6379.conf、redis-6380.conf`

`/opt/redis-sentinel/redis/`下创建`docker-compose.yml`

- 创建redis服务容器，并启动：

```shell script
docker-compose up -d
```
redis-6379.conf
redis-6380.conf
docker-compose (2).yml

### 部署sentinel服务：

`/opt/redis-sentinel/sentinel/`下增加`sentinel1.conf、sentinel2.conf`

`/opt/redis-sentinel/sentinel/`下创建`docker-compose.yml`

- 启动sentinel容器：
```shell script
docker-compose up -d
```
docker-compose.yml
sentinel1.conf
sentinel2.conf

注：docker下redis.conf需要将daemonize设置为no,因为如果是yes，后台模式会导致docker无任务可做而退出
