---
title: '测试Travis CI'
date: 2016-08-27 17:58:39
tags: CI
categories: Travis
description: "Travis CI"
---
-------------------------------

## 自动线上构建博客Travis CI
##### 其他的已经完成



Git global setup
git config --global user.name "zhangpf"
git config --global user.email "zpf12345678910@gmail.com"

Create a new repository
git clone git@123.57.34.235:zhangpf/vsb.git
cd vsb
touch README.md
git add README.md
git commit -m "add README"
git push -u origin master

Push an existing folder
cd existing_folder
git init
git remote add origin git@123.57.34.235:zhangpf/vsb.git
git add .
git commit -m "Initial commit"
git push -u origin master

Push an existing Git repository
cd existing_repo
git remote rename origin old-origin
git remote add origin git@123.57.34.235:zhangpf/vsb.git
git push -u origin --all
git push -u origin --tags