---
title: 数据库监控-Druid监控配置
date: 2018-11-22 02:22:41
tags: [连接池]
categories: [SQL, 系统监控优化,数据库]
description: "监控Druid连接池运行情况，配合其他的Web URL Spring 等监控"
---

Druid连接池监控问题，主要包括，配置，记录慢SQL等。
<!--more-->

[Github repo](https://github.com/alibaba/druid)
[Github Druid Wiki](https://github.com/alibaba/druid/wiki/%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98)


## 添加监控页面



##  WEB应用 URI监控 Spring监控 

web.xml,放在前面的，URI监控filter，

```

 <!-- druid监控 http://host:port/druid/sql.html -->
<filter>
		<filter-name>DruidWebStatFilter</filter-name>
		<filter-class>com.alibaba.druid.support.http.WebStatFilter</filter-class>
		<init-param>
			<!-- 经常需要排除一些不必要的url，比如.js,/jslib/等等。配置在init-param中 -->
			<param-name>exclusions</param-name>
			<param-value>*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*</param-value>
		</init-param>
		<!-- 缺省sessionStatMaxCount是1000个。你可以按需要进行配置 -->
		<init-param>
			<param-name>sessionStatMaxCount</param-name>
			<param-value>1000</param-value>
		</init-param>
		<!-- druid 0.2.7版本开始支持profile，配置profileEnable能够监控单个url调用的sql列表 -->
		<init-param>
			<param-name>profileEnable</param-name>
			<param-value>true</param-value>
		</init-param>
		<init-param>
	  		<param-name>principalCookieName</param-name>
  			<param-value>userName</param-value>
  		</init-param>
		<init-param>
			<param-name>principalSessionName</param-name>
			<param-value>session_user</param-value>
		</init-param>
		<!-- 你可以关闭session统计功能 
		<init-param> 
			<param-name>sessionStatEnable</param-name> 
			<param-value>true</param-value>
		</init-param> -->
	</filter>
	<filter-mapping>
  		<filter-name>DruidWebStatFilter</filter-name>
  		<url-pattern>/*</url-pattern>
  	</filter-mapping> 
  	
	<servlet>
		<servlet-name>DruidStatView</servlet-name>
		<servlet-class>com.alibaba.druid.support.http.StatViewServlet</servlet-class>
		<!-- 
			deny优先于allow，如果在deny列表中，就算在allow列表中，也会被拒绝。
			如果allow没有配置或者为空，则允许所有访问
		 -->
		<init-param>
			<param-name>allow</param-name>
			<param-value>10.38.94.201,127.0.0.1</param-value>
		</init-param>
<!-- 		<init-param> -->
<!-- 			<param-name>deny</param-name> -->
<!-- 			<param-value>10.38.94.201</param-value> -->
<!-- 		</init-param> -->
		<!-- 在StatViewSerlvet输出的html页面中，有一个功能是Reset All，执行这个操作之后，会导致所有计数器清零，重新计数 -->
	   <span style="white-space:pre">	</span><init-param>
	        <span style="white-space:pre">	</span><param-name>resetEnable</param-name>
	        <span style="white-space:pre">	</span><param-value>false</param-value>
	    <span style="white-space:pre">	</span></init-param>
	    <span style="white-space:pre">	</span><!--  用户名和密码 -->
	    <span style="white-space:pre">	</span><init-param>
			<param-name>loginUsername</param-name>
			<param-value>druid</param-value>
		</init-param>
		<init-param>
			<param-name>loginPassword</param-name>
			<param-value>druid</param-value>
		</init-param>
	</servlet>
	<servlet-mapping>
		<servlet-name>DruidStatView</servlet-name>
		<url-pattern>/druid/*</url-pattern>
	</servlet-mapping>

```

spring.xml 监控Spring

```
	<!-- spring 监控 -->
    <bean id="druid-stat-interceptor"
          class="com.alibaba.druid.support.spring.stat.DruidStatInterceptor">
    </bean>
 
    <bean id="druid-stat-pointcut" class="org.springframework.aop.support.JdkRegexpMethodPointcut"
          scope="prototype">
        <property name="patterns">
            <list>
                <value>cn.forp.*.service.*.*(..)</value>
                <value>cn.forp.*.controller.*.*(..)</value>
            </list>
        </property>
    </bean>
 
    <aop:config>
        <aop:advisor advice-ref="druid-stat-interceptor" pointcut-ref="druid-stat-pointcut"/>
    </aop:config>


```
## 慢SQL日志记录


spring.xml的 dataSource bean

```

	<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource" init-method="init" destroy-method="close"> 

		<property name="url" value="jdbc:oracle:thin:@host:port/orcl"/>
		<property name="username" value=""/>
		<property name="password" value=""/>

		<property name="validationQuery" value="select 'x' from dual"/>

		<!-- 配置初始化大小、最小、最大 -->
		<property name="initialSize" value="2"/>
		<property name="minIdle" value="1"/> 
		<property name="maxActive" value="5"/>
		<!-- 配置获取连接等待超时的时间，单位是毫秒 -->
		<property name="maxWait" value="60000"/>
		<!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->
		<property name="timeBetweenEvictionRunsMillis" value="60000"/>
		<!-- 配置一个连接在池中最小生存的时间，单位是毫秒 -->
		<property name="minEvictableIdleTimeMillis" value="300000"/>
		<property name="testWhileIdle" value="true"/>
		<property name="testOnBorrow" value="false"/>
		<property name="testOnReturn" value="false"/>
		<!-- 打开PSCache，并且指定每个连接上PSCache的大小 -->
		<property name="poolPreparedStatements" value="true"/>
		<property name="maxPoolPreparedStatementPerConnectionSize" value="20"/>
		<!-- 启用拦截的filters：sql注入，监控统计 -->
 		<property name="filters" value="wall,stat"/>

		<property name="removeAbandoned" value="true" /> <!-- 打开removeAbandoned功能 -->
	  	<property name="removeAbandonedTimeout" value="1800" /> <!-- 1800秒，也就是30分钟 -->
  		<property name="logAbandoned" value="true" /> <!-- 关闭abanded连接时输出错误日志 -->

		<property name="proxyFilters">
            <list>
                <ref bean="stat-filter"/>
                <ref bean="log-filter"/>
            </list>
        </property>
	</bean>
	    <!-- 慢SQL记录 -->
    <bean id="stat-filter" class="com.alibaba.druid.filter.stat.StatFilter">
        <!-- 慢sql时间设置,即执行时间大于200毫秒的都是慢sql -->
        <property name="slowSqlMillis" value="200"/>
        <property name="logSlowSql" value="true"/>
    </bean>
 
    <bean id="log-filter" class="com.alibaba.druid.filter.logging.Log4jFilter">
        <property name="dataSourceLogEnabled" value="true" />
        <property name="statementExecutableSqlLogEnable" value="true" />
    </bean>


```

log4j.properties

```
# ROOT Logger
log4j.rootLogger = INFO, console, druid

# log4j.category.org.springframework = DEBUG
log4j.category.cn.forp = DEBUG

# Console
log4j.appender.console = org.apache.log4j.ConsoleAppender
log4j.appender.console.layout = org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern = [%d{yyyy-MM-dd HH:mm:ss}] %5p %c{1} %m%n

# File
log4j.appender.file = org.apache.log4j.DailyRollingFileAppender
log4j.appender.file.file = ${catalina.home}/logs/forp-pitaya.log
log4j.appender.file.DatePattern = '.'yyyy-MM-dd
log4j.appender.file.layout = org.apache.log4j.PatternLayout
log4j.appender.file.layout.conversionPattern = [%d{yyyy-MM-dd HH:mm:ss}] %5p %c{1} %m%n
log4j.appender.file.append =false

# Druid
log4j.logger.druid.sql=WARN,druid
log4j.logger.druid.sql.DataSource=WARN,druid
log4j.logger.druid.sql.Connection=WARN,druid
log4j.logger.druid.sql.Statement=WARN,druid

log4j.appender.druid=org.apache.log4j.DailyRollingFileAppender
log4j.appender.druid.layout=org.apache.log4j.PatternLayout
log4j.appender.druid.layout.ConversionPattern= [%d{yyyy-MM-dd HH\:mm\:ss}] %c{1} - %m%n
log4j.appender.druid.datePattern='.'yyyy-MM-dd
log4j.appender.druid.Threshold = WARN
log4j.appender.druid.append=true
log4j.appender.druid.File=${catalina.home}/logs/druid-slow-sql.log


```


参考：

- [使用Druid监控SQL执行状态](https://my.oschina.net/wangmengjun/blog/788386)
- [druid监控及慢sql记录](https://blog.csdn.net/haiyang4988/article/details/73740700)
- [Druid Monitor监控JavaSE和JavaWeb](https://blog.csdn.net/binglovezi/article/details/50610269#)
- [JavaSE 监控](https://github.com/alibaba/druid/blob/master/src/main/scripts/druidStat.sh)