---
title: Elasticsearch集群化部署方案
date: 2020-06-23 20:28:19
tags: 
categories: 集群
description: "安装集群ES,集群方案"
---


应用服务IP为:192.168.1.2,ES_HOME为安装目录

### 1.修改elasticsearch.yml配置：

修改ES_HOME/config.elasticsearch.yml
network.host: 192.168.1.2 # 对外暴露的IP，本机IP
http.port: 9200 #设置对外服务的http端口号
transport.tcp.port: 9300 #设置节点之间交互的端口号
discovery.zen.ping.unicast.hosts: ["192.168.1.2:9300","192.168.1.2:9301","192.168.1.2:9302"]
#集群IP其它可以是主节点的IP加transport.tcp.port端口

修改ES_HOME/config/analysis-hanlp/hanlp.properties
root 为ES_HOME绝对路径

修改ES_HOME/config/analysis-hanlp/hanlp-remote.xml
remote_ext_dict和remote_ext_stopwords
<entry key="remote_ext_dict">http://192.168.1.2:8080/dict</entry>
<entry key="remote_ext_stopwords">http://192.168.1.2:8080/stop_words</entry>

可以添加hanlp插件或者ik插件 并配置远程更新词库

```
    @Logs("分词热更新到分词器中接口")
    @RequestMapping("/dict")
    public ResponseEntity dict(WebRequest request, HttpServletResponse response) {
        // 1. 应用相关的方式计算得到(application-specific calculation)
        Date date = new Date();
        if (request.checkNotModified(date)) {
            return null;
        }
        List<String> list= new ArrayList();
        return ResponseEntity.ok().lastModified(date).body(String.join("\n",list));
    }
    @Logs("停顿词热更新到分词器中接口")
    @RequestMapping("/stop_words")
    public ResponseEntity stopWords(WebRequest request, HttpServletResponse response) {
        // 1. 应用相关的方式计算得到(application-specific calculation)
        Date date = new Date();
        if (request.checkNotModified(date)) {
            return null;
        }
        List<String> list= new ArrayList();
        return ResponseEntity.ok().lastModified(date).body(String.join("\n",list));
    }
```

### 2.	修改下面三个配置

#### vim /etc/security/limits.conf

错误1：max file descriptors [65535] for elasticsearch process is too low, increase to at least [65536]
这个问题是无法创建本地文件,用户最大可创建文件数太小
解决：只需要修改创建文件的最大数目为65536就行了
root用户修改
vim /etc/security/limits.conf


    root soft nofile 65536
    root hard nofile 65536
    * soft nofile 65536
    * hard nofile 65536

保存、退出、重新登录才可生效

参数解释：
- soft nproc:可打开的文件描述符的最大数(软限制)
- hard nproc:可打开的文件描述符的最大数(硬限制)
- soft nofile:单个用户可用的最大进程数量(软限制)
- hard nofile:单个用户可用的最大进程数量(硬限制)
#### vim /etc/sysctl.conf

错误2：max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
虚拟内存太小
切换到root用户修改

vim /etc/sysctl.conf

vm.max_map_count=262144
执行命令：

 sysctl -p
#### vim ES_HOME/config/jvm.options

错误3：Java HotSpot(TM) 64-Bit Server VM warning: INFO: os::commit_memory(0x0000000085330000, 2060255232, 0) failed; error=’Cannot allocate memory’ (errno=12)
jvm需要分配的内存太大
vim ES_HOME/config/jvm.options

设置 –Xmx2g和-Xms2g

推荐单机版16g
单机不得超过32G，否则会引发指针膨胀，虽然堆内存变大了，但是所能指向的实际对象会减少。

### 3.	不能使用root启动，必须创建用户

添加用户：useradd -m 用户名  然后设置密码  passwd 用户名
useradd -m admin 
passwd admin 
修改文件夹权限
chown -R  admin elasticsearch


启动可能出现的错误

### 4.	启动

ES_HOME/bin/elasticsearch -d

### 5.	无法形成集群

删除elsticsearch文件夹下的data文件夹下的节点数据
调整 discovery.zen.minimum_master_nodes: 2 （N  master节点/2）+1

### 6.	设置密码X-pack
1、	安全配置
默认情况下，拥有安全免费许可证时，Elasticsearch安全功能被禁用。 要启用安全功能，需要设置xpack.security.enabled。
在每个节点(包括node-1、node-2、node-3)的elasticsearch.yml配置文件中，新增：
xpack.security.enabled: true
2、	为节点间通信配置传输层安全性(TLS/SSL)
借助elasticsearch-certutil命令生成证书
cd ES_HOME/bin/
./elasticsearch-certutil ca -out /etc/elasticsearch/elastic-certificates.p12 -pass ""

Root用户
chown -R elasticsearch:elasticsearch   /etc/elasticsearch/elastic-certificates.p12
将证书拷贝到其他节点，放入 /etc/elasticsearch 目录下
cd /etc/elasticsearch/
scp elastic-certificates.p12  172.168.201.77:/etc/elasticsearch/
scp elastic-certificates.p12  172.168.201.78:/etc/elasticsearch/
3、	配置加密通信
启用安全功能后，必须使用TLS来确保节点之间的通信已加密。
在elasticsearch.yml中心新增配置如下：(其他节点相同配置)
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12
重启elasticsearch服务
4、	设置集群密码
因为你上面已经做了SSL通信，所以只需要在第一台es(master)上设置用户名和密码就可以了，其他的2台es就会是相同的用户名密码
[elastic@es-node1 bin]$ cd /usr/share/elasticsearch/bin
[elastic@es-node1 bin]$ ./elasticsearch-setup-passwords -h  #查看命令帮助
Sets the passwords for reserved users

Commands
--------
auto - Uses randomly generated passwords          #主要命令选项，表示系统将使用随机字符串设置密码
interactive - Uses passwords entered by a user    #主要命令选项，表示使用用户输入的字符串作为密码

Non-option arguments:
command            

Option         Description      
------         -----------      
-h, --help     show help        
-s, --silent   show minimal output
-v, --verbose  show verbose output
[elastic@es-node1 bin]$ ./elasticsearch-setup-passwords auto  #为了演示效果，这里我们使用系统自动创建
Initiating the setup of passwords for reserved users elastic,kibana,logstash_system,beats_system.
The passwords will be randomly generated and printed to the console.
Please confirm that you would like to continue [y/N]y     #选择y
Changed password for user kibana                   #kibana角色和密码
PASSWORD kibana = 4VXPRYIVibyAbjugK6Ok
Changed password for user logstash_system          #logstash角色和密码
PASSWORD logstash_system = 2m4uVdSzDzpt9OEmNin5
Changed password for user beats_system             #beast角色和密码
PASSWORD beats_system = O8VOzAaD3fO6bstCGDyQ
Changed password for user elastic                  #elasticsearch角色和密码
PASSWORD elastic = 1TWVMeN8tiBy917thUxq
核心：
auto - 随机生成密码。
interactive - 自定义不同用户的密码。
附：elasticsearch-setup-passwords此脚本只能运行一次，如要修改密码可以在kibana 中设置密码


验证是否正常
http://192.168.1.2:9200

输入上一步生成的elastic和密码
  
http://192.168.1.2:9200/_cluster/health
 

http://192.168.160.23:9100/?auth_user=elastic&auth_password=123123
curl -u elastic 'http://192.168.1.2:9200/_xpack/security/_authenticate?pretty’ 

### 7.	重置密码(非必须，如忘记密码)
创建本地超级账户，然后使用api接口本地超级账户重置elastic账户的密码
(1) 停止elasticsearch服务
(2) 确保你的配置文件中支持本地账户认证支持，如果你使用的是xpack的默认配置则无需做特殊修改；如果你配置了其他认证方式则需要确保配置本地认证方式在ES_HOME/config/elasticsearch.yml中；
(3) 使用命令ES_HOME/bin/ elasticsearch-users 建一个基于本地问价认证的超级管理员
bin/elasticsearch-users useradd my_admin -p my_password -r superuser
(4) 启动elasticsearch服务
(5) 通过api重置elastic超级管理员的密码
curl -H "Content-Type:application/json" -XPOST -u my_admin 'http://192.168.1.2:9200/_xpack/security/user/elastic/_password' -d '{ "password" : "123123" }'
(6) 校验下密码是否重置成功
curl -u elastic 'http://192.168.1.2:9200/_xpack/security/_authenticate?pretty'
(7)删除my_admin账号
bin/elasticsearch-users userdel my_admin

## 附：默认集群配置：
```
http.cors.enabled: true  #跨域连接相关设置
http.cors.allow-origin: "*"  #跨域连接相关设置  
http.cors.allow-headers: Authorization,content-type

cluster.name: elasticsearch #集群的名称，同一个集群该值必须设置成相同的
node.name: master #该节点的名字
node.master: true #该节点有机会成为master节点
node.data: true #该节点可以存储数据

#network.bind_host: 0.0.0.0 #设置绑定的IP地址，可以是IPV4或者IPV6
#network.publish_host: 192.168.1.2 #设置其他节点与该节点交互的IP地址
network.host: 192.168.1.2 #该参数用于同时设置bind_host和publish_host

http.port: 9200 #设置对外服务的http端口号
http.max_content_length: 100mb #设置http内容的最大大小
http.enabled: true #是否开启http服务对外提供服务
transport.tcp.port: 9300 #设置节点之间交互的端口号
transport.tcp.compress: true #设置是否压缩tcp上交互传输的数据

cluster.initial_master_nodes: ["master"]
discovery.zen.minimum_master_nodes: 2 #设置这个参数来保证集群中的节点可以知道其它N个有master资格的节点。官方推荐（N/2）+1
discovery.zen.ping_timeout: 120s #设置集群中自动发现其他节点时ping连接的超时时间
discovery.zen.ping.unicast.hosts: ["192.168.1.2:9300","192.168.1.2:9500","192.168.1.2:9700"] #设置集群中的Master节点的初始列表，可以通过这些节点来自动发现其他新加入集群的节点
#discovery.zen.ping.unicast.hosts 使用network.host. transport.tcp.port

#开启x-pack安全验证
xpack.security.enabled: true
xpack.license.self_generated.type: basic
##如果是basic license的话需要加入下面这一行，不然的话restart elasticsearch之后会报错。
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12
```