---
title: Docker Introduction
date: 2019-08-10 20:00:00
categories:
    - Basic Component
tags:
    - Docker
description: "【Docker 系列学习笔记——Docker入门】1.什么是docker，docker的原理，docker的用途。2.如何安装docker"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---


> [Docker文档](https://docs.docker.com/) | [DockerHub](https://hub.docker.com/)

## Docker 原理
> 所谓的镜像，就是持久化后的，安装了各种工具，软件和服务的一个Linux 操作系统。
> Docker 属于 Linux 容器的一种封装，提供简单易用的容器使用接口。
### Docker 特点
1. 文件系统隔离：每个不同的容器都存在不同的独立文件系统中
2. 资源隔离：每个容器都有自己独立的ip，网络和虚拟接口
3. 日志记录：Docker将会收集和记录每个进程容器的标准流（stdout/stderr/stdin），用于实时检索和批量检索

### 三类主要用途

（1）提供一次性的环境。比如，本地测试他人的软件、持续集成的时候提供单元测试和构建的环境。
（2）提供弹性的云服务。因为 Docker 容器可以随开随关，很适合动态扩容和缩容。
（3）组建微服务架构。通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构。

### Docker 术语
仓库： 别人做好的现成的镜像，都放在仓库里
镜像： 自己要用哪个镜像，就先拉到本地来。镜像就相当于还没激活的容器。
> Docker 把应用程序及其依赖，打包在 image 文件里面。只有通过这个文件，才能生成 Docker 容器。image 文件可以看作是容器的模板。Docker 根据 image 文件生成容器的实例。同一个 image 文件，可以生成多个同时运行的容器实例。
> image 是二进制文件。实际开发中，一个 image 文件往往通过继承另一个 image 文件，加上一些个性化设置而生成。

容器： 容器就是跑起来的镜像，就是一个完整的工作环境
> image 文件生成的容器实例，本身也是一个文件，称为容器文件。也就是说，一旦容器生成，就会同时存在两个文件： image 文件和容器文件。而且关闭容器并不会删除容器文件，只是容器停止运行而已。

## 安装 Docker
### Centos yum 安装
```
# 1. 移除旧版本docker
yum remove docker \
    docker-client \
    docker-client-latest \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-selinux \
    docker-engine-selinux \
    docker-engine

# 2. 更新yum
yum -y update

# 3. 安装常用工具
yum install iproute ftp bind-utils net-tools wget yum-utils device-mapper-persistent-data lvm2 -y

# 4. 安装Docker
yum install docker -y

# 5. 查看是否安装成功
docker -v
docker info
```
### 启停Docker
```
systemctl start docker.service
systemctl status docker.service
systemctl stop docker.service
systemctl restart docker.service
```
### 虚拟机里安装Docker注意事项
```
运行
nslookup www.baidu.com
若显示局域网地址，则代表需要进行之后步骤
```
```
把它改为公用的域名服务器地址
vi /etc/resolv.conf

# Generated by NetworkManager
search www.tendawifi.com
nameserver 119.29.29.29
nameserver 182.254.116.116
```
### 配置镜像加速器
```
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://hvmf8r55.mirror.aliyuncs.com"]
}
EOF
systemctl daemon-reload
systemctl restart docker
查看 daemon.json 是否已经生效了
tail /etc/docker/daemon.json
```
### 网络恢复
虚拟机一旦重启，或者关机过，那么就会出现可以访问 Linux，但是无法访问里面的 Docker的情况。可以在 CentOS里做如下事情：
```
vi /etc/sysctl.conf
或者
vi /usr/lib/sysctl.d/00-system.conf
添加如下代码：
net.ipv4.ip_forward=1
重启network服务
systemctl restart network
查看是否修改成功
sysctl net.ipv4.ip_forward
如果返回为“net.ipv4.ip_forward = 1”则表示成功了
```