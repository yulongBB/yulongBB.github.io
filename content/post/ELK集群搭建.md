---
date: 2021-01-08T15:58:55+08:00
description: "使用ElasticSearch+Logstash+Kibana搭建分布式数据管理分析平台"
featured_image: "/images/ELK集群搭建.png"
tags: ["ElasticSearch", "Logstash", "Kibana"]
title: "ELK集群搭建"
disable_share: true
---

## 介绍

`ELK`是三个开源项目的首字母缩写，这三个项目分别是：Elasticsearch、Logstash 和 Kibana。

Elasticsearch 是一个搜索和分析引擎。一个ElasticSearch集群由一个或多个节点(Node)组成，每个集群都有一个共同的集群名称作为标识。一个ElasticSearch实例即一个Node，一台机器可以有多个实例，正常使用下每个实例应该会部署在不同机器上。ElasticSearch的配置文件中可以通过node.master、node.data来设置节点类型。node.master：表示节点是否具有称为主节点的资格。node.data：表示节点是否存储数据。

Logstash 是服务器端数据处理管道，能够同时从多个来源采集数据，转换数据，然后将数据发送到诸如 Elasticsearch 等“存储库”中。

Kibana 则可以让用户在 Elasticsearch 中使用图形和图表对数据进行可视化。

## 步骤

### 1:准备三台服务器

|  服务器   | 角色  |
|  ----  | ----  |
| 192.168.101.103  | node-1 |
| 192.168.101.104  | node-2 |
| 192.168.101.105  | node-3 |

### 2:安装Elasticsearch

```shell
# 分别将Elasticsearch安装包拷贝到三台服务器上
scp elasticsearch-6.6.2.tar.gz root@192.168.101.103:/usr/setups/
scp elasticsearch-6.6.2.tar.gz root@192.168.101.104:/usr/setups/
scp elasticsearch-6.6.2.tar.gz root@192.168.101.105:/usr/setups/
# 解压
tar -xzvf elasticsearch-6.6.2.tar.gz
# 移动安装目录
mv elasticsearch-6.6.2 /usr/program/
# 创建数据目录
mkdir -p /opt/elasticsearch-6.6.2/data
mkdir -p /opt/elasticsearch-6.6.2/logs
```

### 3:修改配置文件

```yaml
# ---------------------------------- Cluster -----------------------------------
#集群名称
cluster.name: elasticsearch-cluster
# ------------------------------------ Node ------------------------------------
#节点名称
node.name: node-1 
#是不是有资格主节点
node.master: true
#是否存储数据
node.data: true
#最⼤集群节点数
node.max_local_storage_nodes: 3 
# ----------------------------------- Paths ------------------------------------
#数据和存储路径
path.data:  /opt/elasticsearch-6.6.2/data
path.logs:  /opt/elasticsearch-6.6.2/logs
# ---------------------------------- Network -----------------------------------
#⽹关地址
network.host: 0.0.0.0
#端⼝
http.port: 9200
#内部节点之间沟通端⼝
transport.tcp.port: 9300
# --------------------------------- Discovery ----------------------------------
#写⼊候选主节点的设备地址，在开启服务后可以被选为主节点
discovery.zen.ping.unicast.hosts: ["192.168.101.103:9300", "192.168.101.104:9300", "192.168.101.105:9300"]
discovery.zen.minimum_master_nodes: 3

bootstrap.system_call_filter: false
http.cors.allow-origin: "*"
http.cors.enabled: true
http.cors.allow-headers : X-Requested-With,X-Auth-Token,Content-Type,Content-Length,Authorization
http.cors.allow-credentials: true

```

### 4: 运行

文件句柄太少，至少要65536

编辑配置 vi /etc/security/limits.conf

* soft nofile 65536
* hard nofile 131072
* soft nproc 2048
* hard nproc 4096
虚拟内存太少，至少262144

编辑配置 vi /etc/sysctl.conf

vm.max_map_count=655360
激活配置 sysctl -p

```shell
# 创建用户
adduser elasticsearch
# 设置密码
passwd elasticsearch
# 授权
chown -R elasticsearch /usr/program/elasticsearch-6.6.2
chown -R elasticsearch /opt/elasticsearch-6.6.2
# 切换用户
su elasticsearch
# 运行
./elasticsearch -d
```

查看集群状态 http://192.168.101.103:9200/_cat/health?v

### 5: 安装Kibana

```shell
# 上传文件到服务器
scp kibana-6.6.2-linux-x86_64.tar.gz root@192.168.101.103:/usr/setups/
# 解压
tar -xzvf kibana-6.6.2-linux-x86_64.tar.gz
# 移动安装目录
mv kibana-6.6.2-linux-x86_64 /usr/program/
```

### 5: 修改配置

```yaml
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://192.168.101.103:9200", "http://192.168.101.104:9200", "http://192.168.101.105:9200"]
```

### 6: 启动

```shell
nohup ./bin/kibana
```

## 链接

[ElasticSearch 7.8.1集群搭建](https://www.cnblogs.com/chenyanbin/p/13493920.html)

[Elasticsearch安装、集群搭建、kibana安装](https://blog.csdn.net/weixin_41947378/article/details/109384397)