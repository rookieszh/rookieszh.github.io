---
title: DockerFile
date: 2019-08-12 17:46:56
categories:
    - Basic Component
tags:
    - Docker
description: "【Docker 系列学习笔记——DockerFile】1.什么是dockerfile，特点是什么。2.docker镜像创建指令。3.dockerfile里的命令详解"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

> Dockerfile的本质就是由一系列顺序编排的命令和参数组成的脚本，并利用它把我们需要的应用制成一个镜像文件

## 特点
1、对于开发人员：可以为开发团队提供完全一致的开发环境；
2、对于测试人员：可以直接用开发所制成的镜像或者通过Dockerfile文件构建一个新的镜像开始工作了；
3、对于运维人员：在部署时，可以实现应用的无缝移植。
## .dockerignore
```
.git
node_modules
npm-debug.log
# 上面代码表示，这三个路径要排除，不要打包进入 image 文件。如果没有路径要排除，这个文件可以不新建。
```
## 创建指令
```
docker build [OPTIONS] PATH | URL | -

--build-arg=[] :设置镜像创建时的变量；
--cpu-shares :设置 cpu 使用权重；
--cpu-period :限制 CPU CFS周期；
--cpu-quota :限制 CPU CFS配额；
--cpuset-cpus :指定使用的CPU id；
--cpuset-mems :指定使用的内存 id；
--disable-content-trust :忽略校验，默认开启；
-f :指定要使用的Dockerfile路径；
--force-rm :设置镜像过程中删除中间容器；
--isolation :使用容器隔离技术；
--label=[] :设置镜像使用的元数据；
-m :设置内存最大值；
--memory-swap :设置Swap的最大值为内存+swap，"-1"表示不限swap；
--no-cache :创建镜像的过程不使用缓存；
--pull :尝试去更新镜像的新版本；
--quiet, -q :安静模式，成功后只输出镜像 ID；
--rm :设置镜像成功后删除中间容器；
--shm-size :设置/dev/shm的大小，默认值是64M；
--ulimit :Ulimit配置。
--tag, -t: 镜像的名字及标签，通常 name:tag 或者 name 格式；可以在一次构建中为一个镜像设置多个标签。
--network: 默认 default。在构建期间设置RUN指令的网络模式
```
```
举例：
1. 使用当前目录的 Dockerfile 创建镜像，标签为 runoob/ubuntu:v1
docker build -t runoob/ubuntu:v1 .

2. 使用URL github.com/creack/docker-firefox 的 Dockerfile 创建镜像。
docker build github.com/creack/docker-firefox

3. 也可以通过 -f Dockerfile 文件的位置：
docker build -f /path/to/a/Dockerfile .
```
## 命令详解
### FROM
```
指定哪种镜像作为新镜像的基础镜像，如：
FROM ubuntu:14.04
```
### MAINTAINER
```
指明该镜像的作者和其电子邮件，如：
MAINTAINER vector4wang "xxxxxxx@qq.com"
```
### ENV
```
设置环境变量 (可以写多条)
ENV key value
```
### RUN
```
在新镜像内部执行的命令，比如安装一些软件、配置一些基础环境，可使用\来换行，如：
RUN echo 'hello docker!' \
    > /usr/local/file.txt

也可以使用exec格式RUN ["executable", "param1", "param2"]的命令，如：
RUN ["apt-get","install","-y","nginx"]
```
### COPY
```
将主机的文件复制到镜像内，如果目的位置不存在，Docker会自动创建所有需要的目录结构：
COPY source_dir/file dest_dir/file
注意：
1. 需要复制的目录一定要放在Dockerfile文件的同级目录下
2. 因为构建环境将会上传到Docker守护进程，而复制是在Docker守护进程中进行的。任何位于构建环境之外的东西都是不可用的。COPY指令的目的的位置则必须是容器内部的一个绝对路径。
3. 如果有压缩文件并不能解压
```
### ADD
```
将主机的文件复制到镜像中，跟COPY一样，限制条件和使用方式都一样，如：
ADD source_dir/file dest_dir/file
ADD会对压缩文件（tar, gzip, bzip2, etc）做提取和解压操作。
```
### EXPOSE
```
暴露镜像的端口供主机做映射，启动镜像时，使用-P参数来讲镜像端口与宿主机的随机端口做映射。可指定多个：
EXPOSE 8080
EXPOSE 8081
...
```
### WORKDIR
```
在构建镜像时，指定镜像的工作目录，之后的命令都是基于此工作目录，如果不存在，则会创建目录。如：
WORKDIR /usr/local
WORKDIR webservice
RUN echo 'hello docker' > text.txt
...
最终会在/usr/local/webservice/目录下生成text.txt文件
```
### ONBUILD
```
当一个包含ONBUILD命令的镜像被用作其他镜像的基础镜像时，
(比如用户的镜像需要从某为准备好的位置添加源代码，或者用户需要执行特定于构建镜像的环境的构建脚本)，该命令就会执行。

如创建镜像image-A：
FROM ubuntu
...
ONBUILD ADD . /var/www
...

然后创建镜像image-B，指定image-A为基础镜像，如：
FROM image-A
...

然后在构建image-B的时候，日志上显示如下:
Step 0 : FROM image-A
# Execting 1 build triggers
Step onbuild-0 : ADD . /var/www
```
### USER
```
指定该镜像以什么样的用户去执行，如：
USER mongo
```
### VOLUME
```
用来向基于镜像创建的容器添加卷。比如你可以将mongodb镜像中存储数据的data文件指定为主机的某个文件。(容器内部建议不要存储任何数据)
如：
VOLUME /data/db /data/configdb
注意:VOLUME 主机目录 容器目录
```
注意：一般直接 VOLUME 容器目录
因为挂载会出现问题
然后docker启动容器的时候再加上 -v 参数以映射，详见docker.md
### CMD 与 ENTRYPOINT
```
CMD与ENTRYPOINT作用与用法一样：
都是在容器启动时需要执行的命令，如：
CMD /bin/bash
ENTRYPOINT /bin/bash

同样可以使用exec语法，如：
CMD ["/bin/bash"]
当有多个CMD的时候，只有最后一个生效。

但两者拥有很大的区别：
1. CMD的命令会被 docker run 的命令覆盖而ENTRYPOINT不会
> 如使用CMD ["/bin/bash"]或ENTRYPOINT ["/bin/bash"]后，再使用docker run -ti image启动容器，如同使用：
docker run -ti image /bin/bash
/bin/bash：容器启动以后，内部第一个执行的命令。这里是启动 Bash，保证用户可以使用 Shell。

但是如果启动镜像的命令为docker run -ti image /bin/ps，使用CMD后面的命令就会被覆盖转而执行bin/ps命令，而ENTRYPOINT的则不会，而是会把 docker run 后面的命令当做ENTRYPOINT<font color="FF0000">执行命令的参数</font>。

以下例子比较容易理解
Dockerfile中
ENTRYPOINT ["/user/sbin/nginx"]

然后通过启动build之后的容器
docker run -ti image -g "daemon off"

此时-g "daemon off"会被当成参数传递给ENTRYPOINT，最终的命令变成了
/user/sbin/nginx -g "daemon off"

CMD和ENTRYPOINT都存在时，CMD的指令变成了ENTRYPOINT的参数，并且此CMD提供的参数会被 docker run 后面的命令覆盖，如：

...
ENTRYPOINT ["echo","hello","i am"]
CMD ["docker"]
之后启动构建之后的容器

使用docker run -ti image
输出“hello i am docker”

使用docker run -ti image world
输出“hello i am world”
```
