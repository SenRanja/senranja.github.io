---
title: submodule
date: 2024-08-28 11:05:30
tags:
top: 30
categories:
- Git
---

- [开发视角：主仓库下多模块设置](#开发视角主仓库下多模块设置)
- [部署视角git pull](#部署视角git-pull)
- [真实案例](#真实案例)


# 开发视角：主仓库下多模块设置

为了方便如MVC框架的前后端管理以及减少代码错误，因此开始使用`git submodule`来做多模块管理。

在某仓库中添加submodule。

    git submodule add <git-source> <option:path>

    git status

会看到子模块是`new file`,需要

    git add .
    git commit -m "xx"
    git push

来把当前子模块的哈希打上去更新。

之后子仓库如果有更新：

    git submodule update --remote

就把hash更新了

然后主仓库再

    git add .
    git commit -m "xx"
    git push
    
就把子模块更新了

# 部署视角git pull

子模块更新和仓库本身是分离的，仓库只更新子模块的hash id
首次拉submodule（即submodule还只是个空目录的时候），使用到`--init`参数

    git clone --recursive https://github.com/xanzy/go-gitlab gitlab

    # 或者进入被克隆仓库目录，手动下载子模块
    git submodule update --init –recursive
    #–recursive的意思是递归，比如如果子模块也有子模块，他就处理这个的
    #git submodule update –init 如果没有子模块的子模块，可以直接执行这个没有递归的命令

如果涉及多个submodule，此步在运行时，如果仓库是private，你需要注意不同子仓库会需要不同的密码验证，**此步一定慢慢等待其下载完**，否则就会出现*只会更新一个submodule，而另一个submodule死活不动*的情况了

# 真实案例

拉取某个git仓库，如果该代码中某个目录下存在.git，那么git会认为这是另一个git仓库，我们在git clone的时候不会拉取该项目，必须得多一步去单拉此项目。

实际案例：鹏哥的gitmisconfig项目导入gitlab

在gitlab-misconfig项目中，由于鹏哥使用的gitlab软件包目录下存在.git，所以在进行git add . xxxx等一系列操作后，该目录下你就算有改动，也不会上传到git。

另外，如果你只是要检出其中的某个目录，则使用命令如下，可以单检出某个目录：

    git clone --recursive https://github.com/xanzy/go-gitlab gitlab
