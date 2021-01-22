---
title: "使用Django开发应用"
description: "使用基于python的django框架搭建web应用"
date: 2021-01-20T14:38:39+08:00
featured_image: "/images/使用Django开发应用.png"
author: "玉龙BB"
disable_share: true
---

## 介绍

Django是一个高级Python Web框架，鼓励快速开发和简洁实用的设计。它由经验丰富的开发人员构建，它解决了Web开发的大部分麻烦，因此您可以专注于编写应用程序而无需重新发明轮子。它是免费和开源的。

## 步骤

### 1：创建项目

```shell
django-admin startproject <your project name>
```

### 2: 创建应用

```shell
python manage.py startapp <your application name> 
```

### 3: 初始化数据

```shell
python manage.py migrate

python manage.py makemigrations <your application name> 

python manage.py sqlmigrate <your application name>  0001
```

### 4: 创建超级管理员

```shell
python manage.py createsuperuser
```
