# keystore的生成：  #

分阶段生成：  

    keytool -genkey -alias apiservice(别名) -keypass apiservice(密钥口令) -keyalg RSA(算法)  -keysize 1024(密钥长度) -validity 365(有效期，天单位) -keystore /apiservice.keystore(指定生成证书的位置和证书名称) -storepass 123456(密钥库口令)；
```
keytool -genkey -alias apiservice -keypass apiservice -keyalg RSA  -keysize 1024 -validity 365 -keystore apiservice.keystore -storepass 123456 -ext san=ip:192.168.1.103
```



一次性生成：

    keytool -genkey -alias apiservice -keypass apiservice -keyalg RSA -keysize 1024 -validity 365 -keystore /apiservice.keystore -storepass 123456 -dname "CN=(名字与姓氏), OU=(组织单位名称), O=(组织名称), L=(城市或区域名称), ST=(州或省份名称), C=(单位的两字母国家代码)";(中英文即可)
```
keytool -genkey -alias apiservice -keypass apiservice -keyalg RSA -keysize 1024 -validity 365 -keystore apiservice.keystore -storepass 123456 -ext san=ip:183.131.17.254 -dname "CN=183.131.17.254, OU=DtDream, O=ADB, L=HangZhou, ST=ZheJiang, C=China" 
```

```
keytool -genkey -alias apiservice  -keyalg RSA -keysize 1024 -validity 365 -keystore apiservice.keystore -ext san=ip:183.131.17.254 -dname "CN=183.131.17.254, OU=DtDream, O=ADB, L=HangZhou, ST=ZheJiang, C=China" 
```

```
keytool -delete  -alias apiservice -keystore keystore-name -storepass apiservice
```

```
keytool -delete  -alias apiservice -storepass apiservice
```



# 证书的导出：  #

    keytool -export -alias yushan -keystore apiservice.keystore(证书源文件) -file /apiservice.cer(指定导出的证书位置及证书名称) -storepass 123456
## 最终是用的命令

1. 生成keystore

```
keytool -genkey -alias apiservice  -keyalg RSA -keysize 1024 -validity 365 -keystore apiservice.keystore -ext san=ip:183.131.17.254 -dname "CN=183.131.17.254, OU=DtDream, O=ADB, L=HangZhou, ST=ZheJiang, C=China" 
# 这个命令中没有设置密码的动作,如果需要设置密码,在生成keystore的过程中直接输入密码, 其中会有两个地方让生成密码,每个地方需要输入两次密码, 注意这两个地方的密码需要是相同的密码,否则tomcat在启动的时候会报错;
```

2. 导出证书

   ```
keytool -export -alias apiservice -keystore apiservice.keystore -file apiservice.cer -storepass DtDream@0209
   ```

3. 如果需要删除之前keystore:

```
keytool -delete  -alias apiservice -keystore keystore-name -storepass apiservice
```

```
keytool -delete  -alias apiservice -storepass apiservice
```

# 

## 下面是各选项的缺省值。  ##

-alias "mykey"

-keyalg "DSA"

-keysize 1024

-validity 90

-keystore 用户宿主目录中名为 .keystore 的文件

-file 读时为标准输入，写时为标准输出 


# tomcat配置 #

修改conf/server.xml


    <!-- Define a SSL HTTP/1.1 Connector on port 8443
         This connector uses the BIO implementation that requires the JSSE
         style configuration. When using the APR/native implementation, the
         OpenSSL style configuration is required as described in the APR/native
         documentation -->
    <!--
    <Connector port="8443" protocol="org.apache.coyote.http11.Http11Protocol"
               maxThreads="150" SSLEnabled="true" scheme="https" secure="true"
               clientAuth="false" sslProtocol="TLS" />
改为

    <Connector port="443" protocol="org.apache.coyote.http11.Http11NioProtocol"
               maxThreads="150" SSLEnabled="true" scheme="https" secure="true"
               clientAuth="false" sslProtocol="TLS" 
               keystoreFile="证书地址" keystorePass="密钥库口令"/>