---
title: 只能有root操作docker，普通用户无法操作dockers处理方法
date: 2024-04-09 17:58:30
tags:
top: 20
categories:
- Docker
---

参考链接: https://www.cnblogs.com/-mrl/p/13836631.html

看当前系统中的用户和组：

    1、用户列表文件：cat /etc/passwd
    2、用户组列表文件：cat /etc/group

# 创建docker组

    vagrant@ubuntu18:~$ sudo groupadd docker
    groupadd: group ‘docker‘ already exists

# 将当前用户加入docker组

    vagrant@ubuntu18:~$ sudo gpasswd -a ${USER} docker
    Adding user xxx to group docker

# 重启docker服务

    vagrant@ubuntu18:~$ sudo service docker restart

# 刷新docker组成员

    vagrant@ubuntu18:~$ newgrp docker
    #再试试命令^_^
    vagrant@ubuntu18:~$ docker images
    REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE

然后当前的shell退了重新开一个，这样你目前的用户就可以操作docker images 命令。

