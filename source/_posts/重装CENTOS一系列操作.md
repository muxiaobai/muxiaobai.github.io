---
title: 重装CENTOS一系列操作
date: 2018-03-02 01:43:26
tags: Linux
categories: [Linux]
description: "手贱，删掉引导盘内容，又重新安装，幸好之前的资料已经上传的github了"
---


## 首先，安装的时候，

使用UltraISO(软碟通)进行刻录U盘，修改盘符名为CENTOS

启动选择U盘启动，

选中install   按table键，修改label

将菜单中vmlinuz initrd=initrd.imginst.stage2=hd:LABEL=CentOS\x207\x20x86_64 rd.live.check quiet

改为：vmlinuz initrd=initrd.img inst.stage2=hd:LABEL=CENTOS quite

以上都是系统安装的问题

引导问题 使用EasyBCD修改

UltraISO EasyBCD 百度有资源下载

## 安装之后，

资源在

进入系统需要修改 java python  pip  默认路径oracle jdk 8+  python3 

之后安装 中文键盘  yum install"@Chinese support"
中文  https://blog.csdn.net/sunxiaopengsun/article/details/53965643

teamviewer eclipse MARS  hadoop  anaconda

python 库 g++ numpy pands matplotlib sklearn xgboost tensroflow 


bash Anaconda-2.1.0-Linux-x86_64.sh
#### CentOS7 联网问题

cd /etc/sysconfig/network-scripts打开配置文件

vi ifcfg-ens33这里可能你的文件名不是这个，但是找前面是 ifcfg-ens 的就是了

将文件里的 ONBOOT=no，改为ONBOOT=yes，然后保存并退出（不要忘记保存！！）

修改完成之后需要重新启动一下网络服务，才能生效。使用下面的命令。

service network restart

#### 环境变量

解压缩之后环境变量

`vim .bashrc`

export ANACONDA_HOME=/root/anaconda3
export PYTHON_HOME=/root/install/Python-3.6.5
export JDK_HOME=/root/install/jdk1.8.0_161
export CLASSPATH=.;$JDK_HOME/lib
export PATH=/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin;/root/bin
export PATH=$PYTHON_HOME:$ANACONDA_HOME/bin:$JDK_HOME/bin:$PATH

`source .bashrc`
#### 修改jupyter
[jupyter 默认配置](https://www.cnblogs.com/dpf-learn/p/7941960.html)
运行
`jupyter notebook --generate-config --allow-root`

修改： /root/.jupyter/jupyter_notebook_config.py  

vim .jupyter/jupyter_notebook_config.py 

```
62 #c.NotebookApp.allow_root = False  
去掉62行的注释，并修改成True即可解决root权限运行的问题。  
163 #c.NotebookApp.ip = 'localhost'  
去掉注释，并把localhost改成0.0.0.0，这样就可以外部访问了，默认只有在本机可以访问的；  
163 c.NotebookApp.ip = '0.0.0.0'  

c.NotebookApp.base_project_url = '/root/jupyter'

203 #c.NotebookApp.notebook_dir = '/root/jupyter'  
改成如下，这样就会默认把notebook上创建的文件保存到指定目录下；需要事先创建。   
203 c.NotebookApp.notebook_dir = u'/opt/jupyter'  
```

#### tensorflow

[conda 安装tensorflow](https://www.cnblogs.com/willnote/p/6746499.html)

修改镜像源
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --set show_channel_urls yes

conda install numpy   #测试是否添加成功

之后会自动在用户根目录生成“.condarc”文件，Ubuntu环境下路径为~/.condarc，Windows环境下路径为C:\用户\your_user_name\.condarc

channels:
 - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
 - defaults
show_channel_urls: yes

如果要删除镜像，直接删除“.condarc”文件即可

`anaconda search -t conda tensorflow`
`anaconda show anaconda/tensorflow-base`对应版本
`conda install --channel https://conda.anaconda.org/anaconda tensorflow-base` 对应https

#### xgboost

需要 `yum install gcc gcc-c++`

`pip install xgboost` or ` conda install xgboost`


