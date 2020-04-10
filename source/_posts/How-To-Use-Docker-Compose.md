---
title: How To Use Docker Compose
date: 2019-09-15 22:56:57
categories:
    - Basic Component
tags:
    - Docker
description: "【Docker 系列学习笔记——DockerCompose命令】介绍了常用的DockerCompose命令"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---


## 常用命令
### 命令选项
```
-f, --file FILE 指定使用的 Compose 模板文件，默认为 docker-compose.yml，可以多次指定。
-p, --project-name NAME 指定项目名称，默认将使用所在目录名称作为项目名。
–x-networking 使用 Docker 的可拔插网络后端特性
–x-network-driver DRIVER 指定网络后端的驱动，默认为 bridge
–verbose 输出更多调试信息。
-v, --version 打印版本并退出。
```
### config
```
# 验证 Compose 文件格式是否正确，若正确则显示配置，若格式错误显示错误原因。如：
docker-compose -f skywalking.yml config
# 此命令不会执行真正的操作，而是显示 docker-compose 程序解析到的配置文件内容
```
### images
```
# 列出 Compose 文件中包含的镜像。如
docker-compose -f skywalking.yml images
```
### 列出所有运行容器
```
docker-compose ps
```
### 查看服务日志输出
```
docker-compose logs
```
### 打印绑定的公共端口
```
下面命令可以输出 eureka 服务 8761 端口所绑定的公共端口
docker-compose port eureka 8761
```
### 构建或者重新构建服务
```
# 构建（重新构建）项目中的服务容器。如：
docker-compose -f skywalking.yml build
# 一般搭配自定义镜像，比如编写的Dockfile，功能类似于docker build .
```
### 启动指定服务已存在的容器
```
docker-compose start eureka
```
### 停止已运行的服务的容器
```
docker-compose stop eureka
```
### 删除指定服务的容器
```
docker-compose rm eureka
```
### 构建、启动、停止容器
```
docker-compose up
docker-compose down

默认情况，如果服务容器已经存在，docker-compose up 将会尝试停止容器，
然后重新创建（保持使用 volumes-from 挂载的卷），以保证新启动的服务匹配 docker-compose.yml 文件的最新内容。
如果用户不希望容器被停止并重新创建，可以使用 docker-compose up --no-recreate。
这样将只会启动处于停止状态的容器，而忽略已经运行的服务。
如果用户只想重新部署某个服务，可以使用 docker-compose up --no-deps -d <SERVICE_NAME> 来重新创建服务并后台停止旧服务，启动新服务，并不会影响到其所依赖的服务。
此命令有如下选项：
1. -d 在后台运行服务容器。
2. --no-color 不使用颜色来区分不同的服务的控制台输出。
3. --no-deps 不启动服务所链接的容器。
4. --force-recreate 强制重新创建容器，不能与 --no-recreate 同时使用。
5. --no-recreate 如果容器已经存在了，则不重新创建，不能与 --force-recreate 同时使用。
6. --no-build 不自动构建缺失的服务镜像。
7. -t, --timeout TIMEOUT 停止容器时候的超时（默认为 10 秒）。
```
### 通过发送 SIGKILL 信号来停止指定服务的容器
```
docker-compose kill eureka
```
### 下载服务镜像：pull
### 设置指定服务运行容器的个数
```
# 以 service=num 形式指定
docker-compose scale user=3 movie=3
```
### 在一个服务上执行一个命令
```
docker-compose run web bash
```