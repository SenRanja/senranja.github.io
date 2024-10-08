---
title: gitlab-ee的破解版返回500，501响应
date: 2024-04-08 17:58:30
tags:
categories:
- Linux-Maintain
---

鹏哥参考过的链接

    https://docs.gitlab.com/ee/install/docker.html#devshm-mount-not-having-enough-space-in-docker-container
    https://blog.kelu.org/tech/2021/12/16/docker-shm.html
    https://blog.csdn.net/weixin_43545410/article/details/120963125

# 方法一：容器启动前修改

    docker run -it --shm-size="1g" --name 001 busybox:latest /bin/sh

# 方法二：容器启动后修改

1.docker ps | grep containerName

2.获取完整 Id

    [root@linfs-bigdata containers]# docker inspect 641ae8c59c0d | grep Id
        "Id": "641ae8c59c0d20814d489cdc17a32f6ab3f35e621e5f67c2e0df1cc1b7efa269",

3.修改 ShmSize，单位为 kb，默认大小为 64M，修改完后重启 docker 就生效了

    vim /var/lib/docker/containers/641ae8c59c0d20814d489cdc17a32f6ab3f35e621e5f67c2e0df1cc1b7efa269/hostconfig.json
