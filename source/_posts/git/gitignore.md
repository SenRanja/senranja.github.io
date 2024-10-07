---
title: .gitignore
date: 2024-04-09 17:58:30
tags:
top: 15
categories:
- Git
---

官方文档：`https://github.com/github/gitignore`


### 我的.gitignore模板

```
# Logs
logs
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*
lerna-debug.log*

node_modules
dist
dist-ssr
*.local

# big files
*.mp4
backend/volume/media/documents/*
!backend/volume/media/documents/.gitkeep
backend/volume/media/import_excel_upload/*
!backend/volume/media/import_excel_upload/.gitkeep
backend/volume/media/videos/*
!backend/volume/media/videos/.gitkeep

# python
**/__pycache__/
**.pyc
# 忽略pycharm
**/.idea/
# 忽略虚拟环境
**/venv


# Editor directories and files
**/.vscode/*
!.vscode/extensions.json
.idea
.DS_Store
*.suo
*.ntvs*
*.njsproj
*.sln
*.sw?
```

### 其他示例

```
# 忽略所有 .a 结尾的文件
a
# 但lib.a 除外
!lib.a
# 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
/TODO
# 忽略build/ 目录下的所有文件
build/
# 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
doc/.txt
```
