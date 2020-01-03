---
title: SpringBoot之https请求
date: 2020-01-02 14:26:33
tags: 
categories: java
description: "https和http，实际上和之前我们配置tomcat 的https一个道理"
---


## 生成证书

```
keytool -genkey -alias spring -keypass 123456 -keyalg RSA -keysize 1024 -validity 365 -keystore E:/spring.keystore -storepass 123456

```

```
-genkey      在用户主目录中创建一个默认文件".keystore",还会产生一个mykey的别名，mykey中包含用户的公钥、私钥和证书
(在没有指定生成位置的情况下,keystore会存在用户系统默认目录，如：对于window xp系统，会生成在系统的C:/Documents and Settings/UserName/文件名为“.keystore”)
-alias       产生别名
-keystore    指定密钥库的名称(产生的各类信息将不在.keystore文件中)
-keyalg      指定密钥的算法 (如 RSA  DSA（如果不指定默认采用DSA）)
-validity    指定创建的证书有效期多少天
-keysize     指定密钥长度
-storepass   指定密钥库的密码(获取keystore信息所需的密码)
-keypass     指定别名条目的密码(私钥的密码)
-dname       指定证书拥有者信息 例如：  "CN=名字与姓氏,OU=组织单位名称,O=组织名称,L=城市或区域名称,ST=州或省份名称,C=单位的两字母国家代码"
-list        显示密钥库中的证书信息      keytool -list -v -keystore 指定keystore -storepass 密码
-v           显示密钥库中的证书详细信息
-export      将别名指定的证书导出到文件  keytool -export -alias 需要导出的别名 -keystore 指定keystore -file 指定导出的证书位置及证书名称 -storepass 密码
-file        参数指定导出到文件的文件名
-delete      删除密钥库中某条目          keytool -delete -alias 指定需删除的别  -keystore 指定keystore  -storepass 密码
-printcert   查看导出的证书信息          keytool -printcert -file yushan.crt
-keypasswd   修改密钥库中指定条目口令    keytool -keypasswd -alias 需修改的别名 -keypass 旧密码 -new  新密码  -storepass keystore密码  -keystore sage
-storepasswd 修改keystore口令      keytool -storepasswd -keystore e:/yushan.keystore(需修改口令的keystore) -storepass 123456(原始密码) -new yushan(新密码)
-import      将已签名数字证书导入密钥库  keytool -import -alias 指定导入条目的别名 -keystore 指定keystore -file 需导入的证书
```

默认参数：

下面是各选项的缺省值。 
-alias "mykey"

-keyalg "DSA"

-keysize 1024

-validity 90

-keystore 用户宿主目录中名为 .keystore 的文件

-file 读时为标准输入，写时为标准输

## 修改yaml配置文件
把E盘下的spring.keystore证书文件拷贝到项目中的resources目录中 , 然后在application.yml中配置
```
server:
  ssl:
    key-alias: spring
    key-password: 123456
    key-store: classpath:spring.keystore
```
和生成的证书的参数对应，

```
server:
  port: 8443
  servlet:
    context-path: /
  ssl:
    key-store: classpath:spring.keystore
    key-password: 123456
    key-alias: spring
    key-store-type:
    key-store-password:
    key-store-provider:
http:
  port: 8080
```



## 添加http和https同时监听
```
import org.apache.catalina.Context;
import org.apache.catalina.connector.Connector;
import org.apache.tomcat.util.descriptor.web.SecurityCollection;
import org.apache.tomcat.util.descriptor.web.SecurityConstraint;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @date 2020/1/2/002 13:52
 * @TODO Http支持
 */
@Configuration
public class TomcatConfig {
    @Bean
    TomcatServletWebServerFactory tomcatServletWebServerFactory() {
        TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory(){
            @Override
            protected void postProcessContext(Context context) {
                SecurityConstraint constraint = new SecurityConstraint();
                constraint.setUserConstraint("CONFIDENTIAL");
                SecurityCollection collection = new SecurityCollection();
                collection.addPattern("/*");
                constraint.addCollection(collection);
                context.addConstraint(constraint);
            }
        };
        factory.addAdditionalTomcatConnectors(createTomcatConnector());
        return factory;
    }
    private Connector createTomcatConnector() {
        // 默认协议为 TomcatServletWebServerFactory.DEFAULT_PROTOCOL=org.apache.coyote.http11.Http11NioProtocol
        Connector connector = new
                Connector(TomcatServletWebServerFactory.DEFAULT_PROTOCOL);
        connector.setScheme("http");
        connector.setPort(port);
        connector.setSecure(false);
        connector.setRedirectPort(httpsPort);//请求8080转到8443端口
        return connector;
    }
    //8080 请求8080转到8443端口
    @Value("${http.port}")
    private Integer port;
    
    //8443
    @Value("${server.port}")
    private Integer httpsPort;
}
```


日志
```

2020-01-02 14:17:43.303  INFO 10076 --- [           main] o.s.cloud.commons.util.InetUtils         : Cannot determine local hostname
2020-01-02 14:17:44.394  INFO 10076 --- [           main] o.s.cloud.commons.util.InetUtils         : Cannot determine local hostname
2020-01-02 14:17:44.629  INFO 10076 --- [           main] o.s.c.n.eureka.InstanceInfoFactory       : Setting initial instance status as: STARTING
2020-01-02 14:17:44.667  INFO 10076 --- [           main] com.netflix.discovery.DiscoveryClient    : Initializing Eureka in region us-east-1
2020-01-02 14:17:44.719  INFO 10076 --- [           main] c.n.d.provider.DiscoveryJerseyProvider   : Using JSON encoding codec LegacyJacksonJson
2020-01-02 14:17:44.720  INFO 10076 --- [           main] c.n.d.provider.DiscoveryJerseyProvider   : Using JSON decoding codec LegacyJacksonJson
2020-01-02 14:17:44.873  INFO 10076 --- [           main] c.n.d.provider.DiscoveryJerseyProvider   : Using XML encoding codec XStreamXml
2020-01-02 14:17:44.874  INFO 10076 --- [           main] c.n.d.provider.DiscoveryJerseyProvider   : Using XML decoding codec XStreamXml
2020-01-02 14:17:45.066  INFO 10076 --- [           main] c.n.d.s.r.aws.ConfigClusterResolver      : Resolving eureka endpoints via configuration
2020-01-02 14:17:45.122  INFO 10076 --- [           main] com.netflix.discovery.DiscoveryClient    : Disable delta property : false
2020-01-02 14:17:45.123  INFO 10076 --- [           main] com.netflix.discovery.DiscoveryClient    : Single vip registry refresh property : null
2020-01-02 14:17:45.123  INFO 10076 --- [           main] com.netflix.discovery.DiscoveryClient    : Force full registry fetch : false
2020-01-02 14:17:45.123  INFO 10076 --- [           main] com.netflix.discovery.DiscoveryClient    : Application is null : false
2020-01-02 14:17:45.123  INFO 10076 --- [           main] com.netflix.discovery.DiscoveryClient    : Registered Applications size is zero : true
2020-01-02 14:17:45.123  INFO 10076 --- [           main] com.netflix.discovery.DiscoveryClient    : Application version is -1: true
2020-01-02 14:17:45.123  INFO 10076 --- [           main] com.netflix.discovery.DiscoveryClient    : Getting all instance registry info from the eureka server
2020-01-02 14:17:45.316  INFO 10076 --- [           main] com.netflix.discovery.DiscoveryClient    : The response status is 200
2020-01-02 14:17:45.319  INFO 10076 --- [           main] com.netflix.discovery.DiscoveryClient    : Starting heartbeat executor: renew interval is: 10
2020-01-02 14:17:45.322  INFO 10076 --- [           main] c.n.discovery.InstanceInfoReplicator     : InstanceInfoReplicator onDemand update allowed rate per min is 4
2020-01-02 14:17:45.327  INFO 10076 --- [           main] com.netflix.discovery.DiscoveryClient    : Discovery Client initialized at timestamp 1577945865325 with initial instances count: 5
2020-01-02 14:17:45.330  INFO 10076 --- [           main] o.s.c.n.e.s.EurekaServiceRegistry        : Registering application APP-SEARCH with eureka with status UP
2020-01-02 14:17:45.330  INFO 10076 --- [           main] com.netflix.discovery.DiscoveryClient    : Saw local status change event StatusChangeEvent [timestamp=1577945865330, current=UP, previous=STARTING]
2020-01-02 14:17:45.333  INFO 10076 --- [nfoReplicator-0] com.netflix.discovery.DiscoveryClient    : DiscoveryClient_APP-SEARCH/192.168.170.1:8443: registering service...
2020-01-02 14:17:45.375  INFO 10076 --- [nfoReplicator-0] com.netflix.discovery.DiscoveryClient    : DiscoveryClient_APP-SEARCH/192.168.170.1:8443 - registration status: 204
2020-01-02 14:17:45.398  INFO 10076 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8443 (https) 8080 (http) with context path ''
```
最终日志启动监听：
```
2020-01-02 14:17:45.398  INFO 10076 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8443 (https) 8080 (http) with context path ''
```

参考：
- [证书生成](https://blog.csdn.net/Smile__1/article/details/99848578)
- [SpringBoot配置同时支持http和https](https://blog.csdn.net/qq_36699423/article/details/93481187)
- [SpringBoot支持https](https://segmentfault.com/a/1190000020052375)
