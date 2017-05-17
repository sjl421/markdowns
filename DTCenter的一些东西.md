# DTCenter的一些东西	

DTcenter 是一套独立的系统，他有自己的管理系统；



## 配置信息

DTCenter的配置信息在安装路径下的：

```json
/etc/DTCenter/DTCenter-EMR-v2.1.1/dtcloud.cfg
```

其中的: uim_server_host 表示的是dtcenter的用户用户管理

其中的: platform_server_host表示的是dtcenter的平台管理界面，里面包括：

* 用户管理
* 角色管理
* 部门管理
* 登录策略管理




---

# DtCenter的安装配置问题

@2017.5.15

老版本的DtCenter中需要在你对接的tomcat中/conf/server.xml中配置DtCenter keystore文件后tomcat中的webapp才能正常的对接DtCenter，新版本的DtCenter不需要在server.xml中配置，只需要在你tomcat 环境中的jdk中导入哪个keytore就可以了;

以下是针对windows 环境：

```shell
keytool -import -keystore "%JAVA_HOME%\jre\lib\security\cacerts" -file $path\ssodthink.crt -alias ssodthink
```

```shell
# 用下面这个命令去生成认证文件
keytool -genkey -keyalg RSA -dname CN=*.dthink.dtdream.com -alias ssodthink -keysize 1024 -validity 730 -keypass changeit -keystore ssodthink.keystore
```

```shell
导入证书
keytool -import -keystore "%JAVA_HOME%\jre\lib\security\cacerts" -file $path\ssodthink.crt -alias ssodthink
```


```shell
#导出认证文件；
keytool -export -alias ssodthink -keystore ssodthink.keystore -file ssodthink.crt
```

```shell
# 删除已经安装的但是无用的认证文件
keytool -delete -keystore "%JAVA_HOME%\jre\lib\security\cacerts" -alias ssodthink
```
```powershell
keytool -list -keystore "%JAVA_HOME%\jre\lib\security\cacerts" | findstr sso
```



**历史巨锅**

在linux环境下，环境变量使用的是$JAVA_HOME, 不知 %JAVA_HOME%，并且linux下路径中文件夹的分隔符是/ ， 而windows下的是\,  别一股脑的乱粘贴复制;