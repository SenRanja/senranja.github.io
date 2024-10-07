---
title: 开启ipv4转发
date: 2024-04-09 17:58:30
tags:
top: 1
categories:
- Docker
---

# 无法访问到docker映射出来的端口

临时开启ipv4转发

    echo 1 >/proc/sys/net/ipv4/ip_forward

并重启docker服务

    service docker restart

永久开启路由功能

    vim /etc/sysctl.conf
    net.ipv4.ip_forward = 1

然后容器内也能上网了

