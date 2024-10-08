---
title: Dell 730风扇速度调整
date: 2024-04-08 17:58:30
tags:
categories:
- Linux-Maintain
---

鹏哥的文章
https://h38ozbw2bw.feishu.cn/docx/Jonpd4cZmoZW2Sx0RrVcXGRsnOc

# 现象

dell emc 风扇转速100%
730安装的显卡为A2，非dell官方指定的显卡，bios主动调速速率为100%，cpu占用率低下，但是风扇全功率运行，声音一言难尽。

环境说明：

    Idrac ip:192.168.3.250

如果有多余网线，可以直接接入到公司网络，如果不方便，将自己的电脑以太网网口配置为如下，可以用网线和服务器直连。

# 解决方案

使用ipmitool手动调速
ipmitool安装：

    apt install ipmitool

github地址：https://github.com/ipmitool/ipmitool

windows下载地址：https://www.dell.com/support/home/zh-cn/drivers/driversdetails?driverid=m63f3

调速命令：

首先要关闭风扇自动调速功能，否则我们手动设置的转速是不会生效的。最后的0x00表示关闭自动调速，0x01表示开启自动调速。

    ipmitool -I lanplus -H 192.168.3.250 -U root -P 1qaz@WSX raw 0x30 0x30 0x01 0x00

关闭自动调速之后，我们就可以按照我们自己的意愿来调整转速了，我这边设置为50%。

    ipmitool -I lanplus -H 192.168.3.250 -U root -P 1qaz@WSX raw 0x30 0x30 0x02 0xff 0x32

最后的0x0a表示转速的百分比的十六进制，0a表示10%，0f表示15%。

![2024-04-10-21-58-11](2024-04-10-21-58-11.png)

通过调整发现，转速确实低了，之前一直稳定在20%-25%（5000+转）左右，功耗大概在170w。通过调低风扇转速，不仅静音了，还降低了功耗。
PS：1、不是永久设置，服务器关闭电源，再次插上电源则需要重新调整。
2、如果是跑应用，为了防止烧显卡，建议直接100%
3、配置一下邮件预警，当温度到达一定程度，主动发送邮件

# TODO

写个脚本，使用ipmitool实时获取cpu和gpu温度，当超过阈值，调用ipmitool自动调速

错误

    Error: Unable to establish IPMI v2 / RMCP+ session

原因：
1、用户名或者密码错误
2、未启用ipml

解决：

1、设置密码：
Menu Overview -> IDRAC SETTINGS -> User Authentication
-> Click on the userID of your admin account -> Next
-> check "change your password" checkbox and enter the same (or new) password
-> Apply


2、启动ipml

![2024-04-10-22-00-30](2024-04-10-22-00-30.png)

测试命令：

![2024-04-10-22-01-26](2024-04-10-22-01-26.png)





