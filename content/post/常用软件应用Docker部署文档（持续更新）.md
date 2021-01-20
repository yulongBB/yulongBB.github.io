---
date: 2021-01-12T11:11:17+08:00
description: "常用软件应用Docker部署文档（持续更新）"
featured_image: "/images/常用软件应用Docker部署文档（持续更新）.png"
tags: []
title: "常用软件应用Docker部署文档（持续更新）"
disable_share: true
---

## Nginx

开源Web服务器和反向代理服务器

1. 拉取镜像

   ```shell
   docker pull nginx
   ```

2. 创建 Nginx 容器

   ```shell
   docker run -di --name nginx -p 80:80 nginx
   ```

3. 将容器内的配置文件拷贝到指定目录

    ```shell
    # 创建目录
    mkdir -p /usr/loacl/nginx
    # 将容器内的配置文件拷贝到指定目录
    docker cp nginx:/etc/nginx /usr/loacl/nginx
    ```

4. 终止并删除容器

   ```shell
   docker stop nginx
   docker rm nginx
   ```

5. 创建 Nginx 容器，并将容器中的 `/etc/nginx` 目录和宿主机的 `/usr/loacl/nginx` 目录进行挂载

   ```shell
   docker run -di --name nginx -p 80:80 -v  /usr/loacl/nginx:/etc/nginx nginx
   ```

## MySQL

关系型数据库

1. 拉取镜像 `docker pull mysql:8.0.21`

2. 创建容器 `docker run -di --name mysql -p 3306:3306 -v /usr/docker/mysql/conf:/etc/mysql/conf.d -v /usr/docker/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=<your password> mysql:8.0.21`

3. 进入容器并连接客户端

```shell
# 进入容器
docker exec -it mysql /bin/bash
# 使用 MySQL 命令打开客户端
mysql -uroot -p<your password> --default-character-set=utf8
```
