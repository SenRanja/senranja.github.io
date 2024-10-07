---
title: pycharm技巧
date: 2024-04-08 17:58:30
tags:
top: 50
categories:
- Py
---

- [破解版安装](#破解版安装)
- [插件推荐](#插件推荐)
- [py解释器和虚拟环境](#py解释器和虚拟环境)
    - [虚拟解释器和机器系统的解释器不同，包环境不同](#虚拟解释器和机器系统的解释器不同包环境不同)
    - [使用pycharm手动安装包环境](#使用pycharm手动安装包环境)
- [结合conda创建新版本的python解释器并配置到pycharm](#结合conda创建新版本的python解释器并配置到pycharm)
- [代码行数统计](#代码行数统计)
- [常用快捷键](#常用快捷键)
- [增加SFTP部署](#增加sftp部署)
- [函数文档](#函数文档)
    - [实际示例](#实际示例)
- [正确的增加远程python解释器](#正确的增加远程python解释器)
    - [坑点：Pycharm:Can't get remote credentials for deployment server](#坑点pycharmcant-get-remote-credentials-for-deployment-server)
- [将项目启用为django支持](#将项目启用为django支持)


# 破解版安装

pycharm: https://www.exception.site/essay/pycharm-pojie-jihuoma

# 插件推荐

    汉化
    statistic
    rainbow brackets(部分功能30天限制，经常让你买注册码，已弃用)
    markdown（其实vscode的md阅览比pycharm的要很快，这个自行选择）
    code glance (强烈推荐，右侧滚动条变小地图)
    translation

# py解释器和虚拟环境

### 虚拟解释器和机器系统的解释器不同，包环境不同

创建一个新的项目，会想创建venv的虚拟解释器和库包
会根据系统现有的py复制一个
但是库文件是不同的，因为环境不同，如图：

![2024-04-10-22-24-45](2024-04-10-22-24-45.png)

### 使用pycharm手动安装包环境

**目前我指定虚拟解释器的py环境路径 直接 install -r requirements.txt 失败，推测requirements的版本不太对，是pycharm不支持的包版本**

但是我是用pycharm的设置 -> 项目  -> py解释器 手动安装包是可以的
此处也容易显示版本

![2024-04-10-22-25-40](2024-04-10-22-25-40.png)


![2024-04-10-22-25-56](2024-04-10-22-25-56.png)

成功解决虚拟环境运行django项目

可以正常启动

![2024-04-10-22-26-26](2024-04-10-22-26-26.png)

# 结合conda创建新版本的python解释器并配置到pycharm

**我思考了下，pycharm本身无法直接下载python3的不同版本，而手动从python官网下载不同的python3版本，操作还是太麻烦。此处依赖conda能自动下载不同python3版本，以及pycharm可以指定conda下载的py复制解释器到不同的项目，并处理不同的包依赖，可以使得调试不同项目的python解释器版本、包依赖相对分离。唯一缺点就是，软件重复程序冗余程度较高，但是这是pip自身设计的问题。**


不清楚为什么pycharm直接加载conda的解释器，会出现各种网络问题

因此手动执行命令：

    conda.exe create -p E:\anaconda3\envs\yolov5\envs -y python=3.10

（若此命令出现网络问题，参考conda的安装、运行一节的报错处理）

然后pycharm添加该环境下的venv解释器（非conda环境）：

![2024-04-10-22-27-33](2024-04-10-22-27-33.png)

相当于，conda的目录下弄了py3.10，然后又在这个代码项目下的env复制了解释器，和弄了包依赖的东西

最后手动执行命令（用户加目录下的pip的换源配置文件有效）：

    python -m pip install -r C:\Users\ranja\Desktop\ToStudy\yolov5\requirements.txt

![2024-04-10-22-27-59](2024-04-10-22-27-59.png)

最终等待此处完成即可。


![2024-04-10-22-28-25](2024-04-10-22-28-25.png)

如果pycharm检查到安装包的版本需要更新，则更新。

![2024-04-10-22-28-56](2024-04-10-22-28-56.png)

至此，解释器设置成功，按住ctrl，鼠标移动到任何函数代码，都显示可调式的高亮。


# 代码行数统计

安装插件  statistic

![2024-04-10-22-29-28](2024-04-10-22-29-28.png)

可以统计当前项目的不同类型代码、统计规则

# 常用快捷键

ctrl alt shift x 部署

# 增加SFTP部署

点击 IDEA 的文件 -> 设置 -> 项目，如图

![2024-04-10-22-30-04](2024-04-10-22-30-04.png)

点击加号减号就可以新增部署。如果部署中没有目标的SSH，则需要先添加“python解释器”，远程解释器里写新的SSH，这里的部署才会有新的SSH账密连接。


# 函数文档

在Python中，函数的文档字符串（docstring）通常被用来描述函数的功能、参数、返回值以及可能的异常等信息。这些文档字符串被包裹在一对三个双引号之间。

一种常见的格式是reStructuredText（reST）格式，这是Python的官方文档所使用的格式。在reST格式下，函数的参数（param）和返回值（return）可以这样标注：

    def function_name(param1, param2):
        """
        这是一个函数的简单描述。

        :param param1: 这是param1的描述。
        :type param1: 这是param1的类型。
        :param param2: 这是param2的描述。
        :type param2: 这是param2的类型。
        :return: 这是返回值的描述。
        :rtype: 这是返回值的类型。
        """
        pass


此外，你也可以使用Google或NumPy风格的docstring。这是Google风格的例子：

    def function_name(param1, param2):
        """
        这是一个函数的简单描述。

        Args:
            param1 (type): 这是param1的描述。
            param2 (type): 这是param2的描述。

        Returns:
            type: 这是返回值的描述。
        """
        pass

PyCharm和其他一些IDE支持这些格式，并可以根据docstring生成相应的函数/方法签名和文档。


### 实际示例

    def timeCompare(time1, time2):
        """用于比较两个类似'2023-05-17'的时间戳字符串看谁更早。time1如果比time2更早或同天的话，返回True；否则返回False

        Args:
            time1 (str):传入类似'2023-05-17'的时间字符串
            time2 (str):传入类似'2023-05-17'的时间字符串
        Return:
            res (bool):
            """
        datetime1 = datetime.strptime(time1, '%Y-%m-%d')
        datetime2 = datetime.strptime(time2, '%Y-%m-%d')
        if datetime1 <= datetime2:
            return True
        else:
            return False

    def singleCveHrefSpider(avd_url, session):
        """使用avd编号，匹配单个漏洞详情页的数据

        Args:
            avd_url(str): avd漏洞的链接地址
            session(requests.Session): 用于网络IO的session
        Returns:
            res(dict): 返回无键默认None的字典
            """

# 正确的增加远程python解释器

总结：
1.py解释器中的上传目录通常是默认传到/tmp/pycharmxxxx中，这个无所谓，改为正确的上传路径即可。
2.`部署`改成正确的上传路径
3.`py解释器`中的路径映射写对，就可以远程调试了。

设置 -> 项目 -> python解释器


![2024-04-10-22-31-57](2024-04-10-22-31-57.png)

![2024-04-10-22-32-10](2024-04-10-22-32-10.png)

这里可以改为正确的路径，不一定非得下面的 /tmp 路径

![2024-04-10-22-32-34](2024-04-10-22-32-34.png)

映射路径选择本地路径就可以。如果解释器的映射路径不对，那回到上图改一下，再到这里选择正确的本地路径就可以进行远程调试了。

Py解释器中的路径映射填写正确，就可以正常进行远程调试了。

![2024-04-10-22-33-14](2024-04-10-22-33-14.png)

### 坑点：Pycharm:Can't get remote credentials for deployment server

不知道为啥我会遇到这个报错，去项目，python解释器，把现有的SSH解释器删除了，重新添加，就没问题了

# 将项目启用为django支持

首先启用django支持，在 设置 –>  语言和框架   Django  勾选“启用Django支持”

配置如下

![2024-04-10-22-33-54](2024-04-10-22-33-54.png)

然后要运行，就点击调试配置器

![2024-04-10-22-34-08](2024-04-10-22-34-08.png)

直接配置后，运行即可


