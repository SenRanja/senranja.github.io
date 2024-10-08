---
title: docker volume
date: 2024-04-09 17:58:30
tags:
categories:
- Docker
---

docker volume 有 ls rm等子命令

使用 docker rm 命令删除容器时，默认情况下不会删除与该容器关联的挂载的 **Docker 卷（Docker volume）**。Docker volume 是一个独立于容器的数据存储区域，通常用于持久化存储容器内的数据。

如果您使用了 -v 参数来挂载卷，那么删除容器时不会删除这个卷。要删除与容器关联的卷，您需要使用 docker volume rm 命令。

例如，如果您的容器启动命令中有如下的 -v 参数：

    docker run -v /host/path:/container/path my_image

那么删除容器时，卷 /host/path 不会被自动删除。要手动删除该卷，可以运行以下命令：

    docker volume rm $(docker volume ls -qf "mount=/host/path")

这将删除与指定路径关联的 Docker 卷。请注意替换 /host/path 为实际使用的卷路径。

总之，docker rm 默认不会删除挂载的 Docker 卷，您需要使用 docker volume rm 命令手动删除它们。
