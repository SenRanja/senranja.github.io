---
title: docker-compose调试和操作
date: 2024-04-09 17:58:30
tags:
categories:
- Docker
---


# 方便调试的技巧

    docker-compose down && docker-compose rebuild && docker-compose up -d

# docker-compose操作

    docker-compose build 构建镜像，镜像文件会存在docker images中
    docker-compose up -d 启动容器
    docker-compose down 移除容器（这点我以前以为down是类似于docker stop的命令，没想到docker-compose down是移除的意思，导致删库）
    docker-compose stop 暂停容器运行

# 导出导入操作

参考链接:https://www.hangge.com/blog/cache/detail_2411.html

不过需要注意一下，导入导出的时候最好根据`docker ps -a`的 名字来操作，不要根据 container id 来操作。否则导入的时候，显示的名字可能是 <none>。这个时候虽然容器可以正常运行，但是很多人会感觉<none>是错误的。


最重要的注意，**docker-compose无法构建容器为镜像，而且也不会复制外部目录到内部**，有的仅仅是volumns: xxx:xxx的挂载。如果容器是使用的挂载目录，而不是复制进去的文件，那么会无法运行相应目录中的文件和程序。

所以Dockerfile和dockerf-compose.yml结合使用。

# 容器：export和import

    docker export f299f501774c > hangger_server.tar
    docker import - new_hangger_server < hangger_server.tar


#　镜像：save和load

（1）下面使用 docker save 命令根据 ID 将镜像保存成一个文件。

    docker save 0fdf2b4c26d3 > hangge_server.tar

**(这么打包，是没有标签名字的。最好指定镜像标签名而非镜像id打包)**

（2）我们还可以同时将多个 image 打包成一个文件，比如下面将镜像库中的 postgres 和 mongo 打包：

    docker save -o images.tar postgres:9.6 mongo:3.4

载入镜像

    docker load < hangge_server.tar

# SD的镜像打包及运行

### 打包导出

先命名个版本

    docker tag f0b0777d65d7 sd_secretdetection_docker:2.3

打包，不要根据镜像id打包，不然未来没有名字，得指定tag：

    docker save -o sd-2.3.tar sd_secretdetection_docker:2.3

加载运行

    docker load < sd-2.3.tar
    docker run -itd -p 8000:8000 sd_secretdetection_docker:2.3


run镜像，并指定容器名字

指定镜像名字:

    docker run --name my_container my_image

    docker load < shenyanjian.tar
    docker run -itd -p 8000:8000 --name secretdetection shenyanjian_secretdetection_docker:latest

将容器保存为本地镜像

    docker commit -a "blingsec" -m "sd2.0" secret_detection_http secret_detection_http:v2.0

-a是作者名，-m 是标注信息，最后的secret_detection_http:v2.0 是命名







