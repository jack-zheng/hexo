---
title: MongoDB 基本概念 + Python 访问
date: 2022-01-18 16:12:21
categories:
- database
tags:
- mongodb
---

## Setup MongoDB the Docker Way

* [镜像地址](https://hub.docker.com/_/mongo), 包含了很多常用操作，比如挂载卷，导出数据，设置密码等

```bash
# 下载并启动一个镜像, 映射到 27017 端口并设置密码访问
# docker run -itd --name mymongo -p 27017:27017 mongo --auth
# 测试时不要加 --auth, 自找麻烦
docker run -itd --name mymongo -v /Users/i306454/mongodb/data/db:/data/db -p 27017:27017 mongo

# 链接到 db 终端
docker exec -it mymongo mongo admin

# 创建 root/root 用户, 指定可操作的数据库
db.createUser({ user:'root',pwd:'root',roles:[ { role:'userAdminAnyDatabase', db: 'admin'},"readWriteAnyDatabase"]});

# 以指定用户登陆
db.auth('root', 'root')
# 1 授权成功

use admin
# 查看系统信息
db.getCollectionNames()
```

### Issues

mymongo 启动失败(之前遇到过，我记得我记录过了，怎么找不到那片文章了呢（；￣ェ￣）), 使用 docker logs mymongo 可以看到异常信息 "{"t":{"$date":"2022-01-18T08:26:55.448+00:00"},"s":"E",  "c":"STORAGE",  "id":22312,   "ctx":"initandlisten","msg":"Error creating journal directory","attr":{"directory":"/data/db/journal","error":"boost::filesystem::create_directory: No space left on device: \"/data/db/journal\""}}"，创建容器时指定挂在位置即可

## MongoDB 基本操作

| RDBMS  | MongoDB                         |
| :----- | :------------------------------ |
| 表     | 集合                            |
| 行     | 文档                            |
| 列     | 字段                            |
| 表联合 | 嵌入文档                        |
| 主键   | 主键(MongoDB 提供了 key 为 _id) |

```bash
# 显示当前库对象，默认为 test 库
db

# 创建并使用库
use local


# 插入数据时会自动创建数据库
db.local.insert({"name":"Jack"})

# 删除数据库
use local
db.dropDatabase()
# 查看结果
show dbs

# 创建集合
db.createCollection("runoob")

# 查看集合
show collections

# 创建集合并指定参数
db.createCollection("mycol", { capped: true, autoIndexId: true, size: 6142800, max: 10000})

# 其实不需要特别创建集合，插入文档时会自动生成
db.mycal2.insert({"name":"Jack"})
> show collections 
mycal2
mycol
runoob

# 删除集合
db.mycal2.drop()

# 插入文档
db.collection_name.insert(doc)

# 插入多个文档
# db.collection.insertMany(
#    [ <document 1> , <document 2>, ... ],
#    {
#       writeConcern: <document>,
#       ordered: <boolean>
#    }
# )
db.runoob.insertMany([{"key":"name"}, {"a":1}])

# 查看已插入的文档
db.runoob.find()

# 更新文档
db.runoob.update({"name":"Jack"}, {$set:{"name":"Jerry"}})
# 修改多条
db.col.update({"name":"Jack"}, {$set:{"name":"Jerry"}},{multi:true})
# save 主键存在就更新，不存在就插入

# 删除文档
# db.collection.remove(
#    <query>,
#    <justOne>
# )
# 2.6以上版本
# db.collection.remove(
#    <query>,
#    {
#      justOne: <boolean>,
#      writeConcern: <document>
#    }
# )
db.runoob.remove({"a":1})
# remove 已过时，官方推荐 deleteOne(), deleteMany()

# 查询文档
# db.collection.find(query, projection)
# .pretty() 格式化输出
db.runoob.find({$or:[{"name":"Jerry"}, {"key":"test"}]}).pretty()

# AND + OR 的例子
db.col.find({"likes": {$gt:50}, $or: [{"by": "菜鸟教程"},{"title": "MongoDB 教程"}]}).pretty()

# 条件操作
# (>) 大于 - $gt
# (<) 小于 - $lt
# (>=) 大于等于 - $gte
# (<= ) 小于等于 - $lte

# 清空集合
db.col.remove({})
# 查看 likes > 100 的数据
db.col.find({likes : {$gt : 100}})

# type 操作
db.col.find({title: {$type:2}})
# or
db.col.find({title: {$type:'string'}})

# limit, 同 SQL 中的 limit
# db.collection.find(query, projection)
# 查询返回所有数据，只显示 title, _id 属性
db.col.find({},{"title":1,_id:0}).limit(2)

# skip 跳过指定数量的数据
db.col.find({},{"title":1,_id:0}).limit(1).skip(1)

# 排序，1 升序，-1 降序
db.col.find({},{"title":1,_id:0}).sort({"likes":-1})

# 创建索引
db.col.createIndex({"title":1})
# 也可以创建复合索引
# 索引可以有额外参数
db.values.createIndex({open: 1, close: 1}, {background: true})

# 聚合，类似 count(*)
db.COLLECTION_NAME.aggregate(AGGREGATE_OPERATION)

# 其实只是 count 的话有更简单的方法
db.col.count()
db.col.count({'age':24})
```

## 集成 pymongo

安装 lib `pip install pymongo`

```python
from pymongo import MongoClient

# 如果是默认配置
client = MongoClient()
# 如果特殊配制
# myclient = pymongo.MongoClient("mongodb://localhost:27017/")

# 显示 db 名字
client.list_database_names()

# 创建集合
collection = client["ttt"]

# 展示集合名字
collection.list_collection_names()

# 创建文档
mydoc = collection["sites"]
# 插入集合
mydict = { "name": "RUNOOB", "alexa": "10000", "url": "https://www.runoob.com" }
mydoc.insert_one(mydict)
# 插入多条
mydoc.insert_many(mylist)

# 查询
mydoc.find_one()
# 查询所有
for sub in mydoc.find():
    print(sub)

# 指定字段
for x in mydoc.find({},{ "_id": 0, "name": 1, "alexa": 1 }): 
    print(x)

# 指定条件
mydoc.find({ "name": "RUNOOB" })
# 比较
mydoc.find({ "name": { "$gt": "H" } })
# 正则
mydoc.find({ "name": { "$regex": "^R" } })
# 限制数量
mydoc.find().limit(3)

# 修改
myquery = { "alexa": "10000" }
newvalues = { "$set": { "alexa": "12345" } }
mydoc.update_one(myquery, newvalues)

# 排序
mydoc.find().sort("alexa")
mydoc.find().sort("alexa", -1)

# 删除
myquery = { "name": "Taobao" }
mydoc.delete_one(myquery)
# 删除多个
myquery = { "name": {"$regex": "^F"} }
mydoc.delete_many(myquery)
# 删除所有
mydoc.delete_many({})
# 删除集合
mydoc.drop()
```