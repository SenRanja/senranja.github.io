---
title: dockerhub && docker
date: 2024-04-09 17:58:30
tags:
top: 1
categories:
- Docker
---

# 中国用户换源

    vim /etc/docker/daemon.json
    {
    "registry-mirrors":["https://hub-mirror.c.163.com","https://registry.aliyuncs.com","https://registry.docker-cn.com","https://docker.mirrors.ustc.edu.cn"]
    }

    service docker restart



# dockerhub

log in https://hub.docker.com/, register an account.

pull your private image
    
    docker pull senranja/secret_detection

build an image with tag, upload image to dockerhub

    
    docker image build -t senranja/secret_detection:v2.1 .
    docker login
    docker push username/repo:tag

# 搭建gitlab-ce为例

安装gitlab-ce参考链接：

    https://blog.csdn.net/BThinker/article/details/124097795

查找镜像

    docker search gitlab

拉取Gitlab镜像

    docker pull gitlab/gitlab-ce:latest

启动容器，这里docker run命令的-v和dockerfile以及docker-compose.yaml中的-v 挂在目录，不需要你自己手动创建目录。即便目录不存在，他会自动创建。

    docker run \
    -itd  \
    -p 9980:80 \
    -p 9922:22 \
    -v /home/gitlab/etc:/etc/gitlab  \
    -v /home/gitlab/log:/var/log/gitlab \
    -v /home/gitlab/opt:/var/opt/gitlab \
    --restart always \
    --privileged=true \
    --name gitlab \
    gitlab/gitlab-ce

命令

    -i	 以交互模式运行容器，通常与 -t 同时使用命令解释
    -t	 为容器重新分配一个伪输入终端，通常与 -i 同时使用
    -d	后台运行容器，并返回容器ID
    -p 9980:80	将容器内80端口映射至宿主机9980端口，这是访问gitlab的端口
    -p 9922:22	 将容器内22端口映射至宿主机9922端口，这是访问ssh的端口
    -v /home/gitlab/etc:/etc/gitlab	将容器/etc/gitlab目录挂载到宿主机/usr/local/gitlab-test/etc目录下，若宿主机内此目录不存在将会自动创建，其他两个挂载同这个一样
    --restart always	容器自启动
    --privileged=true	让容器获取宿主机root权限
    --name gitlab	设置容器名称为gitlab
    gitlab/gitlab-ce	镜像的名称，这里也可以写镜像ID


# 复制目录细节

屁股加不加 /没区别

1.	目录下文件复制到目标目录下，不包含目录本身

    COPY ./dist/ /usr/share/nginx/html/

是将 ./dist/下文件 放入 /html/，dist本身没目录

2.	连带这个目录复制进入

    COPY ./dist /usr/share/nginx/html/dist

# 删除none镜像

    docker images -a | grep "<none>" | awk '{print $3}' | xargs docker rmi

















