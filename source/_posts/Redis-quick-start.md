---
title: Redis 弹射起步
date: 2021-04-30 18:19:46
categories:
- 弹射起步
tags:
- redis
---


## Nosql 概述

> 单机数据库：

1. 数据量如果太大，一个机器放不下
2. 数据的索引，一个机器内存放不下
3. 访问量，一个服务器承受不了

> Memcached(缓存) + MySQL+ 垂直拆分

网站 80% 的情况都是在读数据，每次都查 DB 就十分慢，采用缓存保证效率

发展过程：优化数据结构和索引 -> 文件缓存(IO) -> Memcached

> 分库分表 + 水平拆分 + MySQL 集群

* ISAM：早期，表锁，效率低
* Innodb: 行所

> 如今，数据量大，变化快，MySQL 不太适用这种场景了

MySQL 也不适合存储比较大的文件，博客，图片。表大，效率低，需要专门的数据库

> 为什么用 NoSQL

用户信息爆发式增长，NoSQL 可以解决上面的问题

NoSQL = Not Only SQL

关系型数据库 = 表 + 行 + 列

泛指非关系型数据库，随着 web2.0 互联网的诞生，传统的关系型数据库很难对付，尤其是超大规模的高并发社区。

很多数据类型，比如个人信息，社交网络，地理位置不需要一个固定格式，不需要过多操作就能横向操作

> 特点

1. 方便扩展
2. 大数据量高性能 (redis 写 8w/s, 读 11w/s, NoSQL 的缓存记录级，是一种细粒度的缓存，性能会比较高)
3. 数据类型是多样型的(不需要事先设计数据库)

> RDBMS Vs NoSQL

RDBMS

* 结构化组织(表/列)
* SQL
* 数据和关系都存在单独的表中
* 数据操作语言，数据定义语言(DSL, DML)
* 严格的一致性

NoSQL

* 不仅仅是SQL
* 没有固定的查询语言
* 存储类型多样(键值，文档，列存储，图形数据-社交关系)
* 最终一致性
* CAP定理和BASE 异地多活
* 高性能，高可用，高可扩

> 3v + 3高

3v: 主要描述问题

1. 海量Volume
2. 多样Variety
3. 实时Velocity

3 高:

1. 高并发
2. 高可扩
3. 高性能

真实情况：NoSQL + RDBMS

## 典型电商网站分析

1. 商品基本信息，名称，价格，商家信息，存关系型数据库
2. 商品描述，评论(文字较多) MongoDB
3. 图片，分布式文件系统 FastDFS, 淘宝 TFS，Google GFS， Hadoop HDFD, 阿里云 OSS
4. 商品关键字，搜索，solr, ES, ISearch
5. 商品热门的波段信息，内存数据库，Redis, Tair, Memcache
6. 商品交易 - 第三方接口

## NoSQL 四大分类

KV键值对:

* 新浪： Redis
* 美团：Redis + Tair
* 阿里，百度：Redis + memocache

文档型数据库(bson)

* MongoDB(一般需要掌握)
  * MongoDB 是一个给予分布式文件存储的数据库，C++ 编写，主要用来处理大量文档
  * MongoDB 是一个介于关系型数据库和非关系型数据库的中间产品

列存储数据库：

* HBase
* 分布式文件系统

图关系数据库：

* 社交网络
* Neo4j

## Redis 入门

Remote Dictionary Server: c，基于内存，可持久化， KV，多语言 API

> 能干嘛

1. 内存存储，可持久化(RDB, AOF)
2. 效率高，可高速缓存
3. 发布订阅系统
4. 地图信息分析
5. 计数器 - 订阅量，计时器

## 安装

CentOS:

1. 下载 redis 安装包 redis.xx.tar.gz
2. 放到 /opt 下， `tar -zxcf redis.xx.tar.gz`, 加压后的文件包含配置文件 redis.conf
3. cd 到加压文件下 yum install gcc-c++ 安装 gcc
4. make 编译程序 + make install
5. redis 默认安装路径 /usr/local/bin
6. redis 配置文件 copy 到安装目录下  `mkdir myconfig` + `cp /opt/redis-6.0.6/redis.conf .`
7. redis 默认不是后台启动，修改一下 `daemonize yes`
8. `redis-server myconfig/redis.config` 运行

PS: 6.0.6 版本的 redis 自带配置文件了。。。

```txt
make install
cd src && make install
make[1]: Entering directory `/opt/redis-6.0.6/src'

Hint: It's a good idea to run 'make test' ;)

    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
make[1]: Leaving directory `/opt/redis-6.0.6/src'

# 启动提示
redis-server redis.conf
9841:C 30 Apr 2021 22:15:30.703 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
9841:C 30 Apr 2021 22:15:30.703 # Redis version=6.0.6, bits=64, commit=00000000, modified=0, pid=9841, just started
9841:C 30 Apr 2021 22:15:30.703 # Configuration loaded

redis-cli -p 6379   # 客户端链接测试
127.0.0.1:6379> ping
PONG
127.0.0.1:6379> set name jack
OK
127.0.0.1:6379> get name
"jack"
127.0.0.1:6379> keys *
1) "name"
127.0.0.1:6379> shutdown    # 推出
not connected> exit
[root@iZuf6jaqcwqwkqvydc5mscZ bin]# ps -ef | grep redis
root      9850  5506  0 22:19 pts/0    00:00:00 grep --color=auto redis

```

PS: centos 默认的 gcc 版本是 4.8.5 在编译 redis 时会报错，需要升级到 5.3 以上才行

```sh
gcv -v # 查看版本

make distclean # 清除编译生成的文件

yum -y install centos-release-scl   # 安装新版本 gcc
yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils

# 需要注意的是scl命令启用只是临时的，退出shell或重新打开一个shell就会恢复原系统gcc版本
scl enable devtoolset-9 bash
gcc -v

#执行以下命令永久使用
echo "source /opt/rh/devtoolset-9/enable" >> /etc/profile
# 注：执行完此命令后，其它的shell窗口需要关闭重新打开才生效。
# 重新打开shell窗口，再次编译
```

## 性能测试

自带的性能测试工具 redis-benchmark

```sh
# 测试 100 个并发，每个并发 100 000 个请求
redis-benchmark -h localhost -p 6379 -c 100 -n 100000


# ====== SET ======
#   100000 requests completed in 1.44 seconds         # 100 000 个请求 1.44 秒完成
#   100 parallel clients                              # 100 个并发 
#   3 bytes payload
#   keep alive: 1
#   host configuration "save": 900 1 300 10 60 10000
#   host configuration "appendonly": no
#   multi-thread: no

# 49.47% <= 1 milliseconds
# 99.96% <= 2 milliseconds
# 100.00% <= 2 milliseconds
# 69492.70 requests per second

```

## 基础知识

redis 默认有 16 个数据库(redis.conf - databases 16), 默认用第一个，可以通过 `select n` 修改

```redis
127.0.0.1:6379> dbsize  # 查看大小
(integer) 5
127.0.0.1:6379> select 3 # 换数据库
OK
127.0.0.1:6379[3]> dbsize
(integer) 0
127.0.0.1:6379> keys *    # 查看已有的 key
1) "mylist:{tag}"
2) "key:{tag}:__rand_int__"
3) "myhash:{tag}:__rand_int__"
4) "counter:{tag}:__rand_int__"
5) "name"
127.0.0.1:6379> flushdb   # 清空数据库
OK
127.0.0.1:6379> keys *
(empty array)
127.0.0.1:6379[1]> flushall # 清空所有数据库
OK
```

PS: 八卦，为什么端口时 6379 - 作者追星

> Redis 时单线程的！

Redis 基于内存操作， CPU 不是瓶颈，瓶颈在内存和带宽

Redis 为 C 语言开发， 10w+ QPS, 完全不比 Memecache 差

> Redis 为什么单线程还这么快？

1. 误区：高性能的服务器一定是多线程？
2. 误区：多线程一定比单线程效率高

核心：redis 是将所有数据全部存放在内存中的，省去了上下文切换的时间

## Redis-key

```sh
127.0.0.1:6379> exists name # 判断是否存在
(integer) 1
127.0.0.1:6379> move name 1 # 移除 key
(integer) 1
127.0.0.1:6379> set name jack
OK
127.0.0.1:6379> expire name 10  # 设置过期时间, 单位s
(integer) 1
127.0.0.1:6379> ttl name      # 查看 key 剩余时间
(integer) 7
127.0.0.1:6379> ttl name
(integer) 2
127.0.0.1:6379> ttl name
(integer) -2
127.0.0.1:6379> get name
(nil)
127.0.0.1:6379> ttl name
(integer) -2
127.0.0.1:6379> type name   # 查看类型
string
127.0.0.1:6379> set key v1
OK
127.0.0.1:6379> append key "hello"  # 追加，如果不存在则新加
(integer) 7
127.0.0.1:6379> get key
"v1hello"
127.0.0.1:6379> strlen key  # 给出字符串长度
(integer) 7
127.0.0.1:6379> append key "world"
(integer) 12
127.0.0.1:6379> set views 0
OK
127.0.0.1:6379> incr views  # +1 操作
(integer) 1
127.0.0.1:6379> get views
"1"
127.0.0.1:6379> decr views # -1 操作
(integer) 0
127.0.0.1:6379> decr views
(integer) -1
127.0.0.1:6379> incrby views 10 # +10
(integer) 9
127.0.0.1:6379> decrby views 4 # -4
(integer) 5

### 字符串操作
127.0.0.1:6379> set key "hello,jack"
OK
127.0.0.1:6379> getrange key 0 3  # sub string 
"hell"
127.0.0.1:6379> getrange key 0 -1 # 取全部字符串
"hello,jack"

### 替换
127.0.0.1:6379> set key2 asdfghj
OK
127.0.0.1:6379> setrange key2 1 xx  # 从第 n 位开始替换
(integer) 7
127.0.0.1:6379> get key2
"axxfghj"

###
# setex - set with expire 
# setnx - set if not exist

127.0.0.1:6379> setex key3 30 hello # 添加并设置过期时间
OK
127.0.0.1:6379> ttl key3
(integer) 27
127.0.0.1:6379> setnx key4 redis    # 如果不存在就添加，否则失败
(integer) 1
127.0.0.1:6379> keys *
1) "key"
2) "key4"
3) "key2"
4) "key3"
127.0.0.1:6379> setnx key4 redis2
(integer) 0
127.0.0.1:6379> get key4
"redis"
127.0.0.1:6379> keys *
1) "key"
2) "key4"
3) "key2"


## 批量操作
127.0.0.1:6379> mset k1 v1 k2 v2 k3 v3
OK
127.0.0.1:6379> keys *
1) "k3"
2) "k1"
3) "k2"
127.0.0.1:6379> mget k1 k2 k3
1) "v1"
2) "v2"
3) "v3"
127.0.0.1:6379> msetnx k1 v1 k4 v4  # 由于是原子操作，所以 k4 没有加成功
(integer) 0
127.0.0.1:6379> keys *
1) "k3"
2) "k1"
3) "k2"

## 对象转化
# 需要保存一个 user: {id:1, name:zhangsan, age:2} 对象时，可以这么做
# user:{id}:{field}
127.0.0.1:6379> mset user:1:name zhangsan user:1:age 2
OK
127.0.0.1:6379> mget user:1:name user:1:age
1) "zhangsan"
2) "2"

## getset 先 get 再 set
127.0.0.1:6379> getset db redis # 如果不存在 返回 nil
(nil)
127.0.0.1:6379> get db
"redis"
127.0.0.1:6379> getset db mongodb # 如果存在，获取并更新
"redis"
127.0.0.1:6379> get db
"mongodb"
```