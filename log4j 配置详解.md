# log4j 配置详解

第一步：加入log4j-1.2.8.jar到lib下。

第二步：在CLASSPATH下建立log4j.properties。内容如下：

```xml
#放在src下的话就不用配置 否则得去web.xml里面配置一个Listener
<listener>
  <listener-class>org.springframework.web.util.Log4jConfigListener</listener-class>
</listener>
```



## 详细介绍

以下是一个从watchman中拿到的一个log4j的一个配置文件。

```properties
# 定义输出目的地及日志级别
log4j.rootLogger = INFO,CONSOLE
log4j.logger.com.dtdream.watchman = INFO,myLog
 
# 输出到控制台
log4j.appender.CONSOLE = org.apache.log4j.ConsoleAppender
log4j.appender.CONSOLE.layout = org.apache.log4j.PatternLayout
log4j.appender.CONSOLE.layout.ConversionPattern =  %d{ABSOLUTE} %5p %c{1}:%L [%t:%r]- %m%n
 
# 输出到文件,自动备份，文件大小最大为50MB,备份数量最大为100
log4j.appender.myLog=org.apache.log4j.RollingFileAppender
log4j.appender.myLog.File = /var/log/watchman-server/watchman-server.log
log4j.appender.myLog.Threshold=INFO
log4j.appender.myLog.Append = true
log4j.appender.myLog.MaxFileSize=20MB
log4j.appender.myLog.MaxBackupIndex=10
log4j.appender.myLog.layout=org.apache.log4j.PatternLayout
log4j.appender.myLog.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss} [%t:%r] - [%p] [%c{1}:%L] [%M] %m%n
log4j.additivity.myLog = false
```

**1.在web.xml文件中添加**

```xml
<!-- 配置log4j --> 
 <context-param> 
 	<param-name>webAppRootKey</param-name> 
 	<param-value>com.hsinghsu.testSSH.webapp.root</param-value> 
 </context-param> 
 <context-param> 
 	<param-name>log4jConfigLocation</param-name> 
 	<param-value>/WEB-INF/classes/log4j.properties</param-value> 
 </context-param> 
 <context-param> 
 	<param-name>log4jRefreshInterval</param-name> 
 	<param-value>600000</param-value> 
 </context-param>
```

**2.添加log4j.properties文件**

```properties
log4j.rootCategory=INFO, stdout , R 
log4j.appender.stdout=org.apache.log4j.ConsoleAppender 
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout 
log4j.appender.stdout.layout.ConversionPattern=[QC] %p [%t] %C.%M(%L) | %m%n 
log4j.appender.R=org.apache.log4j.DailyRollingFileAppender 
#log4j.appender.R.File=e:\\test\\avatar.log 
## linux logs file path 
#log4j.appender.R.File=${com.hsinghsu.testSSH.webapp.root}/log/testlog.log 
## Windows logs file path 
log4j.appender.R.File=D:\\eclipsespace\\testSSH\\WebContent\\WEB-INF\\testlog.log 
log4j.appender.R.layout=org.apache.log4j.PatternLayout 
#log4j.appender.R.layout.ConversionPattern=%d-[TS] %p %t %c - %m%n 
#log4j.logger.com.neusoft=DEBUG 
#log4j.logger.com.opensymphony.oscache=ERROR 
log4j.logger.net.sf.navigator=INFO 
#log4j.logger.org.apache.commons=ERROR 
#log4j.logger.org.apache.struts=WARN 
#log4j.logger.org.displaytag=ERROR 
# 
log4j.logger.org.springframework=INFO 
# 
#log4j.logger.com.ibatis.db=WARN 
#log4j.logger.org.apache.velocity=FATAL 
#log4j.logger.com.canoo.webtest=WARN 
#log4j.logger.org.hibernate.ps.PreparedStatementCache=WARN 
#log4j.logger.org.hibernate=DEBUG 
log4j.logger.org.hibernate=INFO 
#log4j.logger.org.logicalcobwebs=WARN
```

**3.使用log4j**

例如在UserServiceImpl.java中使用log4j:

```java
package com.hsinghsu.testSSH.service.impl; 
import org.apache.log4j.Logger; 
import org.springframework.beans.factory.annotation.Autowired; 
import org.springframework.stereotype.Service; 
import com.hsinghsu.testSSH.dao.UserDao; 
import com.hsinghsu.testSSH.model.User; 
import com.hsinghsu.testSSH.service.UserService; 
@Service(value = "userService") 
public class UserServiceImpl implements UserService{ 
 @Autowired
 private UserDao userDao; 
// private final static Logger logger = Logger.getLogger(UserServiceImpl.class); 
 private Logger logger = Logger.getLogger(this.getClass().getName()); 
 public boolean login(String name, String password) 
 { 
  logger.info("--UserServiceImpl login method name:"+name+" password:"+password); 
  User user = userDao.getUserByName(name); 
  if(user!=null) 
  { 
  if(password.equals(user.getPwd())) 
  { 
   return true; 
  } 
  } 
  return false; 
 } 
}
```

## 二、log4j.properties参数详解

log的级别分为debug(调试信息)、info(一般信息)、warn(警告信息)、error(错误信息)、fatal(致命错误信息)。  
Log4j支持两种配置文件格式，一种是XML格式的文件，一种是java属性文件log4j.properties，下面以log4j.properties为例进行说明。  

**1、配置根Logger **

Logger 负责处理日志记录的大部分操作，其语法为：  

```properties
log4j.rootLogger = [ level ] , appenderName1, appenderName2, …  
```

* level : 是日志记录的优先级，分为OFF、FATAL、ERROR、WARN、INFO、DEBUG、ALL或者您定义的级别。  
* appenderName:就是指定日志信息输出目的地的名称,   如：log4j.rootLogger=info,A1,B2,C3   
* 在早期log4j版本中，org.apache.Category实现了记录器的功能，后使用logger扩展了Category类，因此log4j.rootCategory也可以使用,   如：log4j.rootCategory=INFO,A1,A2 

**2、配置日志信息输出目的地 Appender **

Appender 负责控制日志记录操作的输出，其语法为：  

```
log4j.appender.appenderName = fully.qualified.name.of.appender.class   　
```

其中`fully.qualified.name.of.appender.class` 有以下几种：  

1. org.apache.log4j.ConsoleAppender（控制台）,该选项有以下几种：  

   ```properties
   Threshold=WARN:指定日志消息的输出最低层次。  
   ImmediateFlush=true:默认值是true,意谓着所有的消息都会被立即输出。 
   Target=System.err：默认情况下是：System.out,指定输出控制台  
   ```


2. org.apache.log4j.FileAppender（文件）   该选项有以下几种：  

   ```properties
   Threshold=WARN:指定日志消息的输出最低层次。  
   ImmediateFlush=true:默认值是true,意谓着所有的消息都会被立即输出。  
   File=mylog.txt:指定消息输出到mylog.txt文件。  
   Append=false:默认值是true,即将消息增加到指定文件中，false指将消息覆盖指定的文件内容。  
   ```

3. org.apache.log4j.DailyRollingFileAppender（每天产生一个日志文件）   该选项有以下几种：  

   ```properties
   Threshold=WARN:指定日志消息的输出最低层次。  
   ImmediateFlush=true:默认值是true,意谓着所有的消息都会被立即输出。  
   File=a.log:指定消息输出到a.log文件，默认是从web服务器的根路径开始。  
   Append=false:默认值是true,即将消息增加到指定文件中，false指将消息覆盖指定的文件内容。  
   DatePattern='.'yyyy-ww:每周滚动一次文件，即每周产生一个新的文件。当然也可以指定按月、周、天、时和分。即对应的格式如下：  
     '.'yyyy-MM: 每月  
     '.'yyyy-ww: 每周  
     '.'yyyy-MM-dd: 每天
     '.'yyyy-MM-dd-a: 每天两次  
     '.'yyyy-MM-dd-HH: 每小时 
     '.'yyyy-MM-dd-HH-mm: 每分钟  
   ```

4. org.apache.log4j.RollingFileAppender（文件大小到达指定尺寸的时候产生一个新的文件，可通过log4j.appender.appenderName.MaxFileSize=100KB设置文件大小），该选项有以下几种：                     

   ```properties
   Threshold=WARN:指定日志消息的输出最低层次。  
   ImmediateFlush=true:默认值是true,意谓着所有的消息都会被立即输出。  
   File=a.log:指定消息输出到a.log文件，默认是从web服务器的根路径开始。  
   Append=false:默认值是true,即将消息增加到指定文件中，false指将消息覆盖指定的文件内容。  
   MaxFileSize=100KB: 后缀可以是KB, MB 或者是 GB. 在日志文件到达该大小时，将会自动滚动，即将原来的内容移到mylog.log.1文件。  
   MaxBackupIndex=2:指定可以产生的滚动文件的最大数。  
   ```

5.  org.apache.log4j.WriterAppender（将日志信息以流格式发送到任意指定的地方） 

   ```properties
   log4j.appender.R=org.apache.log4j.DailyRollingFileAppender  
   log4j.appender.R.File=D:\eclipsespace\testSSH\WebContent\WEB-INF\testlog.log  
   ```

**3、配置日志信息的格式（布局）Layout **

Layout 负责格式化Appender的输出，其语法为：  

```properties
log4j.appender.appenderName.layout = fully.qualified.name.of.layout.class  
```

其中"fully.qualified.name.of.layout.class" 有以下几种：  

 1.  org.apache.log4j.HTMLLayout（以HTML表格形式布局） ,  该选项有以下几种：  

     ```properties
     LocationInfo=true:默认值是false,输出java文件名称和行号  
     Title=my app file: 默认值是 Log4J Log Messages.  
     ```

2. org.apache.log4j.PatternLayout（可以灵活地指定布局模式）   该选项有以下几种：  

   ```properties
   ConversionPattern=%m%n :指定怎样格式化指定的消息  
   其中%m%n等符号所代表的含义如下：  
   －X号: X信息输出时左对齐；  
   %p: 输出日志信息优先级，即DEBUG，INFO，WARN，ERROR，FATAL,  
   %d: 输出日志时间点的日期或时间，默认格式为ISO8601，也可以在其后指定格式，比如：%d{yyy MMM dd HH:mm:ss,SSS}，输出类似：2002年10月18日 22：10：28，921  
   %r: 输出自应用启动到输出该log信息耗费的毫秒数  
   %c: 输出日志信息所属的类目，通常就是所在类的全名  
   %t: 输出产生该日志事件的线程名  
   %l: 输出日志事件的发生位置，相当于%C.%M(%F:%L)的组合,包括类目名、发生的线程，以及在代码中的行数。举例：Testlog4.main(TestLog4.java:10)  
   %x: 输出和当前线程相关联的NDC(嵌套诊断环境),尤其用到像java servlets这样的多客户多线程的应用中。  
   %%: 输出一个"%"字符  
   %F: 输出日志消息产生时所在的文件名称  
   %L: 输出代码中的行号  
   %m: 输出代码中指定的消息,产生的日志具体信息  
   %n: 输出一个回车换行符，Windows平台为"\r\n"，Unix平台为"\n"输出日志信息换行  
   可以在%与模式字符之间加上修饰符来控制其最小宽度、最大宽度、和文本的对齐方式。如：
   %20c：指定输出category的名称，最小的宽度是20，如果category的名称小于20的话，默认的情况下右对齐。  
   %-20c:指定输出category的名称，最小的宽度是20，如果category的名称小于20的话，"-"号指定左对齐。 
   %.30c:指定输出category的名称，最大的宽度是30，如果category的名称大于30的话，就会将左边多出的字符截掉，但小于30的话也不会有空格。  
   %20.30c:如果category的名称小于20就补空格，并且右对齐，如果其名称长于30字符，就从左边交远销出的字符截掉。  
   如：%-4r %-5p %d{yyyy-MM-dd HH:mm:ssS} %c %m%n  
   [TEST] %p [%t] %C.%M(%L) | %m%n  
   iii.org.apache.log4j.SimpleLayout（包含日志信息的级别和信息字符串）  
   iv.org.apache.log4j.TTCCLayout（包含日志产生的时间、线程、类别等等信息）  
   ```

**4.其他 **

`log4j.logger.com. neusoft =DEBUG`  
指定com.neusoft包下的所有类的等级为DEBUG。 

`log4j.logger.com.opensymphony.oscache=ERROR`  
`log4j.logger.net.sf.navigator=ERROR`  

这两句是把这两个包下出现的错误的等级设为ERROR，如果项目中没有配置EHCache，则不需要这两句。    

`log4j.logger.org.apache.commons=ERROR`  
`log4j.logger.org.apache.struts=WARN`  

这两句是struts的包。  

`log4j.logger.org.displaytag=ERROR`  

这句是displaytag的包。（QC问题列表页面所用）    

`log4j.logger.org.springframework=DEBUG`  
此句为Spring的包。  

`log4j.logger.org.hibernate.ps.PreparedStatementCache=WARN`  
`log4j.logger.org.hibernate=DEBUG` 

此两句是hibernate的包。

**5、将log写入多个文件中**

log4j配置:

```properties
log4j.rootCategory=INFO,stdout  
log4j.appender.stdout=org.apache.log4j.ConsoleAppender  
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout  
log4j.appender.stdout.layout.ConversionPattern=%d{yyyy-MM-dd HH\:mm\:ss,SSS}[ColorClouds] %p [%t] %C.%M(%L) | %m%n  
log4j.logger.net.sf.navigator=INFO  
log4j.logger.org.springframework=INFO  
log4j.logger.runLogger= INFO,R  
log4j.appender.R=org.apache.log4j.RollingFileAppender  
log4j.appender.R.File=G:\\log\\runLog.log  
log4j.appender.R.MaxFileSize=51200KB  
#log4j.appender.R.File=${com.huawei.icity.webapp.root}/log/icity.log  
log4j.appender.R.layout=org.apache.log4j.PatternLayout  
log4j.appender.R.layout.ConversionPattern=%d{yyyy-MM-dd HH\:mm\:ss,SSS}[ColorClouds Run] %p [%t] %C.%M(%L) | %m%n  
#log4j.logger.businessLogger= INFO,B  
#log4j.appender.B=org.apache.log4j.RollingFileAppender  
#log4j.appender.B.File=G:\\log\\businessLog.log  
#log4j.appender.B.MaxFileSize=51200KB  
#log4j.appender.B.layout=org.apache.log4j.PatternLayout  
#log4j.appender.B.layout.ConversionPattern=%d{yyyy-MM-dd HH\:mm\:ss,SSS}[ColorClouds Business] %p [%t] %C.%M(%L) | %m%n  
log4j.logger.interfaceLogger= INFO,I  
log4j.appender.I=org.apache.log4j.RollingFileAppender  
log4j.appender.I.File=G:\\log\\interfaceLog.log  
log4j.appender.I.MaxFileSize=51200KB  
log4j.appender.I.layout=org.apache.log4j.PatternLayout  
log4j.appender.I.layout.ConversionPattern=%d{yyyy-MM-dd HH\:mm\:ss,SSS}[ColorClouds Interface] %p [%t] %C.%M(%L) | %m%n 
```

java 调用：

```java
import org.apache.commons.logging.Log; 
import org.apache.commons.logging.LogFactory; 
public class QueryMyMeetingAction extends BaseFlowAction 
{ 
 /** 
 * uid 
 */
 private static final long serialVersionUID = 7612831197603586815L; 
 private static Log runLog = LogFactory.getLog("runLogger");//运行日志 
 private static Log interfaceLog = LogFactory.getLog("interfaceLogger");//接口日志 
 public String execute() throws Exception 
 { 
   interfaceLog.info("====>>请求"); 
   runLog.info("请求02" ); 
   return super.execute(); 
 } 
}
```

## Web项目中配置Log4j的方法

1. 动态的改变记录级别和策略，不需要重启Web应用，如《Effective Enterprise Java》所说。
2. 把log文件定在 /WEB-INF/logs/ 而不需要写绝对路径。
3. 可以把log4j.properties和其他properties一起放在/WEB-INF/ ，而不是Class-Path。

   在web.xml 添加

​    <!--如果不定义webAppRootKey参数，那么webAppRootKey就是缺省的"webapp.root"-->  

```xml
<context-param>
  <param-name>webAppRootKey</param-name>
  <param-value>xxx.root</param-value>
</context-param>
<context-param>
  <param-name>log4jConfigLocation</param-name>
  <param-value>WEB-INF/log4j.properties</param-value>
</context-param>
<context-param>
  <param-name>log4jRefreshInterval</param-name>
  <param-value>60000</param-value>
</context-param>
<listener>
  <listener-class>org.springframework.web.util.Log4jConfigListener</listener-class>
</listener>
```

 在上文的配置里,Log4jConfigListener会去WEB-INF/log4j.propeties 读取配置文件;

 开一条watchdog线程每60秒扫描一下配置文件的变化;

 并把web目录的路径压入一个叫webapp.root的系统变量。

 然后，在log4j.properties 里就可以这样定义logfile位置:

```properties
 log4j.appender.logfile.File=${webapp.root}/WEB-INF/logs/myfuse.log
```

如果有多个web应用，怕webapp.root变量重复，可以在context-param里定义webAppRootKey。



##  Log4j：单独指定某个Logger的输出级别

一般对于生产系统，日志级别调整为INFO以避免过多的输出日志。
但某些时候，需要跟踪具体问题，那么就得打开DEBUG日志。
但是如果打开log4j.rootLogger，则需要的信息就会淹没在日志的海洋中。
此时，需要单独指定某个或者某些Logger的日志级别为DEBUG，而rootLogger保持INFO不变。
参考配置如下（指定com.bs2.test.MyTest类的日志输出）

```properties
log4j.logger.com.bs2.test.MyTest=DEBUG
```

先看一个配置文件的例子:

```properties
1.配置文件的例子

log4j.rootLogger=DEBUG
#将DAO层log记录到DAOLog,allLog中
log4j.logger.DAO=DEBUG,A2,A4
#将逻辑层log记录到BusinessLog,allLog中
log4j.logger.Businesslog=DEBUG,A3,A4
#A1--打印到屏幕上
log4j.appender.A1=org.apache.log4j.ConsoleAppender
log4j.appender.A1.layout=org.apache.log4j.PatternLayout
log4j.appender.A1.layout.ConversionPattern=%-5p [%t] %37c %3x - %m%n
#A2--打印到文件DAOLog中--专门为DAO层服务
log4j.appender.A2=org.apache.log4j.DailyRollingFileAppender
log4j.appender.A2.file=DAOLog
log4j.appender.A2.DatePattern='.'yyyy-MM-dd
log4j.appender.A2.layout=org.apache.log4j.PatternLayout
log4j.appender.A2.layout.ConversionPattern=[%-5p] %d{yyyy-MM-dd HH:mm:ss,SSS} method:%l%n%m%n

#A3--打印到文件BusinessLog中--专门记录逻辑处理层服务log信息
log4j.appender.A3=org.apache.log4j.DailyRollingFileAppender
log4j.appender.A3.file=BusinessLog
log4j.appender.A3.DatePattern='.'yyyy-MM-dd
log4j.appender.A3.layout=org.apache.log4j.PatternLayout
log4j.appender.A3.layout.ConversionPattern=[%-5p] %d{yyyy-MM-dd HH:mm:ss,SSS} method:%l%n%m%n

#A4--打印到文件alllog中--记录所有log信息
log4j.appender.A4=org.apache.log4j.DailyRollingFileAppender
log4j.appender.A4.file=alllog
log4j.appender.A4.DatePattern='.'yyyy-MM-dd
log4j.appender.A4.layout=org.apache.log4j.PatternLayout
log4j.appender.A4.layout.ConversionPattern=[%-5p] %d{yyyy-MM-dd HH:mm:ss,SSS} method:%l%n%m%n
```



**在需要使用log4j的地方获取Logger实例**
如下是RoleDAO类中的使用例子:

```java
static Logger log = Logger.getLogger("DAO");
```

注意这里使用"DAO"标识符，那么对应的在配置文件中对应的配置信息如下：
定义DAO Logger

```properties
log4j.logger.DAO=DEBUG,A2
```

设置Appender A2的属性

```java
log4j.appender.A2=org.apache.log4j.DailyRollingFileAppender
log4j.appender.A2.file=demo
log4j.appender.A2.DatePattern='.'yyyy-MM-dd
log4j.appender.A2.layout=org.apache.log4j.PatternLayout
log4j.appender.A2.layout.ConversionPattern=%-5p %d{yyyy-MM-dd HH:mm:ss} %l%n%m%n

public class RoleDAO extends BaseDBObject
{
  ...
  static Logger log = Logger.getLogger("DAO");
  ...
  public BeanCollection selectAll() throws SQLException
  {
    StringBuffer sql = new StringBuffer(SQLBUF_LEN);
    sql.append("SELECT * FROM " + tableName + " order by roldId");
    //System.out.println(sql.toString());
    log.debug(sql);
      ...
  }
  ...
}
```

说明： 就是在log4j.properties 文件中配置了指定的 logger时，就在需要使用的该logger的地方通过：

```java
static Logger log = Logger.getLogger("DAO");
```

来获取这个logger；



ops-server 中的log4j的配置文件：

```properties
sog4j.rootLogger = info,stdout,R2

#log4j.rootLogger = on
log4j.rootLogger = debug, stdout, console, FILE
#log4j.rootLogger = info,stdout,R2,console

log4j.logger.mysqlLog=info,mysql,debug


#Console Log
log4j.logger.console=debug,stdout
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

