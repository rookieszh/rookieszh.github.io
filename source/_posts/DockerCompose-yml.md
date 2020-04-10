---
title: DockerCompose.yml
date: 2019-09-16 18:56:57
categories:
    - Basic Component
tags:
    - Docker
description: "【Docker 系列学习笔记——DockerCompose模板文件】介绍了DockerCompose模板文件的属性，并实战举例"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

## docker-compose.yml 模板文件属性
> 默认的模板文件名称为 docker-compose.yml，格式为 YAML 格式。
###### version
```
指定 docker-compose.yml 文件的写法格式
```
###### services
```
多个容器集合
```
###### build
```
配置构建时，Compose 会利用它自动构建镜像
该值可以是一个路径，也可以是一个对象，用于指定 Dockerfile 参数

build: ./dir
---------------
build:
    context: ./dir
    dockerfile: Dockerfile
    args:
        buildno: 1
```
###### command
```
覆盖容器启动后默认执行的命令
command: bundle exec thin -p 3000
----------------------------------
command: [bundle,exec,thin,-p,3000]
```
###### dns
```
配置 dns 服务器，可以是一个值或列表
dns: 8.8.8.8
------------
dns:
    - 8.8.8.8
    - 9.9.9.9
```
###### dns_search
```
配置 DNS 搜索域，可以是一个值或列表
dns_search: example.com
------------------------
dns_search:
    - dc1.example.com
    - dc2.example.com
```
###### environment
```
环境变量配置，可以用数组或字典两种方式
environment:
    RACK_ENV: development
    SHOW: 'ture'
-------------------------
environment:
    - RACK_ENV=development
    - SHOW=ture
```
###### env_file
```
从文件中获取环境变量，可以指定一个文件路径或路径列表，其优先级低于 environment 指定的环境变量
env_file: .env
---------------
env_file:
    - ./common.env
```
###### expose
```
暴露端口，只将端口暴露给连接的服务，而不暴露给主机
expose:
    - "3000"
    - "8000"
```
###### image
```
指定服务所使用的镜像或镜像 ID。如果镜像在本地不存在，Compose 将会尝试拉取这个镜像。
image: java
```
###### network_mode
```
设置网络模式
network_mode: "bridge"
network_mode: "host"
network_mode: "none"
network_mode: "service:[service name]"
network_mode: "container:[container name/id]"
```
###### ports
```
对外暴露的端口定义，和 expose 对应
ports:   # 暴露端口信息  - "宿主机端口:容器暴露端口"
- "8763:8763"
- "8763:8763"
```
###### links
```
将指定容器连接到当前连接，可以设置别名，避免ip方式导致的容器重启动态改变的无法连接情况
links:    # 指定服务名称:别名
    - docker-compose-eureka-server:compose-eureka
```
###### volumes
```
数据卷所挂载路径设置。可以设置为宿主机路径(HOST:CONTAINER)或者数据卷名称(VOLUME:CONTAINER)，并且可以设置访问模式 （HOST:CONTAINER:ro）。
volumes:
      - /app/skywalking/elasticsearch/data:/usr/share/elasticsearch/data:rw
      - conf/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
```
###### logs
```
日志输出信息
--no-color          单色输出，不显示其他颜.
-f, --follow        跟踪日志输出，就是可以实时查看日志
-t, --timestamps    显示时间戳
--tail              从日志的结尾显示，--tail=200
```
######  ulimits
```
指定容器的 ulimits 限制值。
例如，指定最大进程数为 65535，指定文件句柄数为 20000（软限制，应用可以随时修改，不能超过硬限制） 
和 40000（系统硬限制，只能 root 用户提高）。

ulimits:
   nproc: 65535
   nofile:
     soft: 20000
     hard: 40000
```
###### depends_on
```
解决容器的依赖、启动先后的问题。以下例子中会先启动 redis mysql 再启动 web

version: '3'
services:
  web:
    build: .
    depends_on:
      - db
      - redis      
  redis:
    image: redis    
  db:
    image: mysql
```
###### restart
```
指定容器退出后的重启策略为始终重启。
该命令对保持服务始终运行十分有效，在生产环境中推荐配置为 always 或者 unless-stopped。

restart: always
```
## 实战举例
```
version: '3.3'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.8.5
    container_name: elasticsearch
    restart: always
    ports:
      - 9200:9200
      - 9300:9300
    environment:
      discovery.type: single-node
    ulimits:
      memlock:
        soft: -1
        hard: -1
  oap:
    image: skywalking/oap
    container_name: oap
    depends_on:
      - elasticsearch
    links:
      - elasticsearch
    restart: always
    ports:
      - 11800:11800
      - 12800:12800
    environment:
      SW_STORAGE: elasticsearch
      SW_STORAGE_ES_CLUSTER_NODES: elasticsearch:9200
  ui:
    image: skywalking/ui
    container_name: ui
    depends_on:
      - oap
    links:
      - oap
    restart: always
    ports:
      - 8080:8080
    environment:
      SW_OAP_ADDRESS: oap:12800
```