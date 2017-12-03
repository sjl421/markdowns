# CentOS从本地yum源安装软件

问题描述： 

需要从本地配置好的yum源安装docker，

1、首先需要更新本地的repo配置文件,配置文件的地址为：`/etc/yum.respos.d`

2、然后就可以按照正常的yum install package的安装软件了；

```l
# yum clean all
# rpm --rebuilddb
# yum update
```



# Q&A：

我第一次在安装的时候提示有的软件依赖的版本不满足，然后就想着要更新一下软件，调用的命令是：`yum update` ,但是报了很多的404的错误的，最后在网上找到的解决方法是说yum镜像数据库坏了，然后用下面的命令修复：

```
# yum clean all
# rpm --rebuilddb
# yum update
```

> 这些命令是在客户机上执行的，不是在yum源服务器上执行的；

