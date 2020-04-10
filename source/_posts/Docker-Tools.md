---
title: Docker Tools
date: 2019-12-03 14:56:57
categories:
    - Basic Component
tags:
    - Docker
description: "本文介绍了三款好用的 Docker 工具：watchtower、docker-slim、ctop"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

#### watchtower：自动更新 Docker 容器
> https://github.com/v2tec/watchtower

Watchtower 监视运行容器并监视这些容器最初启动时的镜像有没有变动。当 Watchtower 检测到一个镜像已经有变动时，它会使用新镜像自动重新启动相应的容器。
```
# Watchtower 本身被打包为 Docker 镜像，因此可以像运行任何其他容器一样运行它。
docker run -d --name watchtower --rm -v /var/run/docker.sock:/var/run/docker.sock  v2tec/watchtower --interval 30
```
使用挂载文件 /var/run/docker.sock 启动 Watchtower 容器。这么做是为了使 Watchtower 可以与 Docker 守护 API 进行交互。我们将 30 秒传递给间隔选项 interval。此选项定义了 Watchtower 的轮询间隔。
比如现在启动一个 Watchtower 可以监视的容器。
```
$ docker run -p 4000:80 --name friendlyhello shekhargulati/friendlyhello:latest
```
现在，Watchtower 将开始温和地监控这个 friendlyhello 容器。当我将新镜像推送到 Docker Hub 时，Watchtower 在接下来的运行中将检测到一个新的可用的镜像。它将优雅地停止那个容器并使用这个新镜像启动容器。它将传递我们之前传递给这条 run 命令的选项。换句话说，该容器将仍然使用 4000:80 发布端口来启动。

默认情况下，Watchtower 将轮询 Docker Hub 注册表以查找更新的镜像。通过传递环境变量 REPO_USER 和 REPO_PASS 中的注册表凭据，可以将 Watchtower 配置为轮询私有注册表。

#### docker-slim：面向容器的神奇减肥药
> https://github.com/docker-slim/docker-slim

docker-slim 工具使用静态和动态分析方法来为你臃肿的镜像瘦身。要使用 docker-slim，可以从 Github 下载 Linux 或者 Mac 的二进制安装包。成功下载之后，将它加入到你的系统变量 PATH 中。
```
$ docker-slim build --http-probe friendlyhello
```

#### ctop：容器的类顶层接口
> https://github.com/bcicen/ctop

ctop 能够提供多个容器的实时指标视图。
```
# mac 安装
$ brew install ctop
```
一旦完成安装，就可以开始使用 ctop 了。现在，你只需要配置 DOCKER_HOST 环境变量。
你可以运行 ctop 命令，查看所有容器的状态。
若只想查看正在运行的容器，可以使用 ctop -a 命令。
