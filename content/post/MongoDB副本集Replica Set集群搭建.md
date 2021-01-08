---
date: 2021-01-03T12:00:00-05:00
description: "在三台虚拟机上演示MongoDB集群搭建过程"
featured_image: "/images/MongoDB副本集集群搭建.png"
tags: ["MongoDB"]
title: "MongoDB副本集Replica Set集群搭建"
disable_share: true
---

## 介绍

mongodb集群的搭建包括副本集（Replica Set）和分片（Sharding）两种模式。这两种模式的选择是通过数据量和并发量来权衡。当数据量小（GB级别）时副本集方案可以满足，当数据量大（TB级别）且并发压力大的时候通常采用分片模式。

这两种既有自己的优势也有自己的缺点，比如sharding模式分片越多，性能自然下降越多。

副本集的有点可以实现数据库异地备份、读写分离、故障转移

Replica Set 的成员是一堆有着同样的数据内容 mongod 的实例集合，包含以下三类角色：

主节点（Primary）

是 Replica Set 中唯一的接收写请求的节点，并将写入指令记录到 oplog 上。副本节点通过复制 oplog 的写入指令同步主节点的数据。Secondary。一个 Replica Set 有且只有Primary 节点，当Primar挂掉后，其他 Secondary 或者 Arbiter 节点会重新选举出来一个主节点。应用程序的默认读取请求也是发到 Primary节点处理的。

副本节点（Secondary）

通过复制主节点 oplog 中的指令与主节点保持同样的数据集，当主节点挂掉的时候，参与主节点选举。

仲裁者（Arbiter）

不存储实际应用数据，只投票参与选取主节点，但不会被选举成为主节点。

## 步骤

### 1:准备三台服务器

|  服务器   | 角色  |
|  ----  | ----  |
| 192.168.101.100  | primary |
| 192.168.101.101  | Secondary |
| 192.168.101.102  | Arbiter |

### 2:创建配置文件

```yml
systemLog:
   destination: file
   path: "/data/mongodb/log/mongodb.log"
   logAppend: true
storage:
   dbPath: "/data/mongodb/data/"
   journal:
      enabled: true
   wiredTiger:
      engineConfig:
         cacheSizeGB: 6
replication:
    oplogSizeMB: 10000
    replSetName: "mongors"
processManagement:
   fork: true
   pidFilePath: "/data/mongodb/mongodb.pid"
net:
   bindIp: 127.0.0.1,192.168.101.100,192.168.101.101,192.168.101.102
   port: 27017
setParameter:
   enableLocalhostAuthBypass: false
```

### 3:在三台服务器上安装mongodb

```shell
# 创建安装目录
mkdir -p /usr/program
mkdir -p /usr/setups
# 上传安装包
scp mongodb-linux-x86_64-rhel70-4.0.10.gz root@192.168.101.100:/usr/setups/
scp mongodb-linux-x86_64-rhel70-4.0.10.gz root@192.168.101.101:/usr/setups/
scp mongodb-linux-x86_64-rhel70-4.0.10.gz root@192.168.101.102:/usr/setups/
# 解压
tar -xzvf mongodb-linux-x86_64-rhel70-4.0.10.gz
# 移动安装目录
mv mongodb-linux-x86_64-rhel70-4.0.10.gz /usr/program/mongodb
# 上传配置文件
scp mongodb.conf root@192.168.101.100:/usr/program/mongodb/conf
scp mongodb.conf root@192.168.101.101:/usr/program/mongodb/conf
scp mongodb.conf root@192.168.101.102:/usr/program/mongodb/conf
# 启动mongodb
mkdir -p /usr/program/mongodb/data
mkdir -p /usr/program/mongodb/log
touch /usr/program/mongodb/log/mongodb.log
./mongod -f /usr/program/mongodb/conf/mongodb.conf
# 查看是否启动成功
lsof -i:27017
```

### 4:在主节点配置集群

```shell
# 进入mongo命令行
./mongo
# 创建配置
>  config = {
     _id: "mongors", 
     members: [
      {_id: 0, host: "192.168.101.100:27017", priority:2},
      {_id: 2, host: "192.168.101.101::27017", priority:1},
      {_id: 1, host: "192.168.101.102::27110",arbiterOnly:true}
     ]
   }
# 初始化配置
> rs.initiate(config)
# 查看配置状态
> rs.status()

# 创建用户及权限
> use admin
> db.createUser({user: '<your username>', pwd: '<your password>', roles: ['your role']})
> db.auth('<your username>','<your password>')

# 查看创建的用户
> use admin
> db.system.users.find().pretty()

# 关闭节点（先关闭仲裁节点和副本节点，最后关闭主节点）
> db.shutdownServer()   
```

### 5:添加或更换节点

```shell
# 进入mongo命令行
./mongo
# 用户认证
> use admin
> db.auth('<your username>','<your password>')
# 添加新节点
> rs.add("<new IP>:27017")
# 确认复制集的oplog状态
> rs.printReplicationInfo()
# 删除旧节点
> rs.remove("<old IP>:27017")
# 查看配置状态
> rs.status()
```

### 6. 副本集的定时备份与恢复

备份

```shell
folder=`date +%Y%m%d`
mongodump -h 'mongors/<primary IP>:27017,<secondary IP>:27017' -u '<your username>' -p '<your password>' --oplog --gzip -o mongodb/dump/$folder --authenticationDatabase admin
```

恢复备份

```shell
./mongorestore -h 'mongors/<primary IP>:27017,<secondary IP>:27017' -u '<your username>' -p '<your password>' --oplogReplay --gzip /data/mongodb/dump/<folder>
```

设置定时

使用crontab定时备份

## 链接

[mongodb副本集搭建](https://cloud.tencent.com/developer/article/1452632)

## 资源

[mongodb-linux-x86_64-rhel70-4.0.10](https://pan.baidu.com/s/1KNaGILQg8jj2cCyFlABgww)
提取码：1111
