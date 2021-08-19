---
title: git的一些其它操作
date: 2019-07-29 19:21:25
tags: 常用命令
categories: 工具
description: "git常见操作"
---

#### git 拉取代码失败

解决方法很简单，在git clone时加上--depth=1即可解决

> depth用于指定克隆深度，为1即表示只克隆最近一次commit.

这种方法克隆的项目只包含最近的一次commit的一个分支，体积很小，即可解决文章开头提到的项目过大导致Timeout的问题，但会产生另外一个问题，他只会把默认分支clone下来，其他远程分支并不在本地，所以这种情况下，需要用如下方法拉取其他分支：
```

$ git clone --depth 1 https://github.com/dogescript/xxxxxxx.git
$ git remote set-branches origin 'remote_branch_name'
$ git fetch --depth 1 origin remote_branch_name
$ git checkout remote_branch_name

```
只克隆master 分支 深度为1
`git clone -b master  --depth 1 https://github.com/dogescript/xxxxxxx.git`

压缩代码
`git config  --add  core.compression -1`
或者
`git config  --global  core.compression -1`

compression 是压缩的意思，从 clone 的终端输出就知道，服务器会压缩目标文件，然后传输到客户端，客户端再解压。取值为 [-1, 9]，-1 以 zlib 为默认压缩库，0 表示不进行压缩，1..9 是压缩速度与最终获得文件大小的不同程度的权衡，数字越大，压缩越慢，当然得到的文件会越小。

可以增加git的缓存大小

`git config --global http.postBuffer 1048576000`

将http.postBuffer设置的尽量大，例如git config --global http.postBuffer 524288000 （500M）
git config --global http.postBuffer 1048576000 (1G)。再大的应该是依次类推吧

####  或者使用git@  clone

-t 指定密钥类型，默认是 rsa ，可以省略。
-C 设置注释文字，比如邮箱。
-f 指定密钥文件存储文件名。

```
ssh-keygen.exe -t rsa -C "xxxx@gmail.com"
Enter file in which to save the key (/c/Users/Administrator/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
输入密码，push 的时候的密码
```

验证：

```
xxx@xxx MINGW64 ~/.ssh
$ ssh -T git@github.com
Hi xxx! You've successfully authenticated, but GitHub does not provide shell access.
```

再使用 `git clone git@github.com:xxxx/xxxx.git`
#### git config

git config --global user.name
git config --global user.email


git config -l
git config  --global -l  全局
git config  --local -l 本仓库


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

#### git add 回退 

git status 先看一下add 中的文件 
git reset HEAD 如果后面什么都不跟的话 就是上一次add 里面的全部撤销了 
git reset HEAD XXX/XXX/XXX.java 就是对某个文件进行撤销了

#### git commit 回退

`git reset --soft HEAD^`这样就成功的撤销了你的commit
注意，仅仅是撤回commit操作，您写的代码仍然保留。
 
HEAD^的意思是上一个版本，也可以写成HEAD~1

如果你进行了2次commit，想都撤回，可以使用HEAD~2

--soft  
不删除工作空间改动代码，撤销commit，不撤销git add . 

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

### 新分支 
新分支
git branch -b newbranch

提交,创建远程分支
git push origin newbranch:remotebranch

修改指向
git push --set-upstream origin newbranch
 
 关联本地分支指向
git branch --set-upstream-to=origin/remote_branch newbranch

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
