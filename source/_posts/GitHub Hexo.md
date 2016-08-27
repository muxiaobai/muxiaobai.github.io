---
title: GitHub + Hexo 构建我的博客
date: 2016-06-17 14:57:31
tags: [Github,Hexo]
categories: 搭建网站
---

# 有了Hexo后，我的文章就准备全部迁移至Github了。

## 1.本机搭建
### 使用环境 
 git node npm

 Github账户 username.github.io 仓库

### 安装Hexo
  Hexo安装，要用全局安装，加-g参数。

```
zhang@admin MINGW64 ~
$ npm install –g hexo
```

<!--more-->
##### 查看hexo的版本
```
zhang@admin MINGW64 ~
$ hexo -v
hexo-cli: 1.0.2
os: Windows_NT 6.3.9600 win32 x64
http_parser: 2.5.2
node: 4.4.5
v8: 4.5.103.35
uv: 1.8.0
zlib: 1.2.8
ares: 1.10.1-DEV
icu: 56.1
modules: 46
openssl: 1.0.2h
```

hexo 帮助命令

```
zhang@admin MINGW64 ~
$ hexo help
Usage: hexo <command>

Commands:
  clean     Removed generated files and cache.
  config    Get or set configurations.
  deploy    Deploy your website.
  generate  Generate static files.
  help      Get help on a command.
  init      Create a new Hexo folder.
  list      List the information of the site
  migrate   Migrate your site from other system to Hexo.
  new       Create a new post.
  publish   Moves a draft post from _drafts to _posts folder.
  render    Render files with renderer plugins.
  server    Start the server.
  version   Display version information.

Global Options:
  --config  Specify config file instead of using _config.yml
  --cwd     Specify the CWD
  --debug   Display all verbose messages in the terminal
  --draft   Display draft posts
  --safe    Disable all plugins and scripts
  --silent  Hide output on console

For more help, you can use 'hexo help [command]' for the detailed information
or you can check the docs: http://hexo.io/docs/
```

命令行解释：

*  help 查看帮助信息
*  init 创建一个hexo项目
*  migrate 从其他系统向hexo迁移
*  version 查看hexo的版本
*  –config参数，指定配置文件，代替默认的_config.yml
*  –debug参数，调试模式，输出所有日志信息
*  –safe参数，安全模式，禁用所有的插件和脚本
*  –silent参数，无日志输出模式

### 安装好后，我们就可以使用Hexo创建项目了。

##### 新建文件夹Hexo
```
zhang@admin MINGW64 ~
$ mkdir Hexo
$ cd Hexo
```

##### 初始化进入目录。
```
zhang@admin MINGW64 ~/Hexo
$ hexo init
INFO  Cloning hexo-starter to D:\hexo
INFO  Start blogging with Hexo!
```
##### 启动服务
```
zhang@admin MINGW64 ~/Hexo
$ hexo server
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
```

这时端口4000被打开了，我们能过浏览器打开地址，http://localhost:4000/ 。
你可以按Ctrl+C 停止Server。

### Create a new post

```
zhang@admin MINGW64 ~/Hexo
$ hexo new 'My'
INFO  Created: ~\Hexo\source\_posts\My.md
```

刷新http://localhost:4000/，可以发现已生成了一篇新文章 "My"。

### Generate static files

##### 执行`hexo generate`下面的命令，将markdown文件生成静态网页。

```
zhang@admin MINGW64 ~/Hexo
$ hexo generate
```

该命令执行完后，会在 C:\Users\zhang\Hexo\public 目录下生成一系列html，css等文件。

到此已经搭建一个个人博客，但是网站都是给别人看的，因此需要发布，我们可以借用Github来部署我们的网站。

## 2.部署到Github

##### 修改配置文件

部署到Github前需要配置_config.yml文件，首先找到下面的内容

```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
```

然后将它们修改为
```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: https://github.com/muxiaobai/muxiaobai.github.io.git
  branch: master
```

##### 执行`hexo deploy`发布网站

```
zhang@admin MINGW64 ~/Hexo
$ hexo deploy
```

NOTE1:

Repository：
SSH形式的url（git@github.com:muxiaobai/muxiaobai.github.io.git ），
在SSH下，上传错误，于是就用了HTTPS形式

HTTPS形式的url（https://github.com/muxiaobai/muxiaobai.github.io.git ），
会让你输入用户名,密码.

NOTE2：

如果你是为一个项目制作网站，那么需要把branch设置为gh-pages。
### 测试
当部署完成后，在浏览器中打开https://muxiaobai.github.io/（https://muxiaobai.github.io/） ，正常显示网页，表明部署成功。
### 总结：部署步骤

每次部署的步骤，可按以下三步来进行。

hexo clean

hexo generate

hexo deploy