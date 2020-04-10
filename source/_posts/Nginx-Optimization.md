---
title: Nginx Optimization
date: 2019-10-20 20:00:00
categories:
    - Basic Component
tags:
    - Nginx
description: "【Nginx 系列学习笔记——4】 如何提升 Nginx 的效率，减少服务期负担，增快用户访问速度"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

#### 调整 worker_processes
指Nginx要生成的worker数量，最佳实践是每个CPU运行1个工作进程。
```
# 了解系统中的CPU核心数，输入：
$ grep processor / proc / cpuinfo | wc -l
```
#### 最大化worker_connections
Nginx Web服务器可以同时提供服务的客户端数。与worker_processes结合使用时，获得每秒可以服务的最大客户端数
```
单个进程的连接数（文件描述符 fd）
最大客户端数/秒=工作进程*工作者连接数
为了最大化Nginx的全部潜力，应将工作者连接设置为核心一次可以运行的允许的最大进程数1024。
上面是 Nginx 作为通用服务器时，最大的连接数。
Nginx 作为反向代理服务器时，能够服务的最大连接数要除以2
因为 Nginx 反向代理时，会建立 Client 的连接和后端 Web Server 的连接，占用 2 个连接
```
#### 启用Gzip文件压缩
如果我们租用了一个带宽很低的服务器，网站访问速度会很慢，这时我们可以通过让nginx开启GZIP压缩来提高网站的访问速度。（压缩文件大小，减少了客户端http的传输带宽，因此提高了页面加载速度）
* 首先我们对nginx进行限速操作，限制每个连接的访问速度为128K来建立一个比较慢的访问场景；
* 修改mall.conf配置文件，进行限速操作：
```
server {
    listen       80;
    server_name  mall.macrozheng.com;
    limit_rate 128k; #限制网速为128K

    location / {
        root   /usr/share/nginx/html/mall;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```
* 对mall的前端项目mall.macrozheng.com进行访问，我们可以发现网站中有个js文件比较大，需要加载12s
![](Nginx_3.png)
* nginx返回请求头信息如下：
![](Nginx_4.png)
* 修改/mydata/nginx/conf目录下的nginx.conf配置文件，开启GZIP压缩
```
http {
    gzip on; #开启gzip
    gzip_disable "msie6"; #IE6不使用gzip
    gzip_vary on; #设置为on会在Header里增加 "Vary: Accept-Encoding"
    gzip_proxied any; #代理结果数据的压缩
    gzip_comp_level 6; #gzip压缩比（1~9），越小压缩效果越差，但是越大处理越慢，所以一般取中间值
    gzip_buffers 16 8k; #获取多少内存用于缓存压缩结果
    gzip_http_version 1.1; #识别http协议的版本
    gzip_min_length 1k; #设置允许压缩的页面最小字节数，超过1k的文件会被压缩
    gzip_types application/javascript text/css text/plain application/json; #对特定的MIME类型生效,js和css文件会被压缩

    include /etc/nginx/conf.d/*.conf;
}
```
* 再次对mall的前端项目mall.macrozheng.com进行访问，我们可以发现js文件已经被压缩，加载时间缩短到3.88s，提速3倍左右
![](Nginx_5.png)
* nginx返回请求头中添加了Content-Encoding: gzip的信息
![](Nginx_6.png)
#### 为静态文件启用缓存
为静态文件启用缓存，以减少带宽并提高性能，可以添加下面的命令，限定计算机缓存网页的静态文件：
```
location ~* .(jpg|jpeg|png|gif|ico|css|js)$ {
expires 365d;
}

location ~* .(jpg|jpeg|png|gif|ico|css|js)$ {
add_header Cache-Control no-cache;
add_header Cache-Control private; 
}
```
#### Timeouts
keepalive连接减少了打开和关闭连接所需的CPU和网络开销，获得最佳性能需要调整的变量可参考：
```
client_body_timeout 12;
client_header_timeout 12;
keepalive_timeout 15;
send_timeout 10;
```
#### 禁用access_logs
> 访问日志记录，它记录每个nginx请求，因此消耗了大量CPU资源，从而降低了nginx性能。
```
# 完全禁用访问日志记录
access_log off;

# 如果必须具有访问日志记录，则启用访问日志缓冲
access_log /var/log/nginx/access.log
```
