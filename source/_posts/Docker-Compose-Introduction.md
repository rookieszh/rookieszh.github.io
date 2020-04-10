---
title: Docker Compose Introduction
date: 2019-09-15 13:56:57
categories:
    - Basic Component
tags:
    - Docker
description: "【Docker 系列学习笔记——DockerCompose介绍】1.什么是DockerCompose。2.安装DockerCompose"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

## DockerCompose 概念
Compose有2个重要的概念：
* 项目（Project）：由一组关联的应用容器组成的一个完整业务单元，在 docker-compose.yml 文件中定义。
* 服务（Service）：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。
### 工程、服务、容器
> Docker Compose 将所管理的容器分为三层，分别是工程（project）、服务（service）、容器（container）

Docker Compose 运行目录下的所有文件（docker-compose.yml）组成一个工程,一个工程包含多个服务，每个服务中定义了容器运行的镜像、参数、依赖，一个服务可包括多个容器实例

## 安装 Docker-Compose
### github 安装
```
自动下载适应版本的 Compose，并为安装脚本添加执行权限
curl -L https://github.com/docker/compose/releases/download/1.25.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```
### pip 安装
```
pip install docker-compose
# 查看是否安装成功
docker-compose -v
```
### 安装补全插件
```
curl -L https://raw.githubusercontent.com/docker/compose/1.25.0/contrib/completion/bash/docker-compose > /etc/bash_completion.d/docker-compose
```
### 卸载
```
# 二进制卸载
rm /usr/local/bin/docker-compose
# pip卸载
pip uninstall docker-compose
```