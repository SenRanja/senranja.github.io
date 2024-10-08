---
title: goland技巧
date: 2024-04-08 17:58:30
tags:
categories:
- Go
---

- [常用快捷键](#常用快捷键)
- [添加远程调试](#添加远程调试)
- [delve：升级go无法调试](#delve升级go无法调试)


# 常用快捷键

打出err.nn可以快速打出 err!=nil

# 添加远程调试

直接调试器这里新建Go构建（不需要点击Go远程）

![2024-04-10-22-20-14](2024-04-10-22-20-14.png)

在这里配置SSH

![2024-04-10-22-20-45](2024-04-10-22-20-45.png)

# delve：升级go无法调试

我升级go从1.17到1.20后，goland无法调试。
调试时报错 

    WARNING: undefined behavior - version of Delve is too old for Go version 1.20.0 (maximum supported version 1.18)

查阅网上很多方法都不奏效。
以下为奏效方法：
https://zhuanlan.zhihu.com/p/425645473

    $ git clone https://github.com/go-delve/delve
    $ cd delve
    $ go install github.com/go-delve/delve/cmd/dlv

随后在GOPATH/bin中出现最新的dlv.exe，注意看该程序的生成时间
然后在goland的“自定义属性”中设置该值：

    dlv.path=E:\\BiLing\\golang-study\\bin\\dlv.exe

重启goland奏效






