---
title: boot after 1m30s
date: 2024-04-08 17:58:30
tags:
top: 16
categories:
- Linux-Maintain
---

开机启动1m30s慢


Ubuntu开机卡在 A start job is runing for wait for Network to be configured (1min 23s / no limit)解决方法

参考链接：https://blog.csdn.net/weixin_42300866/article/details/123712774

    root@k8s-master1:~# cd /etc/systemd/system/network-online.target.wants/
    root@k8s-master1:/etc/systemd/system/network-online.target.wants# ll
    total 8
    drwxr-xr-x  2 root root 4096 Aug 24  2021 ./
    drwxr-xr-x 19 root root 4096 Mar 24 08:11 ../
    lrwxrwxrwx  1 root root   56 Aug 16  2021 systemd-networkd-wait-online.service -> /lib/systemd/system/systemd-networkd-wait-online.service

添加内容如下：
在【service】代码快最末尾加上

    TimeoutStartSec=2sec

修改内容如下：

    #  SPDX-License-Identifier: LGPL-2.1+
    #
    #  This file is part of systemd.
    #
    #  systemd is free software; you can redistribute it and/or modify it
    #  under the terms of the GNU Lesser General Public License as published by
    #  the Free Software Foundation; either version 2.1 of the License, or
    #  (at your option) any later version.

    [Unit]
    Description=Wait for Network to be Configured
    Documentation=man:systemd-networkd-wait-online.service(8)
    DefaultDependencies=no
    Conflicts=shutdown.target
    Requires=systemd-networkd.service
    After=systemd-networkd.service
    Before=network-online.target shutdown.target

    [Service]
    Type=oneshot
    ExecStart=/lib/systemd/systemd-networkd-wait-online
    RemainAfterExit=yes
    TimeoutStartSec=2sec
    [Install]
    WantedBy=network-online.target


