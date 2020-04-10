---
title: What Can Nginx Do and How to Use Nginx
date: 2019-10-05 20:00:00
categories:
    - Basic Component
tags:
    - Nginx
description: "【Nginx 系列学习笔记——2】 通过实战介绍了Nginx作为Http服务器的使用：正向代理、反向代理、邮件服务代理。重点：负载均衡、静态代理、动态代理"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

### 正向代理服务器
被代理角色通过代理访问目标角色完成一些任务的过程称为代理操作过程。
> 正向代理最大的特点是客户端非常明确要访问的服务器地址；服务器只清楚请求来自哪个代理服务器，而不清楚来自哪个具体的客户端；正向代理模式屏蔽或者隐藏了真实客户端信息。
#### 正向代理的作用
（1）访问原来无法访问的资源，如Google，或者局域网中的客户端不能直接访问Internet，则需要通过代理服务器来访问
（2）可以做缓存，加速访问资源
（3）对客户端访问授权，上网进行认证
（4）代理可以记录用户访问记录（上网行为管理），对外隐藏用户信息
#### 使用示例
```
# 代理 http
server {
  resolver 192.168.34.3; # 指定DNS服务器IP地址
  listen 2020; # 供请求代理的客户端访问的端口
  location / {
    proxy_pass http://$http_host$request_uri; # 设定代理服务器的协议和地址
    proxy_set_header HOST $http_host;
    proxy_buffers 256 4k;
    proxy_max_temp_file_size 0k;
    proxy_connect_timeout 30;
    proxy_send_timeout 60;
    proxy_read_timeout 60;
    proxy_next_upstream error timeout invalid_header http_502;
  }
}

# 代理 https
server {
  resolver 192.168.34.3; # 指定DNS服务器IP地址务器IP地址
  listen 2021; # 供请求代理的客户端访问的端口
  location / {
    proxy_pass https://$http_host$request_uri; #设定代理服务器的协议和地址
    proxy_set_header HOST $http_host;
    proxy_buffers 256 4k;
    proxy_max_temp_file_size 0k;
    proxy_connect_timeout 30;
    proxy_send_timeout 60;
    proxy_read_timeout 60;
    proxy_next_upstream error timeout invalid_header http_502;
  }
}
```
```
# 访问 http
curl -l --proxy 192.168.43.144:2020 http://www.baidu.com

# 访问 https
curl -l --proxy 192.168.43.144:2021 http://www.baidu.com

# 甚至还可以在服务器上请求nginx正向代理 本服务器上被nginx反向代理的服务
curl -l --proxy 192.168.43.144:2020 192.168.43.144:10003/console/service/1/1/get-sys-param-info
curl -l --proxy 192.168.43.144:2021 192.168.43.144:10010/console/service/1/1/get-sys-param-info
```
### 反向代理服务器
> 反向代理就是当请求访问你的代理服务器时，代理服务器会对你的请求进行转发，可以转发到静态的资源路径上去，也可以转发到动态的服务接口上去。
> [淘宝基于nginx改进的tengine](http://tengine.taobao.org/)
#### 反向代理的作用
（1）保证内网的安全，通常将反向代理作为公网访问地址，Web服务器是内网。
（2）负载均衡，通过反向代理服务器来优化网站的负载。
#### 负载均衡
> 将服务器接收到的请求按照规则分发的过程，称为负载均衡。
##### 使用示例
```
首先在http下增加一个upstream ，用来指向各个服务器节点
    upstream weixin_8111_8222{
	          server	127.0.0.1:8111 weight=1;
	          server	xxxxxxxxx:8222 weight=2;
    }

然后修改：
        location / {
            proxy_pass http://weixin_8111_8222;
        }
```

##### Nginx支持的负载均衡调度算法
1. weight轮询(默认)：接收到的请求按照顺序逐一分配到不同的后端服务器，即使在使用过程中，某一台后端服务器宕机，Nginx会自动将该服务器剔除出队列，请求受理情况不会受到任何影响。这种方式下，可以给不同的后端服务器设置一个权重值(weight)，用于调整不同的服务器上请求的分配率；权重数据越大，被分配到请求的几率越大；该权重值，主要是针对实际工作环境中不同的后端服务器硬件配置进行调整的。
2. ip_hash：每个请求按照发起客户端的ip的hash结果进行匹配，这样的算法下一个固定ip地址的客户端总会访问到同一个后端服务器，这也在一定程度上解决了集群部署环境下session共享的问题。
3. fair：智能调整调度算法，动态的根据后端服务器的请求处理到响应的时间进行均衡分配，响应时间短处理效率高的服务器分配到请求的概率高，响应时间长处理效率低的服务器分配到的请求少；结合了前两者的优点的一种调度算法。但是需要注意的是Nginx默认不支持fair算法，如果要使用这种调度算法，请安装upstream_fair模块。
4. url_hash：按照访问的url的hash结果分配请求，每个请求的url会指向后端固定的某个服务器，可以在Nginx作为静态服务器的情况下提高缓存效率。同样要注意Nginx默认不支持这种调度算法，要使用的话需要安装Nginx的hash软件包。

##### session 问题
通过负载均衡课程，我们可以把请求分发到不同的 Tomcat 来缓解服务器的压力，但是这里存在一个问题： 当同一个用户第一次访问tomcat_8111 并且登录成功， 而第二次访问却被分配到了tomcat_8222， 这里并没有记录他的登陆状态，那么就会呈现未登录状态了，严重伤害了用户体验。
1. 解决办法一: ip_hash
```
# 通过ip地址标记用户，如果多次请求都是从同一个ip来的，那么就都分配到同一个tomcat
upstream weixin_8111_8222{
        server	127.0.0.1:8111 backup;   # backup是指主节点全部宕机了才启用此节点
        server	xxxxxxxxx:8222 weight=2;
        ip_hash;
}
```
2. 解决办法二： redis+tomcat-sessoin-manager
```
Tomcat需要链接 redis，所以需要专门的jar包，一共有3个jar包：
jedis-2.5.2.jar，
commons-pool2-2.0.jar，
tomcat-redis-session-manager1.2.jar。
然后修改tomcat/conf/context.xml ，增加下面这坨东西

<Valve className="com.orangefunction.tomcat.redissessions.RedisSessionHandlerValve" />
<Manager className="com.orangefunction.tomcat.redissessions.RedisSessionManager"
 host="127.0.0.1"
 port="6379"
 database="0"
 maxInactiveInterval="60" />
```
#### 静态代理
```
# 1. 首先修改下本机的host文件，例如：
192.168.6.132 docs.macrozheng.com
192.168.6.132 mall.macrozheng.com

# 2. 在/mydata/nginx/conf/conf.d文件夹中添加配置文件docs.conf对文档项目进行反向代理：
server {
    listen       80;
    server_name  docs.macrozheng.com; #修改域名

    location / {
        root   /usr/share/nginx/html/docs; #代理到docs文件夹中
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

# 3. 在/mydata/nginx/conf/conf.d文件夹中添加配置文件mall.conf对mall的前端项目进行反向代理：
server {
    listen       80;
    server_name  mall.macrozheng.com; #修改域名

    location / {
        root   /usr/share/nginx/html/mall; #代理到mall文件夹中
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

# 重启nginx服务：
docker restart nginx
```
```
# 一个server代理多个url

location / {
  root  /home/wwwroot/graduation;
  index index.html;
}

location /mikutap {
  alias  /home/wwwroot/mikutap/;
  index index.html;
}

location /rpremoval {
  alias  /home/wwwroot/realtime-person-removal/;
  index index.html;
}
```
#### 动态代理
```
# 1. 首先修改下本机的host文件，例如：
192.168.6.132 api.macrozheng.com

# 2. 在/mydata/nginx/conf/conf.d文件夹中添加配置文件api.conf对将请求代理到远程的mall-admin服务上去：
server {
    listen       80;
    server_name  api.macrozheng.com; #修改域名

    location / {
    	proxy_pass   http://120.27.63.9:8080; #修改为代理服务地址
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```
#### 地址重写
有的时候我们的网站更换了域名，但还有用户在使用老的域名访问，这时可以通过nginx的地址重写来让用户跳转到新的域名进行访问。

比如说原来用的docs.macrozheng.com这个域名不用了，现在改成www.macrozheng.com了来访问文档项目了；修改docs.conf配置文件，将地址带参数重写到新地址：
```
server {
    listen       80;
    server_name  docs.macrozheng.com;

    rewrite "^/(.*)$" http://www.macrozheng.com/$1; #地址重写到新地址

    location / {
        root   /usr/share/nginx/html/docs;
        index  index.html index.htm;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```
#### 一个IP绑定多个域名
```
匹配顺序:
server_name与host匹配优先级如下：
1、完全匹配
2、通配符在前的，如*.test.com
3、在后的，如www.test.*
4、正则匹配，如~^\.www\.test\.com$

如果都不匹配:
1、优先选择listen配置项后有default或default_server的
2、找到匹配listen端口的第一个server块

server {
	listen 80;
	server_name www;
}
server {
	listen  80;
	server_name www.zkh.com;
}
...
```
#### 反向代理修改url
```
比如对外接口是 192.168.43.144:10050/uuatest/uivs/1/1/user/register
服务实际接口是 192.168.43.144:10050/UUA/uivs/1/1/user/register

/uuatest/ ：uuatest 不传到后台服务
/uuatest  ：uuatest 传到后台服务

则如下：
server{
  listen 10050;
  server_name 192.168.43.144;
  location /uuatest/ {
    proxy_pass http://uua_service_8081/UUA/;
    proxy_set_header        X-Real-IP $remote_addr;
    proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header        Host $http_host;
    proxy_set_header        urlprefix https;
  }
}
```
### IMAP、POP3、SMTP代理服务器