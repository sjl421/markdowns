### 编译C源码文件

1. 先用configure命令配置一下

```
./configure --prefix=/usr/local/gpdb --with-includes=/root/libevent_1.4.6/include --with-libs=/root/libevent_1.4.6/lib --enable-snmp --with-tcl --with-perl --with-python --with-krb5 --with-ldap --with-openssl --with-libxml --with-libxslt --enable-debug
```

2. ​

```
make & make install
```

其中make install的位置是将最终编译输出的结果输出到`--prefix` 参数指定的位置;