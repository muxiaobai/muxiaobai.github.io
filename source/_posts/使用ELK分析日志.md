---
title: 使用ELK分析日志
date: 2019-05-18 23:35:47
tags: [分析,日志]
categories: 工具
description: "日志分析系统搭建"
---

[elk 官网介绍](https://www.elastic.co/cn/elk-stack)
要解决的问题：
对于日志等文件，需要进行分析，例如：访问IP数，什么时候访问最多，用户量最大；
简单的架构就是直接使用filebeat来获取到数据。
还有一种实现方式是：拉取文件后，先通过Logstash（tools）把对应的文件分析出来，然后输出到ElasticSearch（data）中然后使用kibaba来进行虚拟化的展示（view）。
还可以加上output的输出到队列缓存中等。


### 主要技术手段
filebeat、Logstash、ElasticSearch、Kibaba
##### filebeat
通常会有一个客户端和一个服务器，客户端运行在业务应用机上，可以访问到对应的日志文件，
然后连接服务器，服务器把数据发送到Logstash中，也可以把数据直接output到ElasticSearch内。

![ElasticSearch控制台](/使用ELK分析日志/filebeat.png)

##### Logstash
Logstash把获取到的数据进行过滤（filter）处理，把找到的文件进行分析，输出到ElasticSearch,
##### ElasticSearch
不用多讲，存储索引数据用的，基于Lucene，的分布式架构。在这种elk中充当数据源。
##### Kibaba
visual展示

### 示例 使用nginx日志来操作，简单版，不使用filebeat

定义的所有文件路径在`/c/ProgrmTool/dev/`中

##### 启动nginx 

参考[应用部署之nginx负载](https://muxiaobai.github.io/2017/09/02/%E5%BA%94%E7%94%A8%E9%83%A8%E7%BD%B2%E4%B9%8Bnginx%E8%B4%9F%E8%BD%BD/)

`/c/ProgrmTool/dev/ >start nginx`

nginx 中的log_format格式

```
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

```

日志文件在../logs/access.log

```
127.0.0.1 - - [19/May/2019:11:10:18 +0800] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Windows NT 6.3; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36" "-"
127.0.0.1 - - [19/May/2019:11:10:19 +0800] "GET /favicon.ico HTTP/1.1" 404 571 "http://localhost/" "Mozilla/5.0 (Windows NT 6.3; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36" "-"
127.0.0.1 - - [19/May/2019:11:10:19 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.3; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36" "-"
127.0.0.1 - - [19/May/2019:11:36:05 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 6.3; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36" "-"
127.0.0.1 - - [19/May/2019:11:36:05 +0800] "GET /favicon.ico HTTP/1.1" 404 571 "http://localhost/" "Mozilla/5.0 (Windows NT 6.3; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.98 Safari/537.36" "-"

```

##### 配置 Logstash

把这个地址配置到Logstash中

测试：`./logstash -e 'input {stdin 0} output {studout 0}'`
从控制台输入，控制台输出，

验证grok是否正确：[https://grokdebug.herokuapp.com/](https://grokdebug.herokuapp.com/)


配置文件：

```
# Sample Logstash configuration for creating a simple
# Beats -> Logstash -> Elasticsearch pipeline.

input {
        file {
                path => "C:/ProgramTool/dev/nginx/logs/access.log"
                type => "nginx-access"
                start_position => "beginning"
                #sincedb_path => "/usr/local/logstash/sincedb"
        }
}

filter {
        if [type] == "nginx-access" {
                grok {
                    patterns_dir => "C:/ProgramTool/dev/logstash-6.7.0/patterns"        #设置自定义正则路径
                    match => {
                        "message" => "%{NGINXACCESS}"
                        #使用patterns路径下文件内部的解析名字
                    }
                }
                date {
                    match => [ "log_timestamp" , "dd/MMM/YYYY:HH:mm:ss Z" ]
                }
                urldecode {
                    all_fields => true
                }
            #把所有字段进行urldecode（显示中文）
        }
}

output {
        if [type] == "nginx-access" {
            stdout {
                codec    => rubydebug
            }
            # 输出到控制台
            elasticsearch {
                    hosts => ["127.0.0.1:9200"]
                    manage_template => true
                    index => "logstash-nginx-access-%{+YYYY-MM-dd}"
                    # 索引名称
            }
        }

}

```
grok参数设置`C:/ProgramTool/dev/logstash-6.7.0/patterns`路径下设置nginx的匹配

```
NGINXACCESS %{IPORHOST:clientip} %{HTTPDUSER:ident} %{USER:auth} \[%{HTTPDATE:timestamp}\] "(?:%{WORD:verb} %{NOTSPACE:request}(?: HTTP/%{NUMBER:httpversion})?|%{DATA:rawrequest})" %{NUMBER:response} (?:%{NUMBER:bytes}|-) %{QS:referrer} %{QS:agent} %{QS:x_forwarded_for}

```

参考[grok参数设置](https://www.cnblogs.com/Orgliny/p/5592186.html)
更多
```
URIPARM1 [A-Za-z0-9$.+!*'|(){},~@#%&/=:;_?\-\[\]]*
URIPATH1 (?:/[A-Za-z0-9$.+!*'(){},~:;=@#%&_\- ]*)+
URI1 (%{URIPROTO}://)?(?:%{USER}(?::[^@]*)?@)?(?:%{URIHOST})?(?:%{URIPATHPARAM})?
NGINXACCESS %{IPORHOST:clientip} %{HTTPDUSER:ident} %{USER:auth} \[%{HTTPDATE:timestamp}\] "(?:%{WORD:verb} %{NOTSPACE:request}(?: HTTP/%{NUMBER:httpversion})?|%{DATA:rawrequest})" %{NUMBER:response} (?:%{NUMBER:bytes}|-) %{QS:referrer} %{QS:agent} %{QS:x_forwarded_for}
DEMOACCESS %{IPORHOST:remote_addr} - (%{USERNAME:user}|-) \[%{HTTPDATE:log_timestamp}\] %{HOSTNAME:http_host} %{WORD:request_method} \"%{URIPATH1:uri}\" \"%{URIPARM1:param}\" %{BASE10NUM:http_status} (?:%{BASE10NUM:body_bytes_sent}|-) \"(?:%{URI1:http_referrer}|-)\" (%{BASE10NUM:upstream_status}|-) (?:%{HOSTPORT:upstream_addr}|-) (%{BASE16FLOAT:upstream_response_time}|-) (%{BASE16FLOAT:request_time}|-) (?:%{QUOTEDSTRING:user_agent}|-) \"(%{IPV4:client_ip}|-)\" \"(%{WORD:x_forword_for}|-)\"
```

上面用到正则切割日志等功能

启动：`./logstash -f ../config/logstash.conf & `
`./logstash -f ../config/logstash.conf --path.data=C:/ProgramTool/dev/logstash-6.7.0/data `
如果有一个实例的话，启动时，需要指定path.data

##### 简单的es主从

master 默认9200端口
```
http.cors.enabled: true
http.cors.allow-origin: "*"

cluster.name: muxiaobai-test
node.name: master
node.master: true

network.host: 127.0.0.1

```
slave-1
```
cluster.name: muxiaobai-test
node.name: slave-1
#node.master: true

network.host: 127.0.0.1
http.port: 9300

discovery.zen.ping.unicast.hosts: ["127.0.0.1"]
```
slave-2
```
cluster.name: muxiaobai-test
node.name: slave-2
#node.master: true

network.host: 127.0.0.1
http.port: 9400

discovery.zen.ping.unicast.hosts: ["127.0.0.1"]
```

##### 使用elastic-head 图形化工具查看运行情况
[elastic-head github](https://github.com/mobz/elasticsearch-head)
需要node环境
`npm install` ` npm run start`默认9100端口 访问http://127.0.0.1:9100

![Logstash日志进入ElasticSearch](/使用ELK分析日志/elasticsearch-init.png)



##### 使用kibaba

默认端口5601，[http://localhost:5601](http://localhost:5601),把对应的索引加进去。



#### 操作如下

首先访问 nginx ，地址：http://localhost

然后可以看到Logstash窗口输出的日志

![ElasticSearch控制台](/使用ELK分析日志/logstash-nginx.png)

http://localhost:9100,中，可以看到访问日志的内容
效果如下：
![Logstash日志进入ElasticSearch](/使用ELK分析日志/elasticsearch-header.png)
具体切分的数据
![Logstash日志进入ElasticSearch](/使用ELK分析日志/data1.png)

![Logstash日志进入ElasticSearch](/使用ELK分析日志/data2.png)

创建了索引后，
![Logstash日志进入ElasticSearch](/使用ELK分析日志/kibaba2.png)
在kibaba中有默认的时间线，访问次数，即可展示
![Logstash日志进入ElasticSearch](/使用ELK分析日志/kibaba4.png)

### 使用filebeat

[nginx filebeat 配置](https://www.elastic.co/guide/en/beats/filebeat/7.0/filebeat-module-nginx.html#nginx-settings)



已经有模板，使用的时候先开启，然后安装，最后启动即可

- `filebeat  modules enable nginx`
- `filebeat setup -e`
- `filebeat`
- 

nginx.yml配置文件，需要指定日志文件路径
```
- module: nginx
  access:
    enabled: true
    var.paths: ["C:/ProgramTool/dev/nginx/logs/access.log*"]
  error:
    enabled: true
    var.paths: ["C:/ProgramTool/dev/nginx/logs/error.log*"]
```
默认直接输出到ElasticSearch



然后在Logstash的配置文件中使用beat作为input，输入源。
主要是input
```
input {
  beats {
    port => 5044
    host => "0.0.0.0"
  }
}

```
logstash.conf 默认配置就是从beat中获取数据的



参考：
- [ELK系统框架图](https://www.cnblogs.com/aresxin/p/8035137.html)
- [filebeat和Logstash配合使用](https://www.colabug.com/2936270.html)
- [官网 Logstash中的beat nginx 配置](https://github.com/elastic/logstash/blob/88563c86435926a8e5353bd970f92ab61efe58ec/docs/static/filebeat_modules/nginx/pipeline.conf)
