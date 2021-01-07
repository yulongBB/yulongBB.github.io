---
date: 2021-01-03T12:00:00-05:00
description: "Pierre Gringoire"
featured_image: "/images/MongoDB副本集集群搭建.png"
tags: ["MongoDB"]
title: "MongoDB副本集集群搭建"
disable_share: true
---

mongodb集群的副本集和sharding模式目前是用的最广的方案，通常这2种方案的选择通过数据量和并发数来权衡。在GB级别的基本上副本集方案可满足，TB级别或以上采用sharding模式，解决单机容量和单机并发能力。这两种既有自己的优势也有自己的缺点，比如sharding模式分片越多，性能自然下降越多。

准备三台服务器

192.168.101.100
192.168.101.101
192.168.101.102

安装mongodb

软件安装目录   /usr/program
软件安装包存放目录   /usr/setups

上传安装包

scp mongodb-linux-x86_64-rhel70-4.0.10.gz root@192.168.101.100:/usr/setups/
scp mongodb-linux-x86_64-rhel70-4.0.10.gz root@192.168.101.101:/usr/setups/
scp mongodb-linux-x86_64-rhel70-4.0.10.gz root@192.168.101.102:/usr/setups/

解压
tar -xzvf mongodb-linux-x86_64-rhel70-4.0.10.gz

移动安装目录

mv mongodb-linux-x86_64-rhel70-4.0.10.gz /usr/program/mongodb

上传配置文件
scp mongodb.conf root@192.168.101.100:/usr/program/mongodb/conf
scp mongodb.conf root@192.168.101.101:/usr/program/mongodb/conf
scp mongodb.conf root@192.168.101.102:/usr/program/mongodb/conf

启动mongodb

mkdir -p /usr/program/mongodb/data
mkdir -p /usr/program/mongodb/log
touch /usr/program/mongodb/log/mongodb.log

lsof -i:27017

./mongod -f /usr/program/mongodb/conf/mongodb.conf


配置集群

./mongo

 config = { _id: "mongors", members: [
{_id: 0, host: "192.168.101.100:27017", priority:2},
{_id: 2, host: "192.168.101.101::27017", priority:1},
{_id: 1, host: "192.168.101.102::27110",arbiterOnly:true}]
}`

rs.initiate(config)


rs.status()