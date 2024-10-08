---
title: delete something useless before gitignore
date: 2024-04-09 17:58:30
tags:
categories:
- Git
---

# 重写了gitignore但是想删除已提交但想忽略的内容

已经提交过的文件或者文件夹怎么办？此时更改.gitignore文件对已经提交的文件是无效的。

首先，编辑.gitignore文件。

然后如果是单个文件，可以使用如下命令从仓库中删除：

    git rm --cached logs/xx.log

如果是整个目录：

    git rm --cached -r logs

如果文件很多，那么直接

    git rm --cached -r .

如果提示某个文件无法忽略，可以添加-f参数强制忽略。

    git rm -f --cached logs/xx.log

然后

    git add .

    git commit -m "Update .gitignore"

把被忽略的某个文件强制添加回去：

    git add -f filename

ignore规则检查：

    git check-ignore

