# Spring命名空间

Spring在提供了使用XML来装配Bean的方案，虽然现在基于XML的装配不是很推荐，但是作为Spring知识的一部分，这里对其相关的知识做一个简单的补充；

Spring提供了使用XML来装配Bean的方案，相关的知识点有：

xml文件实例：

web.xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<web-app version="3.0"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
	http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd">
    <display-name>Archetype Created Web Application</display-name>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
            <!--WEB-INF/spring-greenplum.xml-->
            classpath:spring-greenplum.xml
            classpath:spring-uim-context.xml
        </param-value>
    </context-param>
    <context-param>
        <param-name>webAppRootKey</param-name>
        <param-value>ops_server</param-value>
    </context-param>
    <context-param>
        <param-name>log4jConfigLocation</param-name>
        <param-value>classpath:log4j.properties</param-value>
    </context-param>
    
    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>*.jpg</url-pattern>
    </servlet-mapping>
    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>*.svg</url-pattern>
    </servlet-mapping>
    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>*.png</url-pattern>
    </servlet-mapping>
    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>*.ttf</url-pattern>
    </servlet-mapping>
    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>*.woff</url-pattern>
    </servlet-mapping>
    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>*.woff2</url-pattern>
    </servlet-mapping>
    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>*.js</url-pattern>
    </servlet-mapping>
    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>*.json</url-pattern>
    </servlet-mapping>
    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>*.css</url-pattern>
    </servlet-mapping>
    <servlet-mapping>
        <servlet-name>default</servlet-name>
        <url-pattern>*.html</url-pattern>
    </servlet-mapping>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <listener>
        <listener-class>org.springframework.web.util.Log4jConfigListener</listener-class>
    </listener>

    <servlet>
        <servlet-name>mvc-dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:mvc-dispatcher-servlet.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>mvc-dispatcher</servlet-name>
        <url-pattern>/api/*</url-pattern>
    </servlet-mapping>

    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/</url-pattern>
    </filter-mapping>
    <!--cors跨域-->
    <filter>
        <filter-name>cors</filter-name>
        <filter-class>com.dtdream.dthink.gp.ops.filter.CorsFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>cors</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <welcome-file-list>
        <welcome-file>index.html</welcome-file>
    </welcome-file-list>

    <session-config>
        <session-timeout>60</session-timeout>
    </session-config>

</web-app>
```

mvc-dispatcher-servlet.xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">


    <mvc:annotation-driven />

    <context:component-scan base-package="com.dtdream.dthink.gp.ops"></context:component-scan>

    <mvc:interceptors>
        <mvc:interceptor>
            <mvc:mapping path="/**"/>
            <bean class="com.dtdream.dthink.gp.ops.interceptor.LoginInterceptor"></bean>
        </mvc:interceptor>
    </mvc:interceptors>
</beans>
```

spring-greenplum.xml:

```xml
<?xml version="2.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util" xmlns:task="http://www.springframework.org/schema/task"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                                 http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
                                 http://www.springframework.org/schema/context
                                 http://www.springframework.org/schema/context/spring-context-4.0.xsd
                                 http://www.springframework.org/schema/util
                                 http://www.springframework.org/schema/util/spring-util-3.0.xsd http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task.xsd">


    <context:component-scan base-package="com.dtdream.dthink.gp.ops.*"/>

    <context:property-placeholder location="classpath:config.properties" ignore-unresolvable="true"/>

    <util:properties id="properties" location="classpath:config.properties"/>

    <bean id="ambariHost" class="com.dtdream.dthink.gp.ops.model.AmbariHost">
        <property name="ambariBaseUrl" value="${ambariBaseUrl}" />
        <property name="ambariUsername" value="${ambariUsername}" />
        <property name="ambariPassword" value="${ambariPassword}" />
        <property name="clusterName" value="${clusterName}" />
    </bean>

    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass">
        <!--<value>com.mysql.jdbc.Driver</value>-->
        <value>org.mariadb.jdbc.Driver</value>
    </property>
    <property name="jdbcUrl">
        <!--<value>jdbc:mysql://192.168.181.215:3306/gpdms</value>-->
        <value>jdbc:mariadb://192.168.181.215/gpops</value>
    </property>
    <property name="user">
        <value>root</value>
    </property>
    <property name="password">
        <value>DtDream0209</value>
    </property>
    <!--连接池中保留的最小连接数。-->
    <property name="minPoolSize">
        <value>1</value>
    </property>
    <!--连接池中保留的最大连接数。Default: 15 -->
    <property name="maxPoolSize">
        <value>20</value>
    </property>
    <!--初始化时获取的连接数，取值应在minPoolSize与maxPoolSize之间。Default: 3 -->
    <property name="initialPoolSize">
        <value>1</value>
    </property>
    <!--最大空闲时间,60秒内未使用则连接被丢弃。若为0则永不丢弃。Default: 0 -->
    <property name="maxIdleTime">
        <value>60</value>
    </property>
    <!--当连接池中的连接耗尽的时候c3p0一次同时获取的连接数。Default: 3 -->
    <property name="acquireIncrement">
        <value>10</value>
    </property>
    <!--JDBC的标准参数，用以控制数据源内加载的PreparedStatements数量。
    但由于预缓存的statements属于单个connection而不是整个连接池。
    所以设置这个参数需要考虑到多方面的因素。
    如果maxStatements与maxStatementsPerConnection均为0，则缓存被关闭。Default: 0-->
    <property name="maxStatements">
        <value>0</value>
    </property>
    <!--每60秒检查所有连接池中的空闲连接。Default: 0 -->
    <property name="idleConnectionTestPeriod">
        <value>60</value>
    </property>
    <!--定义在从数据库获取新连接失败后重复尝试的次数。Default: 30 -->
    <property name="acquireRetryAttempts">
        <value>30</value>
    </property>
    <!--获取连接失败将会引起所有等待连接池来获取连接的线程抛出异常。但是数据源仍有效保留，
    并在下次调用getConnection()的时候继续尝试获取连接。
    如果设为true，那么在尝试获取连接失败后该数据源将申明已断开并永久关闭。
    Default: false-->
    <property name="breakAfterAcquireFailure">
        <value>false</value>
    </property>
    <!--因性能消耗大请只在需要的时候使用它。
    如果设为true那么在每个connection提交的时候都将校验其有效性。
    建议使用idleConnectionTestPeriod或automaticTestTable等方法来提升连接测试的性能。
    Default: false -->
    <property name="testConnectionOnCheckout">
        <value>true</value>
    </property>
    </bean>

    <bean id="gpDataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${gp.driverClass}" />
        <property name="jdbcUrl" value="${gp.url}" />
        <property name="user" value="${gp.user}" />
        <property name="password" value="${gp.password}" />
        <!--连接池中保留的最小连接数。-->
        <property name="minPoolSize" value="${gp.minPoolSize}" />
        <!--连接池中保留的最大连接数。Default: 15 -->
        <property name="maxPoolSize" value="${gp.maxPoolSize}" />
        <!--初始化时获取的连接数，取值应在minPoolSize与maxPoolSize之间。Default: 3 -->
        <property name="initialPoolSize" value="${gp.initialPoolSize}" />
        <!--最大空闲时间,60秒内未使用则连接被丢弃。若为0则永不丢弃。Default: 0 -->
        <property name="maxIdleTime" value="${gp.maxIdleTime}" />
        <!--当连接池中的连接耗尽的时候c3p0一次同时获取的连接数。Default: 3 -->
        <property name="acquireIncrement" value="${gp.acquireIncrement}" />
        <!--JDBC的标准参数，用以控制数据源内加载的PreparedStatements数量。
        但由于预缓存的statements属于单个connection而不是整个连接池。
        所以设置这个参数需要考虑到多方面的因素。
        如果maxStatements与maxStatementsPerConnection均为0，则缓存被关闭。Default: 0-->
        <property name="maxStatements" value="${gp.maxStatements}" />
        <!--每60秒检查所有连接池中的空闲连接。Default: 0 -->
        <property name="idleConnectionTestPeriod" value="${gp.idleConnectionTestPeriod}" />
        <!--定义在从数据库获取新连接失败后重复尝试的次数。Default: 30 -->
        <property name="acquireRetryAttempts" value="${gp.acquireRetryAttempts}" />
        <!--获取连接失败将会引起所有等待连接池来获取连接的线程抛出异常。但是数据源仍有效保留，
        并在下次调用getConnection()的时候继续尝试获取连接。
        如果设为true，那么在尝试获取连接失败后该数据源将申明已断开并永久关闭。
        Default: false-->
        <property name="breakAfterAcquireFailure" value="${gp.breakAfterAcquireFailure}" />
        <!--因性能消耗大请只在需要的时候使用它。
        如果设为true那么在每个connection提交的时候都将校验其有效性。
        建议使用idleConnectionTestPeriod或automaticTestTable等方法来提升连接测试的性能。
        Default: false -->
        <property name="testConnectionOnCheckout" value="${gp.testConnectionOnCheckout}" />
    </bean>


    <task:annotation-driven />
    <bean id="segmentRecoveryTimer" class="com.dtdream.dthink.gp.ops.timer.SegmentRecovery"></bean>
</beans>
```

spring-uim-context.xml:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                                 http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
                                 http://www.springframework.org/schema/context
                                 http://www.springframework.org/schema/context/spring-context-4.0.xsd">

    <context:property-placeholder location="classpath:uim-security.properties" file-encoding="UTF-8" />

    <bean id="uimClient" class="com.dtdream.uim.sdk.utils.UimClient">
        <property name="url" value="${uim.server}"></property>
        <property name="accessKeyId" value="${adb.accesskey.id}"></property>
        <property name="accessKeySecret" value="${adb.accesskey.secret}"></property>
    </bean>

    <bean id="userConfigLoader" class="com.dtdream.uim.sdk.utils.UserConfigLoader">
        <property name="serviceName" value="${uim.serviceName}"></property>
    </bean>

    <!-- 自动扫描依赖的用户管理SDK -->
    <context:component-scan base-package="com.dtdream.uim"/>
    <context:component-scan base-package="com.dtdream.dthink.dtalent.dmall" />

</beans>
```

config.properties:

```properties
#ADB的master节点，用于执行各项脚本
#远程主机账户
remote.shell.hostIp=192.168.181.215
#远程登录账户
remote.shell.userName=root
#远程登录密码
remote.shell.password=DtDream0209

#gp数据源
gp.driverClass=org.postgresql.Driver
gp.url=jdbc:postgresql://192.168.181.215/postgres
gp.user=gpadmin
gp.password=123456
gp.minPoolSize=10
gp.maxPoolSize=100
gp.initialPoolSize=20
gp.maxIdleTime=60
gp.acquireIncrement=10
gp.maxStatements=0
gp.idleConnectionTestPeriod=60
gp.acquireRetryAttempts=30
gp.breakAfterAcquireFailure=false
gp.testConnectionOnCheckout=true

#mysql数据源
mysql.driverClass=org.mariadb.jdbc.Driver
mysql.url=jdbc:mariadb://192.168.181.215/gpops
mysql.user=root
mysql.password=123456
mysql.minPoolSize=10
mysql.maxPoolSize=100
mysql.initialPoolSize=20
mysql.maxIdleTime=60
mysql.acquireIncrement=10
mysql.maxStatements=0
mysql.idleConnectionTestPeriod=60
mysql.acquireRetryAttempts=30
mysql.breakAfterAcquireFailure=false
mysql.testConnectionOnCheckout=true

#ambari配置
ambariBaseUrl=http://192.168.181.215:8080/api/v1
ambariUsername=admin
ambariPassword=admin
clusterName=test

#adb版本号
adb.version=0.6.0

#脚本包的基础路径
adb.scripts_home=/etc/adb_scripts


#Resource Manager Configuration
#rm.baseurl=http://192.168.181.215:61505/dms-server/
rm.baseurl=http://localhost:8081/rm/api
rm.username=admin;
rm.password=test;
```

log4j.properties:

```properties
sog4j.rootLogger = info,stdout,R2

#log4j.rootLogger = on
#log4j.rootLogger = DEBUG, console, FILE
log4j.rootLogger = info,stdout,R2

log4j.logger.mysqlLog=info,mysql


#Console Log
log4j.logger.console=DEBUG,stdout
log4j.appender.stdout = org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout = org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.conversionPattern = %d [%t] %-5p %c - %m%n

log4j.appender.mysql=org.apache.log4j.jdbc.JDBCAppender
log4j.appender.mysql.driver=org.mariadb.jdbc.Driver
log4j.appender.mysql.URL=jdbc:mariadb://ADB_OPS_LOG4J_IP:3306/gpops?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true
log4j.appender.mysql.user=ADB_OPS_LOG4J_USER
log4j.appender.mysql.password=ADB_OPS_LOG4J_PWD
log4j.appender.mysql.sql=insert into ops_log(user_name,user_id,start_time,msg) VALUES ('%X{userName}','%X{userId}','%d{yyyy-MM-dd HH:mm:ss}','%m')
log4j.appender.mysql.layout=org.apache.log4j.PatternLayout

log4j.appender.R2 = org.apache.log4j.RollingFileAppender
log4j.appender.R2.File = ${catalina.home}/logs/my_log.log
log4j.appender.R2.MaxFileSize=1GB
log4j.appender.R2.MaxBackupIndex=30
log4j.appender.R2.layout=org.apache.log4j.PatternLayout
log4j.appender.R2.layout.ConversionPattern=%d [%t] %-5p %c - %m%n
```

uim-security.properties:

```properties
###
# war包依赖的外部服务位置
###

#本服务所在的IP（如果有LB或HA服务，这里填写虚IP或域名）
web.server.host=http://192.168.181.215:61505/ops_front

# UIM（用户中心服务）
uim.server = http://192.168.181.219:80/uim
adb.accesskey.id=adb
adb.accesskey.secret=adb-secret
uim.serviceName=云平台

# 二、IP或者域名部署方式，二者选其一
#网页客户端与cas服务连接的地址、协议和端口
cas.server.protocol = https
cas.server.host = 192.168.181.220
cas.server.port = 443
#应用与cas服务连接的地址、协议和端口
cas.server.protocol.inner = https
cas.server.host.inner = 192.168.181.220
cas.server.port.inner = 443

# 三、生成最终设置(不需要修改)
cas.server.login=${cas.server.protocol}://${cas.server.host}:${cas.server.port}/cas/login
cas.server.logout=${cas.server.protocol}://${cas.server.host}:${cas.server.port}/cas/logout
cas.server.validator=${cas.server.protocol.inner}://${cas.server.host.inner}:${cas.server.port.inner}/cas
```

这里提一下Spring中的@Configuration注解，一般的解释为：@Configuration标注在类上，相当于把该类作为spring的xml配置文件中的`<beans>`（注意是beans，不是bean）作用为：配置spring容器(应用上下文)，在我们的配置中就相当于把上面的spring-greenplum.xml 和 spring-uim-context.xml以java 类的形式声明出来；

参考文献：http://www.cnblogs.com/jianyungsun/p/6680962.html

里面有对这个的比较明了的解释，不理解可以再回头看一下这篇文档

---

### 1. 创建XML配置规范

就是XML文件头部那一大段复杂的配置规范说明：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd">
```

>  Spring Tool Suite 工具可以简化xml文件的配置；

这个配置规范有的配饰是引入了命名空间的概念：引入了命名空间就能够使用形命名空间所设定的方式简化很多xml中的配置；

像 context、util等，这些都是命名空间的声明，然后在xml中就可以利用这些命名空间来配置xml的内容；

后面会给出几个xml详细的配置文件；

---

### 2. 声明一个Bean

```xml
    <bean id="ambariHost" class="com.dtdream.dthink.gp.ops.model.AmbariHost">
        <property name="ambariBaseUrl" value="${ambariBaseUrl}" />
        <property name="ambariUsername" value="${ambariUsername}" />
        <property name="ambariPassword" value="${ambariPassword}" />
        <property name="clusterName" value="${clusterName}" />
    </bean>
```

---

### 3. 初始化Bean

#### 3.1 借助够构造器注入初始化Bean



知识点：

> * 构造器注入（详见Spring In Action 4）
> * 设置属性（属性注入）
> * c:命名空间；
> * p:命名空间 属性注入，设置属性
> * util:命名空间；将一些集合元素创建成一个bean：



* 此时可以通过比较笨重的方式配置，也可以使用spring中的c:命名空间来装配，


* Spring In Action 中提到了能够将字面量装配进Bean，也能够将集合装配进Bean，在使用集合的时候可以使用util:命名空间，其作者用就是将集合元素：list、map，set、properties等合租昂配成bean：

  | 元素                      | 描述                                  |
  | ----------------------- | ----------------------------------- |
  | \<util:constant\>       | 引用某个类型的public static域，并将其暴露为bean    |
  | \<util:list\>           | 创建一个java.util.list 类型的bean，其中包含值或引用 |
  | \<util:map\>            | 创建一个java.util.map类型的bean，其中包含值和引用   |
  | \<util:properties\>     | 创建一个java.util.Properties类型的bean；    |
  | \<util:propertiy-path\> | 引用一个bean的属性（或内嵌属性），并将其暴露为bean       |
  | \<util:set\>            | 创建一个java.util.Set类型的bean，其中包含值或引用   |

---

## 导入和混合配置

知识点：

> * 在JavaConfig中引用XML配置
> * 在XML配置中引入JavaConfig
> * keywords： @Import，@ImportResource