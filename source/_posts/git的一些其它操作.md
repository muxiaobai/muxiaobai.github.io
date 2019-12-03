---
title: git的一些其它操作
date: 2019-07-29 19:21:25
tags: 常用命令
categories: 工具
description: "git常见操作"
---


#### git diff

先把vscode作为git默认编辑器

git config --global core.editor "code --wait"

用vscode 打开 .gitconfig文件

git config --global -e

在里面加上

 
```
[diff]
    tool = default-difftool
[difftool "default-difftool"]
    cmd = code --wait --diff $LOCAL $REMOTE
```

这时候运行git difftool，vscode 就作为默认difftool打开了

#### 修改远程仓库 origin，更换地址

查看远程地址

git remote -v  

移除origin远程地址

git remote rm origin

添加origin 远程地址

git remote add origin https://github.com/muxiaobai/xxx

拉取origin下的代码

git fetch origin


### 已存在的git仓库更换地址

cd existing_repo
git remote rename origin old-origin
git remote add origin http://192.168.120.63/xxx/xxx.git
git push -u origin --all
git push -u origin --tags

#### 切换分支

查看远程和本地分支
git branch -a

创建并切换dev 分支
git checkout -b dev

切换到dev分支
git checkout dev

删除远程xxx分支
git remote rm xxx



#### 

首先，git fetch --all  取回远程库的所有修改；
然后，git reset --hard origin/master
指向远程库origin的master

#### 回滚上次提交

git reset \--hard HEAD^
git log
git reset commit_id

### 每个人的数据库连接信息不一样，可以选择忽略
具体操作如下：

在命令行中输入

git update-index --assume-unchanged [file-path]
命令中的file-path 就是需要忽略提价的文件的路径

如果需要恢复提交，使用：

git update-index --no-assume-unchanged [file-path]
