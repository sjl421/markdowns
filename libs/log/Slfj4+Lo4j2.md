#Slf4j+Lo4j2

## slf4j简介

slf4j是一个日志服务中间层。slf4j封装了多种日志库的接口，使用slf4j后，如果要修改程序使用的日志库，只需要将对应日志库的jar放入classpath，不需要修改任何代码。slf4j为部署时更换日志库提供了灵活便利。



##log4j2简介

log4j2是一个日志库。log4j2是log4j的第二版，log4j2和log4j并不兼容。目前，log4j已经停止维护。

Log4j 2是log4j 1.x和logback的改进版，据说采用了一些新技术（无锁异步、等等），使得日志的吞吐量、性能比log4j 1.x提高10倍，并解决了一些死锁的bug，而且配置更加简单灵活。

slf4j作为日志服务中间层，将调用方和日志库隔离开，调用方不需要知道任何日志库的细节。在部署时，只需将对应日志库的jar包加入classpath，就可以使用这个日志库。 将上面例子中的classpath稍作修改，增加下面3个jar包：log4j-slf4j-impl-2.x.jar、log4j-api-2.x.jar、log4j-core-2.x.jar，移除slf4j-simple-1.7.22.jar， 就成了slf4j和log4j的HelloWorld示例。(slf4j 的详细介绍可以参考http://www.importnew.com/7450.html)

## slf4j 的配置

1. 引入依赖包

在编译时，classpath中需要加入

- slf4j-api-1.7.22.jar

在运行时，classpath需要加入

- slf4j-api-1.7.22.jar
- log4j-slf4j-impl-2.7.jar
- log4j-api-2.7.jar
- log4j-core-2.7.jar

mvn 配置文件:

```xml
   <dependencies>
        <!-- log配置：Log4j2 + Slf4j -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-api</artifactId>
            <version>2.2</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.2</version>
        </dependency>
        <dependency> <!-- 桥接：告诉Slf4j使用Log4j2 -->
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-slf4j-impl</artifactId>
            <version>2.2</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.10</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
    </dependencies>
```

2. 配置log4j2

由于slf4j依赖具体的日志类,这里配置的是slf4j和log4j2的组合,所以这里还需要配置log4j2: log4j2的默认配置文件是classpath中的log4j2.xml。在启动程序时，可以通过设置参数log4j.configurationFile的方式手动指定log4j2配置文件。(关于log4j2的配置有很多内容,这里值配置了一种比较简单的,详细的配置自行google):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="DEBUG">
    <Appenders>
        <RollingFile name="RollingFile" fileName="G:\workspace\log4j2/log4j2.log"
                     filePattern="G:\workspace\log4j2/logs/$${date:yyyy-MM}/yadou-manage-%d{MM-dd-yyyy}-%i.log.gz">
            <PatternLayout>
                <Pattern>%d %-5level [%t]%l - %msg%n</Pattern>
            </PatternLayout>
            <Policies>
                <TimeBasedTriggeringPolicy/>
                <SizeBasedTriggeringPolicy size="250 MB"/>
            </Policies>
        </RollingFile>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%highlight{%d %-5level [%t]%l - %msg%n}"/>
        </Console>
    </Appenders>
    <Loggers>
        <!--控制输出debug,不继承-->
        <Logger name="default" level="debug" additivity="false">
            <!--保存文件,只存储info级别以上的日志-->
            <AppenderRef ref="RollingFile" level="debug"/>
            <!--控制台打印指定包路径下面的debug-->
            <!--<AppenderRef ref="Console"/>-->
        </Logger>
        <!--默认info级别-->
        <Root level="debug">
            <AppenderRef ref="Console"/>
            <AppenderRef ref="RollingFile"/>
        </Root>
    </Loggers>
</Configuration>
```

## slf4j 的使用

1. 导入类-> 构造logger对象->记录日志

```java
import org.junit.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class MyLogger {
    private static final Logger logger = LoggerFactory.getLogger(MyLogger.class);

    @Test
    public void test0() {
        long beginTime = System.currentTimeMillis();
        logger.info("请求处理结束，耗时：{}毫秒", (System.currentTimeMillis() - beginTime));    //第一种用法
        logger.info("请求处理结束，耗时：" + (System.currentTimeMillis() - beginTime) + "毫秒"); //第二种用法
    }
}
```

Tips: 第一种写法是slf4j的典型用法,根据官方测试的数据，第一种用法比第二种快47倍！

## slf4j说明

1. 日志级别:

slf4j支持以下级别的日志

- trace
- debug
- info
- warn
- error

