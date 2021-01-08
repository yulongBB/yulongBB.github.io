---
date: 2021-01-02T12:00:00-05:00
description: "利用HUGO生成静态网站并使用Github Actions快速部署静态网站到Github Pages"
featured_image: "/images/使用Github Actions部署静态网站.jpg"
tags: ["HUGO", "GitHub Actions", "Github Pages"]
title: "使用Github Actions部署静态网站"
disable_share: true
---

## 介绍

[HUGO](https://gohugo.io/)是由Go语言实现的静态网站生成器。简单、易用、高效、易扩展、快速部署。

[GitHub Actions](https://github.com/features/actions)是一个CI / CD工具，用于自动化GitHub中的测试，构建和部署过程。

## 步骤

### 1:创建GitHub仓库

创建创建GitHub仓库并命名为`<your GitHub username>.github.io`。

### 2:安装HUGO并创建项目

在Window 10上安装HUGO需要在官方仓库的[releases](https://github.com/gohugoio/hugo/releases)页面下载最新版本HUGO的编译文件到本地并配置环境变量。

```shell
# 创建项目
hugo new <your GitHub username>.github.io
# 安装主题
git clone --depth 1 --recursive https://github.com/gohugoio/hugoThemes.git themes
# 本地运行
hugo server -D
```

### 3:将HUGO项目推送至GitHub仓库

```shell
# 初始化仓库
git init
# 将全部文件添加到暂存区
git add *
# 提交暂存区代码
git commit -m "创建HUGO项目"
# 创建主分支
git branch -M main
# 配置远程Github仓库
git remote add origin https://github.com/<your GitHub username>/<your GitHub username>.github.io.git
# 推送代码到远程Github仓库
git push -u origin main
```

### 4:创建GitHub的token

在[https://github.com/settings/tokens](https://github.com/settings/tokens)页面生成token（勾选repo）并复制到粘贴板。

### 5:将token添加到Github仓库密钥配置中

在`<your GitHub username>.github.io`仓库的`Setting`->`Secrets`页面创建创库密钥`ACTIONS_DEPLOY_KEY`并把token粘贴到value中。

### 6:创建GitHub Action配置文件

在`<your GitHub username>.github.io`仓库创建`workflows/hugo.yml`配置文件。

```yml
name: HUGO
on: push
jobs:
  deploy:
    runs-on: ubuntu-18.04
    steps:
      - name: Git checkout
        uses: actions/checkout@v2

      - name: Update theme
        # (Optional)If you have the theme added as submodule, you can pull it and use the most updated version
        run: git submodule update --init --recursive

      - name: Setup hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: "0.64.0"

      - name: Build
        # remove --minify tag if you do not need it
        # docs: https://gohugo.io/hugo-pipes/minification/
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          personal_token: ${{ secrets.ACTIONS_DEPLOY_KEY }}
          external_repository: <your GitHub username>/<your GitHub username>.github.io
          publish_dir: ./public
          #   keep_files: true
          user_name: <your GitHub username>
          user_email: <your GitHub email>
          publish_branch: gh-pages
        #   cname: example.com
```

## 7:在Github Pages访问部署的静态网站

`http://<your GitHub username>.github.io`

## 8:自定义域名

## 链接

1. [Hgohugoio/hugo](https://github.com/gohugoio/hugo)

2. [Hugo: Deploy Static Site using GitHub Actions](https://ruddra.com/hugo-deploy-static-page-using-github-actions/)

## 资源

[hugo_0.80.0_Windows-64bit](https://pan.baidu.com/s/1PKgRFJfWn50Q6MAo9lnxPA) 提取码：1111
