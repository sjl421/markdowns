# CentOS 网络配置

首先通过ip addr 命令查看当前主机的网卡名；

网卡的配置文件地址：

/etc/sysconfig/network-script/

然后该文件夹瞎下面有一个主机网卡的配置文件，文件名和网卡的名字一致；

手动配置静态ip的话需要：

```shell
BOOTPROTO=static		//将dhcp改为static
#如果没有下面的内容则添加
IPADDR=192.168.181.215
NETMAST=255.255.255.0
GATEWAY=192.168.181.254

#然后重启网络
systemcrtl restart network
或者
service netword restart
```

