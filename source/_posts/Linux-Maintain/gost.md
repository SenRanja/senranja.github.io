---
title: gost
date: 2024-04-08 17:58:30
tags:
categories:
- Linux-Maintain
---

项目参考
https://github.com/ginuerzh/gost

# 代理转发

只要本地客户端执行-L，就可以转发http，https，socks等协议的转发，但是实测我无法https转发，会遇到证书的报错
	
### kcp协议： udp封装的协议

    server:
    gost -L kcp://gost:1qaz@WSX@45.77.115.252:52783
    client:
    gost -L=:6666 -F=kcp://gost:1qaz@WSX@45.77.115.252:52783


### mwss协议：多媒体协议

    server:
    ./gost -L mwss://fivecolorstone:fivecolorstone@192.210.xxx.99:29753
    gost -L mwss://fivecolorstone:fivecolorstone@45.77.115.252:29723
    client:
    gost -L=:6666 -F=mwss://fivecolorstone:fivecolorstone@192.210.xxx.99:29753
    gost -L=:6666 -F=mwss://fivecolorstone:fivecolorstone@45.77.115.252:29723


# curl挂代理检测socks代理

curl -x socks5://127.0.0.1:1024 http://www.google.com # -x 参数等同于 --proxy

访问ip.sb会直接显示当前你连接的ip

    curl -x socks5://127.0.0.1:6666 ip.sb


# 在racknerd的socks的vps搭建的坑点

如果是AlmaLinux，则使用yum安装 net-tools 等软件
防火墙不同于ubuntu的ufw，使用firewalld

需要关闭firewalld：

    systemctl stop firewalld
    systemctl disable firewalld

在如下链接中找到解决方案：
https://almalinux.discourse.group/t/no-match-for-argument-screen/790
    
    dnf install epel-release
    dnf install screen

然后再 screen -S gost开screen的shell，运行命令
mwss协议：

    server:
    ./gost -L mwss://fivecolorstone:fivecolorstone@192.227.xxx.113:39753
    client:
    gost -L=:6666 -F=mwss://fivecolorstone:fivecolorstone@192.227.xxx.113:39753

# 内网穿透

参考帖子：
https://github.com/ginuerzh/gost/issues/745

假设A地址是公网vps，B是某内网服务器，C是客户端
B、C不能访问，C想访问B的5000端口的服务，需要以A为中继，gost部署如下：

A部署：
    
    gost -L socks5://:2345 (意思是，建立端口为2345的gost服务端。2345仅是服务端，用来gost访问搭桥的，不是让人类访问的)

B部署：

    gost -L rtcp://:8080/:5000 -F socks5://10.80.5.244:2345 (意思是，将机器192.168.217.128端口5000绑定到机器10.80.5.244端口8080上。此处rtcp无所谓，gost支持的都行)

C访问：

    访问 A:8080，就可以访问到B:5000 了。


### 案例：将gitea内网穿透

服务端（192.227.167.113）：

    ./gost -L socks5://:38441

被穿透机器：

    gost -L rtcp://:3001/:3000 -F socks5://192.227.xxx.113:38441

访问：
http://192.227.167.113:3001/


