---
title: 应用部署之nginx负载
date: 2017-09-02 10:46:10
tags: [nginx,tomcat]
categories: deploy
description: "通过nginx部署两个tomcat并实现session共享"
---

## 端口占用

#### 1、在windows下如何查看80端口占用情况?是被哪个进程占用?如何终止等.
        这里主要是用到windows下的DOS工具,点击"开始"--"运行",输入"cmd"后点击确定按钮,进入DOS窗口,接下来分别运行以下命令:
        >netstat -aon | findstr "80"
                Proto  Local Address          Foreign Address        State            PID
                ====  ============      ==============  ==========  ======
                TCP    0.0.0.0:80                    0.0.0.0:0                LISTENING      1688
可以看出80端口被进程号为1688的程序占用.
        >tasklist | findstr "1688"
图像名                                                PID            会话名                    会话#       内存使用
               ========================= ====== ================ ======== ============
               inetinfo.exe                                        1688           Console                      0              2,800 K
很明显,是inetinfo占用了80端口;inetinfo.exe主要用于支持微软Windows IIS网络服务的除错,这个程序对你系统的正常运行是非常重要的.
        当然,并不是只有inetinfo.exe进程会占用80端口,这只是我机器上的情况.如果你并不了解此进程是干什么用的,千万不要盲目地将其kill掉,最好先百度或Google搜索一下;当然如果你很了解它,并确定可以终止,那么继续下面的命令.
        >taskkill /pid 1688 /F
成功: 已终止 PID 为 1688 的进程。
如果你很熟悉此进程,并确定可以终止,那么就直接使用上面的命令把PID为1688的进程终止.(这一步同样可以在任务管理器中执行,inetinfo.exe就是任务管理器中的映像名称,选中它,点击"结束进程"即可)
        >tasklist | findstr "1688"
再次确认是否成功终止,如果成功终止此次执行命令后应返回空.

#### 2、linux下如何查看80端口占用情况?是被哪个进程占用?如何终止等

查询端口是否被占用，被哪个进程占用有两种方式：1、netstat -anl | grep "80" ；2、lsof -i:80

终止进程的方式：kill pid

## 启动停止nginx

#### 1、启动：

C:\server\nginx-1.0.2>start nginx

或

C:\server\nginx-1.0.2>nginx.exe

注：建议使用第一种，第二种会使你的cmd窗口一直处于执行中，不能进行其他命令操作。

#### 2、停止：

C:\server\nginx-1.0.2>nginx.exe -s stop

或

C:\server\nginx-1.0.2>nginx.exe -s quit


注：stop是快速停止nginx，可能并不保存相关信息；quit是完整有序的停止nginx，并保存相关信息。

#### 3、重新载入Nginx：

C:\server\nginx-1.0.2>nginx.exe -s reload

当配置信息修改，需要重新载入这些配置时使用此命令。

#### 4、重新打开日志文件：

C:\server\nginx-1.0.2>nginx.exe -s reopen

#### 5、查看Nginx版本：

C:\server\nginx-1.0.2>nginx -v

## 配置nginx.conf

```

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;
    # gzip 压缩
    gzip  on; #Gzip compression

    #Server cluster 
    upstream  netitcast.com {  #Server cluster   
        server    127.0.0.1:8081  weight=1; #Weight is the meaning of weight, the greater the weight, the greater the probability of distribution. 
        server    127.0.0.1:8082  weight=2;  
    }
    server {
        listen       80;
        server_name  127.0.0.1;

        #charset koi8-r;

        #access_log  logs/host.access.log;
        # 日志按天生成
    	if ($time_iso8601 ~ '(\d{4}-\d{2}-\d{2})') {
            set $tttt $1;
        }
        access_log  logs/access-$tttt.log  main;

        #location / {
        #    root   html;
        #    index  index.html index.htm;
        #}
        location / {  
            proxy_pass http://netitcast.com;  #proxy name is upstream name
            proxy_redirect default;  
            expires      3d;   #cache three days
        }  
        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #     listen       8081;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

```