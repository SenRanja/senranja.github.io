---
title: chown cannot dereference
date: 2024-04-09 17:58:30
tags:
top: 19
categories:
- Docker
---

疑难杂症：拷贝mysql原始目录到其他目录，报错

    chown: cannot dereference '/var/lib/mysql/mysql.sock': No such file or directory

模仿此贴
https://github.com/docker-library/mysql/issues/939

![2024-04-10-21-36-32](2024-04-10-21-36-32.png)

解决

原因是复制docker数据过来的时候没有保持原先文件的chown属性

不需要设置privileged: true  # 允许特权模式 属性
