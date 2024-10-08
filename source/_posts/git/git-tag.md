---
title: git tag & github release
date: 2024-08-28 10:49:34
tags:
categories:
- Git
---

- [tag](#tag)
- [github release](#github-release)


# tag

`git tag`只能自己git命令打，没法在github上进行处理。git tag是独立于正常push存在的，比如新建一个tag然后push上去，不会有新commit。push tag仅是推了tag的名字而已。

    git tag 2.0
    # tag本质就是这个a参数 annotated，也可以附带message信息（通常较少见使用-m）
    git tag -a v1.0 -m "Release version 1.0"
    
    git tag
    >>> 2.0

    # 方便未来checkout切换到tag表示的某具体版本
    git checkout v1.0

以上仅仅是**local tag**，推tag上去需要执行命令

    git push origin --tags

或者某个单一的tag
    
    git push origin <tag>

![git_tag](git_tag.png)

# github release

打了tag后，可以发布`release的tarballs的source code`或者`二进制文件`（这些功能是**github特有功能**，而**非git**自身能支持的功能）

比如，如果没有选择tag，那么你发布不成功release。如图我暂时保存为草稿，再选择个tag进行发布。点击“笔”那个进行编辑。

![tag2](tag2.png)

![tag3](tag3.png)

点击 发布release后，如下，可以看到这个tag的源代码，默认会以tar.gz和zip的形式放在release中，另外我们手动一个一个传的文件也在其中。

![tag4](tag4.png)
