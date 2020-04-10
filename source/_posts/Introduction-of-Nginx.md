---
title: Introduction of Nginx
date: 2019-10-03 20:00:00
categories:
    - Basic Component
tags:
    - Nginx
description: "【Nginx 系列学习笔记——1】1.比较了另一款重量级web服务器apache。2.介绍了Nginx的原理。3.如何安装Nginx"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

> Nginx: Engine X
## Web 服务器
Nginx 与 Apache一样，是一种WEB服务器。是基于REST架构风格，以统一资源描述符(Uniform Resources Identifier)URI或者统一资源定位符(Uniform Resources Locator)URL作为沟通依据，通过HTTP协议提供各种网络服务。

不同于重量级且不支持高并发的Apache，Nginx具有如下特点：
1. 轻量级
> 功能模块少 - Nginx仅保留了HTTP需要的模块，其他都用插件的方式，后天添加代码模块化 - 更适合二次开发，如阿里巴巴Tengine
2. 高并发
> 使用基于事件驱动架构，使得其可以支持数以百万级别的TCP连接。
3. CPU亲和
> 把CPU核心和Nginx工作进程绑定，把每个worker进程固定在一个CPU上执行，减少切换CPU的cache miss，从而提高性能。

## Nginx 原理
### Nginx进程模型
![](Nginx进程模型.png)
Nginx 服务器，正常运行过程中：
1. 多进程：一个 Master 进程、多个 Worker 进程
2. Master 进程：管理 Worker 进程
3. 对外接口：接收外部的操作（信号）
4. 对内转发：根据外部的操作的不同，通过信号管理 Worker
5. 监控：监控 worker 进程的运行状态，worker 进程异常终止后，自动重启 worker 进程
6. Worker 进程：所有 Worker 进程都是平等的
7. 实际处理：网络请求，由 Worker 进程处理
8. Worker 进程数量：在 nginx.conf 中配置，一般设置为核心数，充分利用 CPU 资源，同时，避免进程数量过多，避免进程竞争 CPU 资源，增加上下文切换的损耗。

### HTTP 连接建立和请求处理过程
1. Nginx 启动时，Master 进程，加载配置文件
2. Master 进程，初始化监听的 socket
3. Master 进程，fork 出多个 Worker 进程
4. Worker 进程，竞争新的连接，获胜方通过三次握手，建立 Socket 连接，并处理请求

### 高性能、高并发
> 异步，非阻塞，使用了IO 多路复用 epoll 和大量的底层代码优化。
> 单个连接的请求处理速度没有优势，适合 IO 密集型 场景，事件驱动

如果一个server采用一个进程负责一个request的方式，那么进程数就是并发数。正常情况下，会有很多进程一直在等待中。
而nginx采用一个master进程，多个woker进程的模式。
master进程主要负责收集、分发请求。每当一个请求过来时，master就拉起一个worker进程负责处理这个请求。
同时master进程也负责监控woker的状态，保证高可靠性。
woker进程一般设置为跟cpu核心数一致。nginx的woker进程在同一时间可以处理的请求数只受内存限制，可以处理多个请求。

Nginx 的异步非阻塞工作方式正把当中的等待时间利用起来了。在需要等待的时候，这些进程就空闲出来待命了，因此表现为少数几个进程就解决了大量的并发问题。

每进来一个request，会有一个worker进程去处理。但不是全程的处理，只处理到可能发生阻塞的地方，比如向上游(后端)服务器转发request，并等待请求返回。那么，这个处理的worker会在发送完请求后，注册一个事件：“如果upstream返回了，告诉我一声，我再接着干”。于是他就休息去了。此时，如果再有request 进来，他就可以很快再按这种方式处理。而一旦上游服务器返回了，就会触发这个事件，worker才会来接手，这个request才会接着往下走。

所以不同于Apache: 创建多个进程或线程，而每个进程或线程都会为其分配 cpu 和内存(线程要比进程小的多，所以worker支持比perfork高的并发)，并发过大会耗光服务器资源。

Nginx: 采用单线程来异步非阻塞处理请求(管理员可以配置Nginx主进程的工作进程的数量)(epoll)，不会为每个请求分配cpu和内存资源，节省了大量资源，同时也减少了大量的CPU的上下文切换。所以才使得Nginx支持更高的并发。
### Nginx事件处理模型
![](Nginx事件处理模型.png)
### 模块化体系结构
![](Nginx模块化体系结构.png)
nginx的模块根据其功能基本上可以分为以下几种类型：
#### event module
搭建了独立于操作系统的事件处理机制的框架，及提供了各具体事件的处理。包括ngx_events_module， ngx_event_core_module和ngx_epoll_module等。nginx具体使用何种事件处理模块，这依赖于具体的操作系统和编译选项。
#### phase handler
此类型的模块也被直接称为handler模块。主要负责处理客户端请求并产生待响应内容，比如ngx_http_static_module模块，负责客户端的静态页面请求处理并将对应的磁盘文件准备为响应内容输出。
#### output filter
也称为filter模块，主要是负责对输出的内容进行处理，可以对输出进行修改。例如，可以实现对输出的所有html页面增加预定义的footbar一类的工作，或者对输出的图片的URL进行替换之类的工作。
#### upstream
upstream模块实现反向代理的功能，将真正的请求转发到后端服务器上，并从后端服务器上读取响应，发回客户端。upstream模块是一种特殊的handler，只不过响应内容不是真正由自己产生的，而是从后端服务器上读取的。
#### load-balancer
负载均衡模块，实现特定的算法，在众多的后端服务器中，选择一个服务器出来作为某个请求的转发服务器。

## Nginx 安装
### yum 安装
```
$ yum install epel-release
$ yum install nginx

启停命令
$ systemctl enable nginx
$ systemctl start nginx
$ systemctl restart nginx
$ systemctl reload nginx
浏览器访问IP确认是否开启成功

配置文件默认位置
/etc/nginx/nginx.conf
```

### Docker 安装
```
$ docker pull nginx:1.10

为了从容器中拷贝nginx配置，首先运行一次容器：
docker run -p 80:80 --name nginx \
-v /mydata/nginx/html:/usr/share/nginx/html \
-v /mydata/nginx/logs:/var/log/nginx  \
-d nginx:1.10

将容器内的配置文件拷贝到指定目录：
docker container cp nginx:/etc/nginx /mydata/nginx/

修改文件名称：
mv nginx conf

终止并删除容器：
docker stop nginx
docker rm nginx

使用docker命令启动
docker run -p 80:80 --name nginx \
-v /mydata/nginx/html:/usr/share/nginx/html \
-v /mydata/nginx/logs:/var/log/nginx  \
-v /mydata/nginx/conf:/etc/nginx \
-d nginx:1.10
```
