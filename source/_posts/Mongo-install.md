---
title: Mongo DB 安装
date: 2021-12-02 20:20:53
categories:
- DB
tags:
- MongoDB
---

使用 Docker 的方式安装 MongoDB

1. docker pull mongo
2. `docker run --name mymongo -d mongo` 使用默认设置启动
3. `docker exec -it mymongo bash` 链接控制台输入 `mongod --version` 验证安装情况
4. `mongo` 链接数据库, `exit` 退出

## 基本命令

```sql
show dbs -- 查看库

db -- 当前库名

use 库名 -- 使用

> db.students.insertOne({"name":"Jack"}) -- 插入数据
{
	"acknowledged" : true,
	"insertedId" : ObjectId("61a8bfa9e0344414ad0ee1e3")
}

> show collections -- 查看插入结果
students

> db.students.find() -- 查询
{ "_id" : ObjectId("61a8bfa9e0344414ad0ee1e3"), "name" : "Jack" }
```

## 问题

Mac 上测试，启动立马停止，通过 `docker logs mymongo` 可以看到异常

```txt
{"t":{"$date":"2021-12-02T12:12:30.478+00:00"},"s":"E",  "c":"STORAGE",  "id":22312,   "ctx":"initandlisten","msg":"Error creating journal directory","attr":{"directory":"/data/db/journal","error":"boost::filesystem::create_directory: No space left on device: \"/data/db/journal\""}}
```

Google 了一下，说是 Mac 才有的，docker volumn 不够了，可以通过指定挂在的 volumn 或者清理不用的 volumn 解决问题

```bash
docker run -d -p 3000:27017 -v /Users/user/work/m1:/data/db --name m1 --net mongo-net mongo mongod
# or
docker volume prune
# 最后，上面那个 prune 没用，用了下面这个简化版的
docker  run -d -p 3000:27017 -v /Users/xxxx/mongodb/data/db:/data/db --name mymongo mongo

docker exec -it some-mongo bash
docker logs some-mongo
```