---
title: "Docker部署MongoDB副本集"
date: 2021-01-17T15:39:26+08:00
description: "使用docker技术在容器中部署MongoDB副本集"
featured_image: "/images/Docker部署MongoDB副本集.png"
disable_share: true
---

## 步骤

### 1: 拉取mongo镜像

```shell
docker pull mongo
```

### 2: 启动三个容器实例

```shell
docker run -dit -p 30001:27017 --name mongo1 --net my-mongo-cluster mongo mongod --replSet my-mongo-set 
docker run -dit -p 30002:27017 --name mongo2 --net my-mongo-cluster mongo mongod --replSet my-mongo-set 
docker run -dit -p 30003:27017 --name mongo3 --net my-mongo-cluster mongo mongod --replSet my-mongo-set 
```

### 3:配置副本集
```shell
docker exec -it mongo1 mongo
> db = (new Mongo('localhost:27017')).getDB('test')
test
> config = {
     "_id" : "my-mongo-set",
     "members" : [
         {
             "_id" : 0,
             "host" : "mongo1:27017"
         },
         {
             "_id" : 1,
             "host" : "mongo2:27017"
         },
         {
             "_id" : 2,
             "host" : "mongo3:27017"
         }
     ]
  }
> rs.initiate(config)
```

### 4: 测试副本集是否工作


```shell
# 在主节点插入数据
> db.mycollection.insert({name : 'sample'})
WriteResult({ "nInserted" : 1 })
> db.mycollection.find()
{ "_id" : ObjectId("6003ece416a4633b9565260d"), "name" : "sample" }
# 创建副节点连接，并查看数据是否复制
> db2 = (new Mongo('mongo2:27017')).getDB('test')
> db2.setSecondaryOk()
> db2.mycollection.find()
{ "_id" : ObjectId("6003ece416a4633b9565260d"), "name" : "sample" }
```

