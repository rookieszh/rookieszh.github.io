---
title: Nginx Error
date: 2019-10-29 20:00:00
categories:
    - Basic Component
tags:
    - Nginx
description: "【Nginx 系列学习笔记——5】 Nginx 返回的 Http 报错码以及原因"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

## 502报错
### FastCGI相关
1. FastCGI进程是否已经启动
2. FastCGI worker进程数是否不够
3. FastCGI执行时间过长
4. FastCGI Buffer不够
```
nginx和apache一样，有前端缓冲限制，可以调整缓冲参数
fastcgi_buffer_size 32k;
fastcgi_buffers 8 32k;
```
### Proxy Buffer不够
```
如果你用了Proxying，调整
proxy_buffer_size 16k;
proxy_buffers 4 16k;
```
### php脚本执行时间过长
```
将php-fpm.conf的
<value name="request_terminate_timeout">0s</value>
0s改成一个时间
```