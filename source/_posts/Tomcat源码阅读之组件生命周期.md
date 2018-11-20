---
title: Tomcat源码阅读之组件生命周期
date: 2018-04-16 10:39:09
tags: tomcat
categories: java
description: "研究tomcat系列,组件之间是怎么组合的，生命周期"
---


![start&stop](/Tomcat源码阅读之组件生命周期/Lifecycle.PNG)

上篇说道Catalina中的load 和init 方法,提到了getServer().init(),和getServer().start(),这两个方法，本文主要通过这两个方法，进一步研究组件之间的关系和各种状态，希望先看一下server.xml配置文件中的组件关系图。

需要先了解一个知识点digester，讲xml文件转换成java对象，
常用的几个方法

- digester.addObjectCreate("Server","org.apache.catalina.core.StandardServer","className");
- digester.addSetProperties("Server");
- digester.addSetNext("Server","setServer","org.apache.catalina.Server");



## Catalina中的load方法

重点代码

```
Digester digester = createStartDigester();
....
....
....
file = configFile();
inputStream = new FileInputStream(file);
inputSource = new InputSource(file.toURI().toURL().toString());
......
......
......
inputSource.setByteStream(inputStream);
digester.push(this);
digester.parse(inputSource);
....
....
....
getServer().setCatalina(this);
getServer().setCatalinaHome(Bootstrap.getCatalinaHomeFile());
getServer().setCatalinaBase(Bootstrap.getCatalinaBaseFile());
// Stream redirection
initStreams();
// Start the new server
getServer().init();
···
···
···
log.info("Initialization processed in " + ((t2 - t1) / 1000000) + " ms");

```
createStartDigester()设置规则，找到xml节点执行对应的方法，configFile()这里就是读取具体的配置文件server.xml,`protected String configFile = "conf/server.xml";`然后就是digester把Catalina  push进来，parse进行解析xml。这里说明的是中间有一段  digester.addSetNext("Server","setServer","org.apache.catalina.Server");,这句就是执行setServer方法，对应的参数是org.apache.catalina.Server这种类型，然后上面还有一句，addObjectCreate，org.apache.catalina.core.StandardServer这个就是实际创建的对象类，这样就可以通过Catalina把Server联系起来的，之后的init()等这种方法，就都是调用的getServer()，上面这个对象了。

##  Standard其他类中的方法Server Service Connector Engine等

StandardServer.addService() 方法
```
    public void addService(Service service) {

        service.setServer(this);

        synchronized (servicesLock) {
            Service results[] = new Service[services.length + 1];
            System.arraycopy(services, 0, results, 0, services.length);
            results[services.length] = service;
            services = results;

            if (getState().isAvailable()) {
                try {
                    service.start();
                } catch (LifecycleException e) {
                    // Ignore
                }
            }

            // Report this property change to interested listeners
            support.firePropertyChange("service", null, service);
        }

    }
```

每一个Server可以包含多个Service，一样通过Catalina的digester来调用addService方法，这里维护了一个数组，这也是神奇的地方，一对多的关系是用数组来维护的，虽然说java中没有所谓的动态数组概念，但是，这里通过
``` 
Service results[] = new Service[services.length + 1];
System.arraycopy(services, 0, results, 0, services.length);
results[services.length] = service;
services = results;
```
这四句话，System.arraycopy是native的方法，比较奇特。另外这个synchronized 了一个空对象` private final Object servicesLock = new Object();`来保证对services的操作是线程安全。其他的findService,findServices,removeService等都是如此操作，remove用的是for循环。

#### 其他容器的关系调用方法

- StandardService 中的  setContainer(是一个Engine，在EngineRuleSet中)  addConnector addExecutor  digester规则在Catalina里面对应  还有一个await方法没有讲☆
- Connector 中的 addSslHostConfig
- StandardEngine 中的 addChild(HostRuleSet)只添加Host、setCluster、addValve digester规则在EngineRuleSet里面对应
- StandardHost 中的 addChild只添加Host，setCluster、addValve  digester规则在HostRuleSet里面对应
等等之类的调用关系，都在digester中

## Lifecycle类

通过上面的init(),找到StandredServer,可发现这没有init方法，继续`public final class StandardServer extends LifecycleMBeanBase implements Server `，`public abstract class LifecycleMBeanBase extends LifecycleBase implements JmxEnabled `,`public abstract class LifecycleBase implements Lifecycle `,最终我们在这个抽象类LifecycleBase中发现了这个方法.然后就发现，这个init里面中会调用一个`initInternal`方法，这个在LifecycleBase中是一个抽象方法，这些集成了它的类都重写了，因此我们在getServer.init(),实际上就相当于调用initInternal。


### 模板方法

org.apache.catalina.LifecycleState  和 org.apache.catalina.Lifecycle 看最上面的图(此图也是在Lifecycle内)：
```
NEW(false, null),
INITIALIZING(false, Lifecycle.BEFORE_INIT_EVENT),
INITIALIZED(false, Lifecycle.AFTER_INIT_EVENT),
STARTING_PREP(false, Lifecycle.BEFORE_START_EVENT),
STARTING(true, Lifecycle.START_EVENT),
STARTED(true, Lifecycle.AFTER_START_EVENT),
STOPPING_PREP(true, Lifecycle.BEFORE_STOP_EVENT),
STOPPING(false, Lifecycle.STOP_EVENT),
STOPPED(false, Lifecycle.AFTER_STOP_EVENT),
DESTROYING(false, Lifecycle.BEFORE_DESTROY_EVENT),
DESTROYED(false, Lifecycle.AFTER_DESTROY_EVENT),
FAILED(false, null);
```
12种状态: new(1)  init(2)  start(3)  stop(3)  destroy(2)  failed(1)
Lifecycle有四个基本的方法，init start stop destroy 外加一个addLifecycleListener()事件监听
`public abstract class LifecycleBase implements Lifecycle` 这个org.apache.catalina.util.LifecycleBean中重写了上面四个方法(synchronized)，然后这里又
在内部加一个initInternal方法调用,这里就用到了模板方法，在调用init的时候，前后做一些操作，判断当前状态啊，日志啊，等等。
另外这里又有一个方法fireLifecycleEvent，触发生命周期事件。

![start&stop](/Tomcat源码阅读之组件生命周期/LifecycleBaseinit.PNG)

一般所有的组件是实现的org.apache.catalina.util.LifecycleMBeanBase这个BaseBean的生命周期，重写了initInternal，然后在使用组件的时候对生命周期做得操作，init等就直接调用LifecycleMBeanBase，又调用本组件的initInternal.

见图StandardService中的initInternal方法:
![start&stop](/Tomcat源码阅读之组件生命周期/StandardService.PNG)


                             
系列文章

- [Tomcat源码阅读之从server.xml看组件关系](http://muxiaobai.github.io/2018/04/16/Tomcat%E6%BA%90%E7%A0%81%E9%98%85%E8%AF%BB%E4%B9%8B%E4%BB%8Eserver-xml%E7%9C%8B%E7%BB%84%E4%BB%B6%E5%85%B3%E7%B3%BB/)
- [Tomcat源码阅读之组件生命周期](http://muxiaobai.github.io/2018/04/16/Tomcat%E6%BA%90%E7%A0%81%E9%98%85%E8%AF%BB%E4%B9%8B%E7%BB%84%E4%BB%B6%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F/)
- [Tomcat源码阅读之URL请求解析](http://muxiaobai.github.io/2018/04/17/Tomcat%E6%BA%90%E7%A0%81%E9%98%85%E8%AF%BB%E4%B9%8BURL%E8%AF%B7%E6%B1%82%E8%A7%A3%E6%9E%90/)
- [Tomcat源码阅读之Container责任链](https://muxiaobai.github.io/2018/04/20/Tomcat%E6%BA%90%E7%A0%81%E9%98%85%E8%AF%BB%E4%B9%8BContainer%E8%B4%A3%E4%BB%BB%E9%93%BE/)

参看文献:
- 《深入剖析Tomcat》
- [Tomcat 系统架构与设计模式](https://www.ibm.com/developerworks/cn/java/j-lo-tomcat1/)
- [tomcat8.5.30源码](http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-8/v8.5.30/src/apache-tomcat-8.5.30-src.zip)
- [手写一个简化版Tomcat](https://my.oschina.net/liughDevelop/blog/1790893#comment-list)