---
title: Manage Docker Container
date: 2019-09-12 17:46:56
categories:
    - Basic Component
tags:
    - Docker
description: "【Docker 系列学习笔记——Docker容器资源管理】1.Docker资源类型。2.如何查看容器占用资源多少。3.如何管理CPU资源。4.如何管理内存资源"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---


## 容器资源管理
### 资源类型
当我们启动一个容器的时候，它可以使用一些系统资源，这与我们在物理机上启动程序基本是一致的。比如主要的几类：
* CPU
* 内存
* 网络
* I/O
* GPU
### 查看容器占用资源
```
docker stats 容器id
或者稍微简化的命令 docker top 容器id
# docker top $(docker ps -ql) -o pid,c,cmd  

# 可加参数
--no-stream  # docker stats 命令默认是一个持续的动态流式输出（每秒一次），给它传递 --no-stream 参数后，它就只输出一次便会退出了。

# Container ID：容器的 ID，也是一个容器生命周期内不会变更的信息。
# Name：容器的名称，如果没有手动使用 --name 参数指定，则 Docker 会随机生成一个，运行过程中也可以通过命令修改。
# CPU %：容器正在使用的 CPU 资源的百分比，下面会详细说。
# Mem Usage/Limit：当前内存的使用及容器可用的最大内存。
# Mem %：容器正在使用的内存资源的百分比。
# Net I/O：容器通过其网络接口发送和接受到的数据量。
# Block I/O：容器通过块设备读取和写入的数据量。
# Pids：容器创建的进程或线程数。
```
### 管理容器的 CPU 资源
```
docker run --help |grep CPU

# 1. 限制容器占用cpu资源，默认无限制
docker update --cpus "0.5" $(docker ps -ql) # 限制容器只可以使用 0.5 CPU
# 如果是多核，比如八核，则最多可以分配8个cpu，cpu占用率也会显示800%

# 2. 指定可使用 CPU 核
# 可以使用 --cpuset-cpus 来指定分配可使用的 CPU 核，指定 0 表示使用第一个 CPU 核。
docker update --cpus "1.5" --cpuset-cpus 0  $(docker ps -ql)
```
### 管理容器的 内存 资源
```
docker run --help |egrep 'memory|oom'

# OOM——Out Of Memory
# 当内核检测到没有足够的内存来运行系统的某些功能时候，就会触发 OOM 异常，并且会使用 OOM Killer 来杀掉一些进程，腾出空间以保障系统的正常运行。
# 内核中 OOM Killer 的代码，在 torvalds/linux/mm/oom_kill.c 可直接看到
# Docker 在启动的时候默认设置了一个 -500 的 oom_score_adj 以尽可能地避免 Docker 进程本身被 OOM Killer 给杀掉。
# 如果我们想让某个容器，尽可能地不要被 OOM Killer 杀掉，那我们可以给它传递 --oom-score-adj 配置一个比较低的数值。
# 但是注意：不要通过 --oom-kill-disable 禁用掉 OOM Killer，或者给容器设置低于 dockerd 进程的 oom_score_adj 值，这可能会导致某些情况下系统的不稳定。

# 限制内存
docker run --rm -it --memory 10m alpine # --memory 10m 限制其可使用的内存为 10 m
# 查看限制大小
# docker stats查询
docker stats --no-stream $(docker ps -ql)
# 在容器内执行
cat /sys/fs/cgroup/memory/memory.limit_in_bytes 
>>> 10485760
# 在宿主机上执行
cat /sys/fs/cgroup/memory/system.slice/docker-$(docker inspect --format '{{ .Id}}' $(docker ps -ql)).scope/memory.limit_in_bytes
>>> 10485760

# 更新容器可使用的内存
docker update --memory 20m $(docker ps -ql)
```
#### 内存限制参数的特定行为
> 这里的特定参数行为，主要是指我们前面使用的 --memory 和未介绍过的 --memory-swap 这两个参数。

1. --memory 用于限制内存使用量，而 --memory-swap 则表示内存和 Swap 的总和。
--memory-swap 始终应该大于等于 --memory （毕竟 Swap 最小也只能是 0 ）。
2. 如果只指定了 --memory 则最终 --memory-swap 将会设置为 --memory 的两倍。也就是说，在只传递 --memory 的情况下，容器只能使用与 --memory 相同大小的 Swap。
如果创建容器的时候 --memory 10，那么直接对容器update扩大至 20m 的时候能成功，而扩大到 100m 的时候会出错，在上述场景中只指定了 --memory 为 10m，所以 --memory-swap 就默认被设置成了 20m。
3. 如果 --memory-swap 和 --memory 设置了相同值，则表示不使用 Swap。
4. 如果 --memory-swap 设置为 -1 则表示不对容器使用的 Swap 进行限制。
5. 如果设置了 --memory-swap 参数，则必须设置 --memory 参数。
