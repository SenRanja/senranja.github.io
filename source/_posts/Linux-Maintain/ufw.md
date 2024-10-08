---
title: ufw
date: 2024-04-08 17:58:30
tags:
categories:
- Linux-Maintain
---

firewall和ufw，两者安装其一即可，都安装的话会冲突。

# iptables

iptables 是一个通过控制 Linux 内核的 netfilter 模块来管理网络数据包的流动与转送的应用软件，其功能包括不仅仅包括防火墙的控制出入流量，还有端口转发等等。

iptables 内部有表 tables、链 chains、规则 rules 这三种概念。
iptables 的每一个 “表” 都和不同的数据包处理有关、决定数据包是否可以穿越的是 “链”、而一条 “规则” 在链里面则可以决定是否送往下一条链（或其它的动作）。

iptables 是 firewall 和 ufw 的底层。

# firewalld

如almaLinux、centos使用 firewalld 为防火墙，区别于 ubuntu 的ufw作为防火墙。

关闭防火墙可以使用systemctl stop firewalld 和 systemctl disable firewalld 。

    firewall-cmd --permanent --add-port=51080/tcp
    [root@vultr ~]# firewall-cmd --list-ports
    50022/tcp 51080/tcp 38388/tcp 38388/udp 23307/tcp

# ufw

UFW（Uncomplicated Firewal）是 Ubuntu 下基于 iptables 的接口，旨在简化配置防火墙的过程。默认情况下 UFW 为关闭状态，开启时默认为拒绝所有传入链接，并允许所有传出连接。只有root权限才能操作

    ufw status verbose

打印拒绝连接的地址，和开放的端口、关闭的端口

UFW防火墙的默认行为是**阻止所有传入和转发流量**，并**允许所有出站流量**。这意味着除非您打开指定的端口，否则任何尝试访问您的服务器的人都将无法连接。

    sudo ufw allow ssh
    sudo ufw enable

将会启用ubuntu防火墙，并提示你命令可能会中断SSH的连接是否要进行该操作

打开端口，如果未给出协议，则UFW会同时为tcp和udp创建规则：

    ufw allow port_number/protocol

如果写协议名字的话，ufw会检查**/etc/services**中对应指定的服务和端口

UFW还允许您打开指定的端口范围。起始端口和结束端口用冒号:分隔，并且您必须指定协议tcp或udp。

例如，如果要同时在tcp和udp允许来自7100到7200端口的连接，则可以运行命令

    sudo ufw allow 7100:7200/tcp。

指定端口号最常用的方式例如命令sudo ufw allow 80将会打开80端口。UFW还支持使用proto关键字指定协议。

    sudo ufw allow 80 #tcp和udp
    sudo ufw allow 80/tcp #仅tcp
    sudo ufw allow proto tcp to any port 80

### 允许源IP地址接口/网卡

如果只写协议（如 http ssh），会检查/etc/services对应端口来封端口


要允许来自指定源IP的连接，请使用from关键字，后跟源地址。如果要仅允许指定的IP地址访问指定的端口，请使用to any port关键字，后跟端口号

    sudo ufw allow from 192.168.1.100 #仅允许单IP地址
    sudo ufw allow from 192.168.1.100 to any port 3306 #仅允许单IP地址连接3306
    sudo ufw allow from 192.168.1.0/24 to any port 3306
    sudo ufw allow in on eth2 to any port 3306

    ufw allow ssh
    ufw allow 7722/tcp

### 拒绝连接

所有传入连接的默认策略均设置为deny，如果您未更改默认策略，除非打开指定端口的连接，否则UFW会阻止所有传入连接。

撰写拒绝规则与撰写允许规则相同。使用deny关键字而不是allow。假设打开了端口80和443，并且服务器受到23.24.25.0/24网络的攻击。要拒绝来自23.24.25.0/24的所有连接。

    sudo ufw deny from 23.24.25.0/24

命令将会拒绝23.24.25.0/24网段的连接，如果你仅需要拒绝指定IP地址的连接，则不需要添加子网掩码。

你还可以拒绝指定的IP地址连接到指定的端口，例如命令sudo ufw deny proto tcp from 23.24.25.0/24 to any port 80,443。将会拒绝23.24.25.0/24访问端口80和443的示例

    deny：
        当使用 deny 动作时，防火墙会默默地丢弃被拒绝的数据包，而不向源发送任何响应。
        对于 TCP 连接，源主机将继续尝试发送数据包，直到达到超时或放弃为止。
        这种动作会使得攻击者难以确定目标主机是否在线，因为他们不会收到拒绝连接的响应。
        但这也可能导致一些问题，比如可能会使得攻击者更难以确定是由于网络故障还是被防火墙拒绝。

    reject：
        当使用 reject 动作时，防火墙会发送一条 ICMP 错误消息（如“目标不可达”）给源主机，通知其连接被拒绝。
        对于 TCP 连接，这将导致源主机立即收到拒绝连接的响应，从而不再尝试建立连接。
        与 deny 不同，使用 reject 动作会立即通知源主机连接被拒绝，这样可以更快地释放网络资源并减少攻击者的网络流量。
---
    sudo ufw deny from 23.24.25.100 #拒绝指定的IP连接
    sudo ufw deny from 23.24.25.0/24  #整个网段
    sudo ufw deny proto tcp from 23.24.25.0/24 to any port 80,443
    拒绝使用reject
    ufw reject from 202.54.5.7 to any

### 查看并删除防火墙规则

    sudo ufw status numbered

假设要删除的ufw规则编号是3，该规则号允许连接到端口8080。你可运行命令

    sudo ufw delete 3

第二种方法是通过指定实际规则来删除规则。例如，如果您打开了8069端口的规则，则可以运行命令sudo ufw delete allow 8069将其删除。

    sudo ufw status numbered #查看防火墙的状态
    sudo ufw delete 3 #根据编号删除
    sudo ufw delete allow 8069 通过规则删除规则


### 禁用/启用

如果出于某些原因要停止UFW并停用所有规则，则可以运行命令sudo ufw disable禁用防火墙。

以后，如果您想重新启用ufw并激活所有规则，运行命令sudo ufw enable即可。

重置UFW将禁用UFW，并删除所有活动规则。 如果您不想还原所有更改并重新开始，这将很有帮助。要重置UFW，可以运行命令sudo ufw reset。

    sudo ufw disable #禁用
    sudo ufw enable #开启
    sudo ufw reset #重置

### IP伪装/转发

IP伪装是Linux内核中NAT网络地址转换的一种变体，它通过重写源IP地址和目标IP地址端口来转换网络流量。借助IP伪装，您可以使用一台Linux计算机充当网关，允许私有网络中的一台或多台计算机与互联网通信。例如VMware或者Virtualbox此类虚拟软件就是通过一个NAT适配器充当网卡，转发多台虚拟机网络数据，连接到互联网。

使用UFW配置IP伪装涉及几个步骤。首先，您需要启用IP转发ip_forward。

请使用你喜欢的编辑器编辑/etc/ufw/sysctl.conf文件，在本教程中将使用vim编辑器打开文件：

    sudo vim /etc/ufw/sysctl.conf

查找并取消注释以下行：

    net/ipv4/ip_forward=1

您还需要配置UFW以允许转发数据包。打开UFW配置文件/etc/default/ufw。找到DEFAULT_FORWARD_POLICY键，然后将值从DROP更改为ACCEPT。

    sudo vim /etc/default/ufw

#找到DEFAULT_FORWARD_POLICY改为ACCEPT

    DEFAULT_FORWARD_POLICY="ACCEPT"

现在，您需要在nat表中设置POSTROUTING链的默认策略和伪装规则。 请打开/etc/ufw/before.rules文件。

    sudo vim /etc/ufw/before.rules

追加以下几行到文件/etc/ufw/before.rules

    #NAT table rules 启用nat 表
    *nat
    # 允许POSTROUTING 链
    :POSTROUTING ACCEPT [0:0]

    # 转发eth0接口的数据包，请将eth0更改为你对应的接口
    -A POSTROUTING -s 10.8.0.0/16 -o eth0 -j MASQUERADE

    # don't delete the 'COMMIT' line or these rules won't be processed
    COMMIT

注意不要删除COMMIT关键词，它永远是在最后一行。别忘了在-A POSTROUTING行中替换eth0以匹配你的计算机可以连接到互联网的名称。

完成后，保存并关闭文件。最后，通过命令sudo ufw disable禁用和命令sudo ufw enable重新启用UFW重新加载UFW规则。

### systemctl enable ufw和ufw enable区别

我对这两个命令疑惑，不过有相关的说明：
https://unix.stackexchange.com/questions/555020/how-should-i-enable-ufw-through-systemctl-enable-or-ufw-enable

systemctl只是要不要考虑ufw作为系统引导时启动的后台进程。

ufw本身默认是关着的，systemctl status ufw可以看到ufw是enabled，但是ufw status看到他是inactive，这是不冲突的，要开启ufw需要 ufw enable.



