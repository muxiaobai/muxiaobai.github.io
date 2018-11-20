---
title: Tomcat源码阅读之URL请求解析
date: 2018-04-17 10:07:03
tags: tomcat
categories: java
description: "研究tomcat系列,组件之间的请求转发，Connector怎么到了Container？ "
---

## 这个流程写的很详细
可参考这个博客的[时序图](http://hill007299.iteye.com/blog/1757198)

http://localhost:8080/examples/composite.jsp
---
- 在端口8080启动Server，并通知Service完成启动，Service通知Connector完成初始化和启动的过程
- Connector首先收到这个请求，会调用ProtocolHandler完成http协议的解析，然后交给SocketProcessor处理，解析请求头，通过ConnectionHandler，给到Http11Processor，再交给CoyoteAdapter解析请求行和请求体，并把解析信息封装到Request和Response对象中， 把请求（此时应该是Request对象，这里的Request对象已经封装了Http请求的信息）交给Container容器 
- Container容器交给其子容器——Engine容器，并等待Engine容器的处理结果 Engine容器匹配其所有的虚拟主机，这里匹配到Host
- 请求被移交给hostname为localhost的Host容器，host匹配其所有子容器Context，这里找到contextPath为/examples的Context容器。如果匹配不到就把该请求交给路径名为”“的Context去处理
- 请求再次被移交给Context容器，Context继续匹配其子容器Wrapper，由Wrapper容器加载composite.jsp对应的servlet，这里编译的servlet是basic_002dcomparisons_jsp.class文件
- Context容器根据后缀匹配原则*.jsp找到composite.jsp编译的java类的class文件
- Connector构建一个org.apache.catalina.connector.Request以及org.apache.catalina.connector.Response对象，使用反射调用Servelt的service方法
- Context容器把封装了响应消息的Response对象返回给Host容器
- 容器把Response返回给Engine容器
- Engine容器返回给Connector
- Connetor容器把Response返回给浏览器
- 浏览器解析Response报文
- 显示资源内容


![流程主要代码](/Tomcat源码阅读之URL请求解析/Connector.png)

## Connector到Container
Connector 中的init和start 都是对应service调用的参考上一篇[Tomcat组件生命周期]()

这里主要通过initInternal和startInternal来进行详细分析。

```
//代码段1
    protected String protocolHandlerClassName = "org.apache.coyote.http11.Http11NioProtocol";
    public Connector(String protocol) {
        setProtocol(protocol);
        // Instantiate protocol handler
        ProtocolHandler p = null;
        try {
            Class<?> clazz = Class.forName(protocolHandlerClassName);
            p = (ProtocolHandler) clazz.getConstructor().newInstance();
        } catch (Exception e) {
            log.error(sm.getString(
                    "coyoteConnector.protocolHandlerInstantiationFailed"), e);
        } finally {
        
            this.protocolHandler = p;//初始化的时候这里的protocolHandler是通过反射获取的Http11NioProtocol实例。
        }

        if (Globals.STRICT_SERVLET_COMPLIANCE) {
            uriCharset = StandardCharsets.ISO_8859_1;
        } else {
            uriCharset = StandardCharsets.UTF_8;
        }
    }
    @Override
    protected void initInternal() throws LifecycleException {
        super.initInternal();
        
        // Initialize adapter
        adapter = new CoyoteAdapter(this);
        protocolHandler.setAdapter(adapter);//protocolHandler到CoyoteAdapter的关联关系

      
        try {
            protocolHandler.init();//AbstractProtocol.init(),这里会调用到endpoint.bind()
        } catch (Exception e) {
            throw new LifecycleException(
                    sm.getString("coyoteConnector.protocolHandlerInitializationFailed"), e);
        }
    }


    /**
     * Begin processing requests via this Connector.
     *
     * @exception LifecycleException if a fatal startup error occurs
     */
    @Override
    protected void startInternal() throws LifecycleException {

        // Validate settings before starting
        if (getPort() < 0) {
            throw new LifecycleException(sm.getString(
                    "coyoteConnector.invalidPort", Integer.valueOf(getPort())));
        }

        setState(LifecycleState.STARTING);

        try {
            protocolHandler.start();//AbstractProtocol.start(),这里会调用到endpoint.start()
        } catch (Exception e) {
            throw new LifecycleException(
                    sm.getString("coyoteConnector.protocolHandlerStartFailed"), e);
        }
    }
   

```

Connector中主要确定要哪一个协议来处理请求，最后又交回到CoyoteAdapter中。



#### AbstractProtocol


```
 @Override
    public void init() throws Exception {
        if (getLog().isInfoEnabled()) {
            getLog().info(sm.getString("abstractProtocolHandler.init", getName()));
        }

        if (oname == null) {
            // Component not pre-registered so register it
            oname = createObjectName();
            if (oname != null) {
                Registry.getRegistry(null, null).registerComponent(this, oname, null);
            }
        }

        if (this.domain != null) {
            rgOname = new ObjectName(domain + ":type=GlobalRequestProcessor,name=" + getName());
            Registry.getRegistry(null, null).registerComponent(
                    getHandler().getGlobal(), rgOname, null);
        }

        String endpointName = getName();
        endpoint.setName(endpointName.substring(1, endpointName.length()-1));
        endpoint.setDomain(domain);

        endpoint.init();//这里在初始化Http11NioProtocol的时候有一个实例化NioEndpoint，这个就是endpoint
        ···················
        Http11NioProtocol 的构造方法
         public Http11NioProtocol() {
        super(new NioEndpoint());
        }
        ····················
        
    }


    @Override
    public void start() throws Exception {
        if (getLog().isInfoEnabled()) {
            getLog().info(sm.getString("abstractProtocolHandler.start", getName()));
        }

        endpoint.start();

        // Start async timeout thread
        asyncTimeout = new AsyncTimeout();
        Thread timeoutThread = new Thread(asyncTimeout, getNameInternal() + "-AsyncTimeout");
        int priority = endpoint.getThreadPriority();
        if (priority < Thread.MIN_PRIORITY || priority > Thread.MAX_PRIORITY) {
            priority = Thread.NORM_PRIORITY;
        }
        timeoutThread.setPriority(priority);
        timeoutThread.setDaemon(true);
        timeoutThread.start();
    }

```

此步骤已经从Connector转到了AbstractProtocol协议处理内部，然后找NioEndpoint

需要注意在Connector找AbstractProtocol的时候，
```
    public AbstractHttp11Protocol(AbstractEndpoint<S> endpoint) {
        super(endpoint);
        setConnectionTimeout(Constants.DEFAULT_CONNECTION_TIMEOUT);
        ConnectionHandler<S> cHandler = new ConnectionHandler<>(this);
        setHandler(cHandler);
        getEndpoint().setHandler(cHandler);
    }
```
这里在AbstractHttp11Protocol构造函数中设置了NioEndpoint的handler，就是AbstractProtocol中的ConnectionHandler，因此在Protocol处理完后给ConnectionHandler，然后通过getHandler调用process(),



#### AbstractEndpoint 和NioEndpoint  
首先是抽象类AbstractEndpoint中的init和start，会bind()和start()一下，这才会到NioEndpoint进行处理

AbstractEndpoint

```
    public void init() throws Exception {
        if (bindOnInit) {
            bind();
            bindState = BindState.BOUND_ON_INIT;
        }
        if (this.domain != null) {
            // Register endpoint (as ThreadPool - historical name)
            oname = new ObjectName(domain + ":type=ThreadPool,name=\"" + getName() + "\"");
            Registry.getRegistry(null, null).registerComponent(this, oname, null);

            for (SSLHostConfig sslHostConfig : findSslHostConfigs()) {
                registerJmx(sslHostConfig);
            }
        }
    }
    
    public final void start() throws Exception {
        if (bindState == BindState.UNBOUND) {
            bind();
            bindState = BindState.BOUND_ON_START;
        }
        startInternal();
    }
    public abstract void bind() throws Exception;
    public abstract void startInternal() throws Exception;
    
```
NioEndpoint

```
@Override
    public void bind() throws Exception {

        serverSock = ServerSocketChannel.open();
        socketProperties.setProperties(serverSock.socket());
        InetSocketAddress addr = (getAddress()!=null?new InetSocketAddress(getAddress(),getPort()):new InetSocketAddress(getPort()));
        serverSock.socket().bind(addr,getAcceptCount());
        serverSock.configureBlocking(true); //mimic APR behavior

        selectorPool.open();
    }

    @Override
    public void startInternal() throws Exception {

        if (!running) {
 
         // Start poller threads 这里启动了并一直在空转看内部的run()方法有一个while(true)
            pollers = new Poller[getPollerThreadCount()];
            for (int i=0; i<pollers.length; i++) {
                pollers[i] = new Poller();
                Thread pollerThread = new Thread(pollers[i], getName() + "-ClientPoller-"+i);
                pollerThread.setPriority(threadPriority);
                pollerThread.setDaemon(true);
                pollerThread.start();
            }

            startAcceptorThreads();//调用到AbstractEndpoint然后new Acceptor(),并start()
            ···············
             Acceptor.run()方法
             
               // Accept the next incoming connection from the server
             socket = serverSock.accept();这里阻塞了，等待请求
             setSocketOptions();// getPoller0().register(channel);
            ················
        }
    }
```
Acceptor的作用是控制与tomcat建立连接的数量，但Acceptor只负责建立连接。socket内容的读写是通过Poller来实现的。

setSocketOptions()是关键关联点

#### NioEndpoint到SocketProcessor到ConnectionHandler

这里应该是使用了一个注册监听。

Poller 和事件类PollerEvent
Poller 是在startInternal就启动了`pollerThread.start();`
先把channel注册到Poller并添加addEvent(),对应一个PollerEvent，然后每次Poller在run()的时候，就会events()[PollerEvent.run()],如果有事件，就会返回true，

```
Poller implements Runnable {
    public boolean events() {
        boolean result = false;
        //events.poll()看event里面还有没有事件
        PollerEvent pe = null;
        for (int i = 0, size = events.size(); i < size && (pe = events.poll()) != null; i++ ) {
            result = true;
            try {
                pe.run();
                pe.reset();
                if (running && !paused) {
                    eventCache.push(pe);
                }
            } catch ( Throwable x ) {
                log.error("",x);
            }
        }

        return result;
    }
    public void register(final NioChannel socket) {
        PollerEvent r = eventCache.pop();
        addEvent(r);
    }
    public void run() {
            // Loop until destroy() is called
            while (true) {

                boolean hasEvents = false;

                try {
                    if (!close) {
                        hasEvents = events();
                        if (wakeupCounter.getAndSet(-1) > 0) {
                            //if we are here, means we have other stuff to do
                            //do a non blocking select
                            keyCount = selector.selectNow();
                        } else {
                            keyCount = selector.select(selectorTimeout);
                        }
                        wakeupCounter.set(0);
                    }
                    if (close) {
                        events();
                        timeout(0, false);
                        try {
                            selector.close();
                        } catch (IOException ioe) {
                            log.error(sm.getString("endpoint.nio.selectorCloseFail"), ioe);
                        }
                        break;
                    }
                } catch (Throwable x) {
                    ExceptionUtils.handleThrowable(x);
                    log.error("",x);
                    continue;
                }
                //either we timed out or we woke up, process events first
                if ( keyCount == 0 ) hasEvents = (hasEvents | events());

                Iterator<SelectionKey> iterator =
                    keyCount > 0 ? selector.selectedKeys().iterator() : null;
                // Walk through the collection of ready keys and dispatch
                // any active event.
                while (iterator != null && iterator.hasNext()) {
                    SelectionKey sk = iterator.next();
                    NioSocketWrapper attachment = (NioSocketWrapper)sk.attachment();
                    // Attachment may be null if another thread has called
                    // cancelledKey()
                    if (attachment == null) {
                        iterator.remove();
                    } else {
                        iterator.remove();
                        //这样就到了继续处理的时候
                        processKey(sk, attachment);
                    }
                }//while

                //process timeouts
                timeout(keyCount,hasEvents);
            }//while

            getStopLatch().countDown();
        }
}


PollerEvent implements Runnable {
     public void run() {
           socket.getIOChannel().register(
                            socket.getPoller().getSelector(), SelectionKey.OP_READ, socketWrapper);
     }
}
```


Poller只要执行run()之后，就会依次按照下面这个步骤：
- `processKey(sk, attachment);`
    - `processSendfile(sk,attachment, false); `
- `processSocket(socketWrapper, SocketEvent.OPEN_READ, true)`
- 找到AbstractEndpoint.processSocket()
- ` sc = createSocketProcessor(socketWrapper, event); sc.run();`
- NioEndpoint.createSocketProcessor()内部`new SocketProcessor(socketWrapper, event);`
-  `state = getHandler().process(socketWrapper, SocketEvent.OPEN_READ);`


```
    @Override
    protected SocketProcessorBase<NioChannel> createSocketProcessor(
            SocketWrapperBase<NioChannel> socketWrapper, SocketEvent event) {
        return new SocketProcessor(socketWrapper, event);
    }
```

到SocketProcessor后，getHandler()就能用ConnectionHandler进而到Http11Processor处理(AbstractHttp11Protocol的构造函数定义的handler)

####  ConnectionHandler到Http11Processor


NioEndpoint调用processSocket()方法，最终还是执行SocketProcessor.doRun();这个doRun中就使用了getHandler().process(socketWrapper);此处的handler是上文中AbstractHttp11Protocol构造函数中设置的ConnectionHandler（内部类AbstractProtocol），这个类中就把SocketProcessor和Http11Processor 关联起来l，也正如这个名字所示ConnectionHandler.

AbstractProtocol.ConnectionHandler.process()方法内部
```
 processor = getProtocol().createProcessor();
 ···················
  AbstractHttp11Protocol.createProcessor();这就是上一行的创建这个处理就是Http11Processor。
  Http11Processor processor = new Http11Processor(getMaxHttpHeaderSize(),
                getAllowHostHeaderMismatch(), getRejectIllegalHeaderName(), getEndpoint(),
                getMaxTrailerSize(), allowedTrailerHeaders, getMaxExtensionSize(),
                getMaxSwallowSize(), httpUpgradeProtocols, getSendReasonPhrase());
 ····················
 processor.process(wrapper, status);
```
那么这个process就是Http11Processor的执行了，下面就是Http11Processor和CoyoteAdapter的转换。


#### Http11Processor到CoyoteAdapter到Container

上面的`processor.process(wrapper, status);`实际上是调用AbstractProcessorLight.process()-------------->内部有一个service()

就又回到了Http11Processor重写的方法service()

```
  public SocketState process(SocketWrapperBase<?> socketWrapper, SocketEvent status)
            throws IOException {

        SocketState state = SocketState.CLOSED;
        Iterator<DispatchType> dispatches = null;
        do {
                       // There may be pipe-lined data to read. If the data isn't
                    // processed now, execution will exit this loop and call
                    // release() which will recycle the processor (and input
                    // buffer) deleting any pipe-lined data. To avoid this,
                    // process it now.
                    state = service(socketWrapper);
          
        } while (state == SocketState.ASYNC_END ||
                dispatches != null && state != SocketState.CLOSED);

        return state;
    }
    
```

Http11Processor.process()-------------->AbstractProcessorLight.process()
 state = service(socketWrapper);----------->Http11Processor.service()


Http11Processor.service() :
`import org.apache.coyote.Request;`
`getAdapter().service(request, response);`

getAdapter是在Connector.init()中给的就是下面这个CoyoteAdapter


CoyoteAdapter.service()方法中有代码：
这里构造request，response

``` 
request = connector.createRequest();
request.setCoyoteRequest(req);
response = connector.createResponse();
response.setCoyoteResponse(res);
connector.getService().getContainer().getPipeline().getFirst().invoke(request, response);

```
`org.apache.catalina.connector.Request`这个req.res都有了，然后就是找到具体的请求处理模块并返回。

这样就找到了Container，实际上到这里已经完成了Connector到Container的转换。

[百度脑图Connector](http://naotu.baidu.com/file/3a08dd05ba3011c349941c95a4814be4?token=1ab842bbabc206c6)

## Container内部进行责任链处理

#### Engine Host Context Wrapper

见下篇"Tomcat源码阅读之Container责任链"
                             
系列文章

- [Tomcat源码阅读之从server.xml看组件关系](http://muxiaobai.github.io/2018/04/16/Tomcat%E6%BA%90%E7%A0%81%E9%98%85%E8%AF%BB%E4%B9%8B%E4%BB%8Eserver-xml%E7%9C%8B%E7%BB%84%E4%BB%B6%E5%85%B3%E7%B3%BB/)
- [Tomcat源码阅读之组件生命周期](http://muxiaobai.github.io/2018/04/16/Tomcat%E6%BA%90%E7%A0%81%E9%98%85%E8%AF%BB%E4%B9%8B%E7%BB%84%E4%BB%B6%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F/)
- [Tomcat源码阅读之URL请求解析](http://muxiaobai.github.io/2018/04/17/Tomcat%E6%BA%90%E7%A0%81%E9%98%85%E8%AF%BB%E4%B9%8BURL%E8%AF%B7%E6%B1%82%E8%A7%A3%E6%9E%90/)
- [Tomcat源码阅读之Container责任链](https://muxiaobai.github.io/2018/04/20/Tomcat%E6%BA%90%E7%A0%81%E9%98%85%E8%AF%BB%E4%B9%8BContainer%E8%B4%A3%E4%BB%BB%E9%93%BE/)

参看文献：

- [Poller](https://blog.csdn.net/yanlinwang/article/details/46382889)
- [随笔分类 - Tomcat](http://www.cnblogs.com/coldridgeValley/category/797239.html)
- [Tomcat中的设计模式](https://www.cnblogs.com/coldridgeValley/p/6606271.html)
- 《深入剖析Tomcat》
- [Tomcat 系统架构与设计模式](https://www.ibm.com/developerworks/cn/java/j-lo-tomcat1/)
- [tomcat8.5.30源码](http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-8/v8.5.30/src/apache-tomcat-8.5.30-src.zip)
- [手写一个简化版Tomcat](https://my.oschina.net/liughDevelop/blog/1790893#comment-list)
- [Tomcat对HTTP请求的处理(二)](http://www.cnblogs.com/coldridgeValley/p/6252781.html)
- [请求流程](https://blog.csdn.net/u011116672/article/details/50994038)