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

```
CFLAGS="-O0"这个参数让编译不要做优化，方便查看程序执行，默认是做了优化的，做了优化的，gdb调试的时候代码会跳来跳去
```

```
CFLAGS="-O0" ./configure --prefix=/usr/local/gpdb --with-includes=/root/libevent_1.4.6/include --with-libs=/root/libevent_1.4.6/lib --enable-snmp --with-tcl --with-perl --with-python --with-krb5 --with-ldap --with-openssl --with-libxml --with-libxslt --enable-debug 
```

```
./configure --prefix=/usr/local/gpdb --with-includes=/root/libevent_1.4.6/include --with-libs=/root/libevent_1.4.6/lib --enable-snmp --with-tcl --with-perl --with-python --with-krb5 --with-ldap --with-openssl --with-libxml --with-libxslt --enable-debug CFLAGS="-O0"
```



temp

```
./configure --prefix=/opt/gpdb_v1.1.0sp1_lockdebug --with-includes=/root/libevent_1.4.6/include --with-libs=/root/libevent_1.4.6/lib --enable-snmp --with-tcl --with-perl --with-python --with-krb5 --with-ldap --with-openssl --with-libxml --with-libxslt --enable-debug CFLAGS="-O0"
```

