---
title: 新建某分支并推送、合并分支
date: 2024-04-09 17:58:30
tags:
top: 1
categories:
- Git
---

# 新建某分支并推送至某分支：以考试系统es为例版本管理

新建新分支并推送至新分支之前，需要检查当前分值是否还有未提交的状态

`git status` # 可以检查是否有新文件未被track

`git remote -v` # 查看远程分值，一般如下，同一个地址具有拿下或push的权限

```
origin  http://shenyanjian.cn:3000/senranja/exam_system.git (fetch)
origin  http://shenyanjian.cn:3000/senranja/exam_system.git (push)
```

`git fetch` # 拿下仓库最新分值信息。`git pull==git fetch + git merge`，git fetch是安全的、不会合并的命令

`git branch -r` # 查看全部的远程分支，通常显示如下：（若远程分支只有main）

```
  origin/HEAD -> origin/main
  origin/main
```

`git checkout -b V0.0.2` # 以当前分支为副本创建V0.0.2

`git checkout V0.0.2` # 切换到分值V0.0.2

（现在在此处放一些部署文档、版本交付说明）

```shell
git add .
git commit -m "V0.0.2"
git push
```

```
fatal: The current branch V0.0.2 has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin V0.0.2
```
如上报错，意思是远程分值还没有V0.0.2，需要手动设置下

    git push --set-upstream origin V0.0.2

完成

切换回主分支：

    git checkout main

# 本地合并其他分支到本分支，再推送

要将本地的 dev 分支合并到 main 分支，你可以按照以下步骤进行操作：

确保你当前位于 main 分支上。你可以通过以下命令来确保：

    git checkout main

然后，执行 git pull 命令来确保你的 main 分支是最新的：

    git pull origin main

接着，执行 git merge 命令来合并 dev 分支到 main 分支：

    git merge dev

如果有冲突，在合并过程中会停下来，需要你解决这些冲突。解决完冲突后，你需要执行 `git add `命令来标记冲突已解决的文件，然后执行 `git merge --continue` 来继续合并过程。

最后，当合并完成，你可以将合并后的 main 分支推送到远程仓库：

    git push origin main

通过这些步骤，你就可以将本地的 dev 分支成功合并到 main 分支，并将更新推送到远程仓库。



