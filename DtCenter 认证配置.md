DtCenter 认证配置



```
# 用下面这个命令去生成认证文件
keytool -genkey -keyalg RSA -dname CN=*.dthink.dtdream.com -alias ssodthink -keysize 1024 -validity 730 -keypass changeit -keystore ssodthink.keystore
```



```
#导入认证文件；
keytool -export -alias ssodthink -keystore ssodthink.keystore -file ssodthink.crt
```



```
# 删除已经安装的但是无用的认证文件
keytool -delete -keystore "%JAVA_HOME%\jre\lib\security\cacerts" -alias ssodthink
```





