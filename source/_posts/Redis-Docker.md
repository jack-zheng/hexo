---
title: Redis + Docker 搭建方案
date: 2022-01-24 18:35:10
categories:
- Redis
tags:
- redis
- docker
---

通过 Redis 搭建 docker 实验环境

```bash
docker pull redis

# 创建容器
docker run -itd --name redis-test -p 6379:6379 redis

# 进入容器
docker exec -it redis-test /bin/bash

# 开启客户端
redis-cli
# 启动远程客户端
# redis-cli -h host -p port -a password

# 测试
ping
# PONG

# 默认有 16 个数据库，默认用第 0 个
select 1

# 显示当前库下所有的键
keys *

# 清空当前 DB
flushdb

# 删除所有数据库，危
flushall
```

### Issues

清空的时候抛错，我估计是我的 docker redis 没有配置本地存储信息，并不能进行本地化相关操作，创建的时候挂在到本地可能就行了
```bash
flushdb 
(error) MISCONF Redis is configured to save RDB snapshots, but it is currently not able to persist on disk. Commands that may modify the data set are disabled, because this instance is configured to report errors during writes if RDB snapshotting fails (stop-writes-on-bgsave-error option). Please check the Redis logs for details about the RDB error.
```

猜测失败，是配置问题，关闭了快找功能，导致保存失败，可以在客户端使用 `config set stop-writes-on-bgsave-error no` 解决问题

## 数据类型

```bash
# 单个值最大能存储 512M
set name "Jack"

# 拿值
get name

# 查看类型
type name

# 存储 hash
HMSET runoob field1 "Hello" field2 "World"

# hash 取值
HGET runoob field1
#"Hello"
HGET runoob field2
#"World"

# list 列表，左序存入
lpush runoob redis
lpush runoob mongodb
lpush runoob rabbitmq

lrange runoob 0 10
# 1) "rabbitmq"
# 2) "mongodb"
# 3) "redis"

# 查看列表大小
llen runoob

# Set 无序集合
sadd members jack
sadd members tom
sadd members tom
smembers members
# 1) "tom"
# 2) "jack"

# 查看大小
scard members

# zset 有序集合
zadd dbs 0  mongodb
zadd dbs 0  redis
zadd dbs 0  rabit

ZRANGEBYSCORE dbs 0 10000
# 1) "mongodb"
# 2) "rabit"
# 3) "redis"

# 查看大小
zcard dbs
```

## Using Redis in Python

```bash
# 安装依赖
pip install redis

# ipython 操作
import redis
r = redis.StrictRedis(host='localhost', port=6379, db=0)
r.set('foo', 'bar')
print(r['foo'])
# b'bar'
print(type(r.get('foo')))
# <class 'bytes'>
```