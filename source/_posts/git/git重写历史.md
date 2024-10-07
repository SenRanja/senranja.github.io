---
title: git重写历史
date: 2024-08-24 13:39:30
tags:
top: 21
categories:
- Git
---

在第一家公司工作，我曾经写了从git history检查secret leaks的引擎工作。但是当时作为新人，对git history了解有限，当时发现git目录下secret已被删除，但是扫描git history时又依旧检测出本来已经被删除的secret的幽灵。后来才知道，原来是git history是持续性存储历史代码，而我当时单纯从最新commit删除了secret，但是git history中依然会存在这个code snippet。

可惜当时工作的时候并没有对我的代码扫描器做更进一步规划，也未能及时和领导、客户提出问题所在，后来离职后，我自个儿研究，发现确实有git重写历史的工具，尽管他不那么普遍。遗憾我没有想到这么多，是我过去的失误，今天以blog补上当时我忘记的知识点。

如果要删除 git history 带来的 leakage, 需要先把这个文件删除，然后以文件为单位将git history改写，最后再重新把正确的文件上传即可。

git重写历史步骤如下：

    git filter-branch --index-filter "git rm -r --cached --ignore-unmatch ./部署文件" HEAD`

    git filter-branch --index-filter "git rm -r --cached --ignore-unmatch ./backend/volume/media/import_excel_upload/*" HEAD

检查成功没有这个文件后，然后
    
    git push --force

如果提示需要加 -f 强制重写，如：

    git filter-branch -f --index-filter "git rm -r --cached --ignore-unmatch ./backend/volume/media/import_excel_upload/*" HEAD

最后git push即可
