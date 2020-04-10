---
title: How To Use Docker
date: 2019-08-11 20:00:00
categories:
    - Basic Component
tags:
    - Docker
description: "【Docker 系列学习笔记——使用Docker】本文介绍了最常用的docker指令与用法"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

###### help
```
# 比如：
docker run --help
```
###### login
```
# docker后台登陆
# 注意，账号不是邮箱地址
docker login
```
###### search
```
#  查看仓库里有些什么镜像
docker search jenkins

# 默认从docker hub搜寻，镜像名称前面会默认加上 docker.io/
# 当然也可以找其他常见的，如 mysql, tomcat, nginx 等等。

# NAME：镜像的仓库名
# DESCRIPTION：镜像描述
# STARS：用户评价，即用户欢迎程度
# OFFICIAL：是否官方
```
###### pull
```
# 拉取镜像到本地，如果不加版本默认会下载latest版本
docker pull 镜像名:版本
```
###### push
```
# 推送镜像到hub
docker push 镜像名:版本
```
###### run 与 参数
```
# 创建容器
docker run -dit --rm --privileged=true\ 
-p21:21 -p9010:9010 -p8080:8080 -p30000-30010:30000-30010\
--name how2jtmall how2j/tmall:latest /usr/sbin/init

# 1. docker run 表示运行一个镜像
# 2. -dit 是 -d -i -t 的缩写。 -d ，表示 detach，即在后台运行。 -i 表示提供交互接口，这样才可以通过 docker 和 跑起来的操作系统交互。-t 表示提供一个 tty (伪终端)，与 -i 配合就可以通过 ssh 工具连接到这个容器里面去了
# 3. --rm 表示在容器终止运行后自动删除容器文件，或者如果容器已经存在了，自动删除容器
# 4. --privileged 启动容器的时候，把权限带进去。这样才可以在容器里进行完整的操作
# 5. -p21:21 第一个21，表示在CentOS上开放21端口。第二个21表示在容器里开放21端口。 这样当访问CentOS 的21端口的时候，就会间接地访问到容器里了。
# 6. -p9010:9010 和 21一个道理
# 7. -p8080:8080 和 21 一个道理，tomcat端口
# 8. -p30000-30010 和21也是一个道理，这个是ftp用来传输数据的
# 9. --name how2jtmall 给容器取了个名字，叫做 how2jtmall，方便后续管理
# 10. how2j/tmall:latest how2j/tmall就是镜像的名称， latest是版本号，即最新版本
# 11. /usr/sbin/init: 表示启动后运行的程序，即通过这个命令做初始化
# 12. -v 表示目录映射关系（前者是宿主机目录，后者是映射到宿主机上的目录），可以使用多个－v做多个目录或文件映射。
# 13. -e 表示添加容器的环境变量
```
###### exec
```
# 进入容器，进去之后，就可以像操作一个普通 linux 那样操作了
docker exec -it how2jtmall /bin/bash
# 输入 exit 退出容器
```
###### images 
```
# 查看本地有些什么镜像
docker images
# REPOSITORY：镜像名称
# TAG：镜像标签
# IMAGE ID：镜像ID
# CREATED：镜像的创建日期（不是获取该镜像的日期）
# SIZE：镜像大小
```
###### rmi 
```
# 删除本地镜像
docker rmi docker.io/tomcat:8.0
# 有容器的情况下需要先删除容器
# 如果需要一次性删除全部镜像
docker rmi $(docker images -q)
```
###### commit
```
# 对容器做了修改后，把改动后的容器，再次转换为镜像。
docker commit 容器名 保存的镜像名
```
###### 生命周期管理
```
暂停：pause
恢复：unpause
停止：stop
开始：start
```
###### ps
```
docker ps     查看正在运行的容器
docker ps –a  查看所有的容器
docker ps -a --filter status=dead --filter status=exited --last 1 # 找出最近运行失败的Docker容器
```
###### inspect
```
# 检查容器具体内容
docker inspect 容器名或id

# 该命令获取两个主要响应：镜像级别的详细信息和容器级别的详细信息。
# 1. 容器ID以及创建的时间戳
# 2. 当前状态（在尝试识别容器是否已停止以及为何停止时很有用）
# 3. Docker镜像信息、文件系统绑定、卷信息以及挂载
# 4. 环境变量，例如传递给容器的命令行参数
# 5. 网络配置：IPv4和IPv6的IP地址以及网关和辅助地址
```
###### rm 
```
# 删除容器
docker rm 容器名/id
# 删除所有容器
docker rm -f $(docker ps -a -q)
```
###### tag
```
# 通过tag可以对镜像进行标记
docker tag [imageName] [username]/[repository]:[tag]
docker tag docker.io/tomcat:8.0 docker.io/mytomcat:8.0
# 然后docker images，会发现上面两个镜像都拥有同样的imageid
```
###### cp
```
# 拷贝文件
docker cp 需要拷贝的文件或目录容器名称:容器目录->把宿主机的文件拷贝到容器里
docker cp 容器名称:容器目录需要拷贝的文件或目录->从容器中拷贝文件到宿主机
```
###### save & load
```
# 备份：备份的作用在于保留一个版本的镜像。
docker save –o 打包的后的文件名.tar 镜像名
# 恢复：假如我们对修改运行一段时间后的容器不满意，想要回到最初的状态，这时我们就可以删除现在的容器和镜像，然后恢复备份的镜像，直接run。
docker rm 你要删除的镜像名或者id
docker load –i 你的备份镜像tar包
```
###### logs
```
# 查看容器运行日志
docker logs 容器id
```
###### history
```
# 查看镜像build历史
docker history 镜像id
```
