---
title: mysql-docker
date: 2024-04-09 17:58:30
tags:
top: 1
categories:
- Docker
---

mysql的docker：https://registry.hub.docker.com/_/mysql


我的文件名称是docker-compose-basic.yml,MySQL数据挂载到宿主机/root/basic/mysql下

数据库一般需要将数据挂载到宿主机上,以免容器损坏导致数据丢失

    version: '3'
    services:
    #mysql
    mysql:
        image: mysql:5.6
        container_name: mysql
        environment:
        - MYSQL_ROOT_PASSWORD=1qaz@WSX
        - TZ=Asia/Shanghai
        - SET_CONTAINER_TIMEZONE=true
        - CONTAINER_TIMEZONE=Asia/Shanghai
        volumes:
        - ./mysql/conf:/etc/mysql/conf.d
        - ./mysql/conf:/etc/mysql/mysql.conf.d
        - ./mysql/data:/var/lib/mysql
        - ./mysql/logs:/var/log/mysql
        - /etc/localtime:/etc/localtime:ro
        ports:
        - 13306:3306
        restart: always

关于修改配置文件my.cnf(类似于my.ini)这种文件，上面已经指向了/etc/mysql/mysql.conf.d和/etc/mysql/conf.d两处配置文件的目录，但是完全可以在上面的挂载目录下新建my.cnf。因为我们的cnf文件也是配置文件。
