---
title: docker network
date: 2024-04-09 17:58:30
tags:
top: 20
categories:
- Docker
---


# **docker network网络的优势**

同一个 Docker **网络中的容器之间可以使用容器名进行网络通信，就像访问域名一样**。这是因为 Docker 默认提供了一个内置的 DNS 服务，它会为在同一网络中运行的容器分配主机名，并且其他容器可以使用这些主机名进行通信。

# 创建docker network

使用 Docker 网络：另一种可能的解决方案是创建一个新的 Docker 网络，并在这个网络上运行你的容器。你可以使用 docker network create 命令创建一个新的网络，然后使用 --network 选项运行你的容器。

调整防火墙规则：如果你的主机上运行有防火墙，可能需要调整防火墙规则以允许 Docker 容器访问内网。具体的步骤取决于你的防火墙软件和配置。

**检查 DNS 配置：如果你的问题与 DNS 解析有关，可能需要检查和调整你的 Docker 容器的 DNS 配置。你可以通过在 Docker 守护进程的配置文件（例如 /etc/docker/daemon.json）中设置 dns 选项来全局配置 DNS，也可以使用 --dns 选项在运行容器时指定 DNS。**

**使用 VPN：如果你的内网服务在 VPN 后面，你可能需要在 Docker 容器中设置 VPN。这可能需要在你的 Docker 镜像中安装和配置 VPN 客户端。**




