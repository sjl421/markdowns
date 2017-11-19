#keepalived基本应用解析

概念简单认知：

Keepalived:它的诞生最初是为ipvs（一些服务，内核中的一些规则）提供高可用性的，最初最主要目的是能够自主调用ipvsadm来生成规则，并且能够自动实现将用户访问的地址转移到其他节点上进行实现的。

Keepalived:核心包含两个ckeckers和VRRP协议。

ckeckers：检查服务检查reserved的健康状况的，基于脚本也可检查服务本身的健康状况。这里是实现ipvs后端健康状况的检测的。

VRRP：是一种容错协议，它保证当主机的下一跳路由器出现故障时，由另一台路由器来代替出现故障的路由器进行工作，从而保持网络通信的连续性和可靠性。VRRP中每个节点之间都有优先级的一般为0-255（0,255有特殊用法）数字越大优先级越高。

相关术语解析：

虚拟路由器：由一个Master路由器和多个Backup路由器组成。主机将虚拟路由器当作默认网关。

VRID：虚拟路由器的标识。有相同VRID的一组路由器构成一个虚拟路由器。

Master路由器：虚拟路由器中承担报文转发任务的路由器。

Backup路由器：Master路由器出现故障时，能够代替Master路由器工作的路由器。

虚拟IP 地址：虚拟路由器的IP 地址。一个虚拟路由器可以拥有一个或多个IP地址。

IP地址拥有者：接口IP地址与虚拟IP地址相同的路由器被称为IP地址拥有者。

虚拟MAC地址：一个虚拟路由器拥有一个虚拟MAC地址。虚拟MAC地址的格式为00-00-5E-00-01-{VRID}。通常情况下，虚拟路由器回应ARP请求使用的是虚拟MAC地址，只有虚拟路由器做特殊配置的时候，才回应接口的真实MAC地址。

优先级：VRRP根据优先级来确定虚拟路由器中每台路由器的地位。

非抢占方式：如果Backup路由器工作在非抢占方式下，则只要Master路由器没有出现故障Backup路由器即使随后被配置了更高的优先级也不会成为Master路由器。

抢占方式：如果Backup路由器工作在抢占方式下，当它收到VRRP报文后，会将自己的优先级与通告报文中的优先级进行比较。如果自己的优先级比当前的Master路由器的优先级高，就会主动抢占成为Master路由器；否则，将保持Backup状态。

------

平台信息介绍：

 Master：172.16.18.7

 backup：172.16.18.9

 系统版本：centosx86_64

 keepalived版本：1.2.7

 主配置文件：/etc/keepalived/keepalived.conf

 服务脚本：/etc/rc.d/init,d/keepalived

------

应用实践：

将两个节点的时间同步

```
##############实现双机互信#######
#########node1#######
ssh-keygen -t rsa -P ''
ssh-copy-id -i .ssh/id_rsa.pub root@172.16.18.9
#########node2#######
ssh-keygen -t rsa -P ''
ssh-copy-id -i .ssh/id_rsa.pub root@172.16.18.7
##############查看时间###########
[root@node1 ~]# date;ssh node2 'date'
#####为实现同步可使用下面同步#####
crontab -e
*/5 * * * * /usr/sbin/ntpdate 172.16.0.1 &> /dev/null
```

安装：

```
[root@node1 ~]# yum -y install keepalived
```

查看编辑配置文件：

```
vim /etc/keepalived/keepalived.conf
```

解析配置文件：

配置文件有三部分组成：

```
（1）：GLOBAL CONFIGURATION        全局配置段
    有两个字段：
    Global definitions              #全局定义
    Static routes                   #静态路由
（2）：VRRPD CONFIGURATION         配置VRRP子进程协议段又称定义虚拟路由的
    有两个字段：
    VRRP synchronization group(s)   #VRRP的同步组（一般不用）
    什么是同步组？就是一台机器上有配置两个VIP，为了实现两个VIP要同步工作同时转移出去，所以必须要定义成同步组从而当成一个资源来转移。
    VRRP instance(s)                #VRRP的实例
    任何一个虚拟路由定义好之后在任何一个节点上都应该定义一个keepalived运行实例，这两个节点上的实例要匹配。keepalived最令人头疼的是两个节点上的初始实例是不一样的，因为每一个节点都有初始状态而且它有默认的优先级，高的为Master低的为Backup所以导致了两个节点上的虚拟路由的实例配置是不一样的。
（3）：LVS CONFIGURATION            LVS配置段
    有两个字段：
    Virtual server group(s)         #虚拟服务器组
    Virtual server(s)               #虚拟服务器（ipvs规则）
```

详细解析：

keepalived.conf

```
global_defs {    #全局配置，这里额外的静态路由并未添加因为它是非必要的，除非我们在当前或特定的主机上生成特殊的静态路由等
   notification_email {                #收件人信息
     acassen@firewall.loc
     failover@firewall.loc
     sysadmin@firewall.loc
   }
   notification_email_from Alexandre.Cassen@firewall.loc   #发件人信息（可以随意伪装）
   smtp_server 192.168.200.1      #发邮件的服务器（一定不可为外部地址）
   smtp_connect_timeout 30      #连接超时时间
   router_id LVS_DEVEL          #路由器的标识（可以随便改动）
}
vrrp_instance VI_1 {            #配置虚拟路由器的（VI_1是实例名称）
    state MASTER               #初始状态，master|backup，当state指定的instance的初始化状态，在两台服务器都启动以后，马上发生竞选，优先级高的成为MASTER，所以这里的MASTER并不是表示此台服务器一直是MASTER
    interface eth0              #通告选举所用端口
    virtual_router_id 51        #虚拟路由的ID号（一般不可大于255）
    priority 100                #优先级信息
    advert_int 1                #初始化通告几个
    authentication {            #认证
        auth_type PASS          #认证机制
        auth_pass 1111          #密码（尽量使用随机）
    }
    virtual_ipaddress {         #虚拟地址（VIP地址）
        192.168.200.16
        192.168.200.17
        192.168.200.18
    }
}
```

编辑设置配置信息：

```
[root@node1 ~]# vim /etc/keepalived/keepalived.conf     #主节点
global_defs {
   notification_email {
   root@localhost
   }
   notification_email_from keadmin@localhost
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 55
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        172.16.18.100
    }
}
[root@node1 ~]# scp /etc/keepalived/keepalived.conf 172.16.18.9:/etc/keepalived/             #复制至备节点
[root@node2 keepalived]# vim keepalived.conf     #备节点
global_defs {
   notification_email {
   root@localhost
   }
   notification_email_from keadmin@localhost
   smtp_server 127.0.0.1
   smtp_connect_timeout 30
   router_id LVS_DEVEL
}

vrrp_instance VI_1 {
    state BACKUP                         #状态
    interface eth0
    virtual_router_id 55                 #一定要和主节点一致
    priority 90                          #优先级别一定低于主节点
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        172.16.18.100
    }
}
```

启动主节点：

```
[root@node1 ~]# service keepalived start
```

启动备节点：

```
[root@node2 ~]# service keepalived start
```

查看状态：

```
###############节点一############
[root@node1 ~]# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:06:a6:49 brd ff:ff:ff:ff:ff:ff
    inet 172.16.18.7/16 brd 172.16.255.255 scope global eth0
    inet 172.16.18.100/32 scope global eth0    #此时VIP在node1节点上
    inet6 fe80::20c:29ff:fe06:a649/64 scope link
       valid_lft forever preferred_lft forever
###############节点二############
[root@node2 keepalived]# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:12:c8:b5 brd ff:ff:ff:ff:ff:ff
    inet 172.16.18.9/16 brd 172.16.255.255 scope global eth0
    inet6 fe80::20c:29ff:fe12:c8b5/64 scope link
       valid_lft forever preferred_lft forever
```

查看主节点启动日志信息：

```
##############主节点#############
[root@node1 ~]# tail -20 /var/log/messages
Sep 25 17:32:17 node1 Keepalived_healthcheckers[16628]: Netlink reflector reports IP 172.16.18.7 added
Sep 25 17:32:17 node1 Keepalived_healthcheckers[16628]: Netlink reflector reports IP fe80::20c:29ff:fe06:a649 added
Sep 25 17:32:17 node1 Keepalived_healthcheckers[16628]: Registering Kernel netlink reflector
Sep 25 17:32:17 node1 Keepalived_healthcheckers[16628]: Registering Kernel netlink command channel
Sep 25 17:32:17 node1 Keepalived_healthcheckers[16628]: Opening file '/etc/keepalived/keepalived.conf'.
Sep 25 17:32:17 node1 Keepalived_healthcheckers[16628]: Configuration is using : 6832 Bytes
Sep 25 17:32:18 node1 kernel: IPVS: Registered protocols (TCP, UDP, SCTP, AH, ESP)
Sep 25 17:32:18 node1 kernel: IPVS: Connection hash table configured (size=4096, memory=64Kbytes)
Sep 25 17:32:18 node1 kernel: IPVS: ipvs loaded.
Sep 25 17:32:18 node1 Keepalived_healthcheckers[16628]: Using LinkWatch kernel netlink reflector...
Sep 25 17:32:18 node1 Keepalived_vrrp[16629]: Opening file '/etc/keepalived/keepalived.conf'.
Sep 25 17:32:18 node1 Keepalived_vrrp[16629]: Configuration is using : 62657 Bytes
Sep 25 17:32:18 node1 Keepalived_vrrp[16629]: Using LinkWatch kernel netlink reflector...
Sep 25 17:32:18 node1 Keepalived_vrrp[16629]: VRRP sockpool: [ifindex(2), proto(112), fd(11,12)]
Sep 25 17:32:19 node1 Keepalived_vrrp[16629]: VRRP_Instance(VI_1) Transition to MASTER STATE       #事务开始转换为Master状态
Sep 25 17:32:20 node1 Keepalived_vrrp[16629]: VRRP_Instance(VI_1) Entering MASTER STATE          #进入master状态
Sep 25 17:32:20 node1 Keepalived_vrrp[16629]: VRRP_Instance(VI_1) setting protocol VIPs.
Sep 25 17:32:20 node1 Keepalived_vrrp[16629]: VRRP_Instance(VI_1) Sending gratuitous ARPs on eth0 for 172.16.18.100
Sep 25 17:32:20 node1 Keepalived_healthcheckers[16628]: Netlink reflector reports IP 172.16.18.100 added             #添加IP172.16.18.100
Sep 25 17:32:25 node1 Keepalived_vrrp[16629]: VRRP_Instance(VI_1) Sending gratuitous ARPs on eth0 for 172.16.18.100
###############备节点##########
[root@node2 keepalived]# tail -20 /var/log/messages
Sep 25 17:32:23 node2 Keepalived_healthcheckers[20357]: Interface queue is empty
Sep 25 17:32:23 node2 Keepalived_vrrp[20358]: Interface queue is empty
Sep 25 17:32:23 node2 Keepalived_healthcheckers[20357]: Netlink reflector reports IP 172.16.18.9 added
Sep 25 17:32:23 node2 Keepalived_vrrp[20358]: Netlink reflector reports IP 172.16.18.9 added
Sep 25 17:32:23 node2 Keepalived_healthcheckers[20357]: Netlink reflector reports IP fe80::20c:29ff:fe12:c8b5 added
Sep 25 17:32:23 node2 Keepalived_vrrp[20358]: Netlink reflector reports IP fe80::20c:29ff:fe12:c8b5 added
Sep 25 17:32:23 node2 Keepalived_vrrp[20358]: Registering Kernel netlink reflector
Sep 25 17:32:23 node2 Keepalived_healthcheckers[20357]: Registering Kernel netlink reflector
Sep 25 17:32:23 node2 Keepalived_healthcheckers[20357]: Registering Kernel netlink command channel
Sep 25 17:32:23 node2 Keepalived_vrrp[20358]: Registering Kernel netlink command channel
Sep 25 17:32:23 node2 Keepalived_vrrp[20358]: Registering gratuitous ARP shared channel
Sep 25 17:32:23 node2 Keepalived_healthcheckers[20357]: Opening file '/etc/keepalived/keepalived.conf'.
Sep 25 17:32:23 node2 Keepalived_healthcheckers[20357]: Configuration is using : 6985 Bytes
Sep 25 17:32:23 node2 Keepalived_vrrp[20358]: Opening file '/etc/keepalived/keepalived.conf'.
Sep 25 17:32:23 node2 Keepalived_vrrp[20358]: Configuration is using : 62678 Bytes
Sep 25 17:32:23 node2 Keepalived_vrrp[20358]: Using LinkWatch kernel netlink reflector...
Sep 25 17:32:23 node2 Keepalived_vrrp[20358]: VRRP_Instance(VI_1) Entering BACKUP STATE              #进入BACKUP状态
Sep 25 17:32:23 node2 Keepalived_vrrp[20358]: VRRP sockpool: [ifindex(2), proto(112), fd(10,11)]
Sep 25 17:32:23 node2 Keepalived_healthcheckers[20357]: Using LinkWatch kernel netlink reflector...
```

测试：将node1关闭，node2会不会将地址取走？？

```
##############主节点############
[root@node1 ~]# service keepalived stop
Stopping keepalived:                                       [  OK  ]
[root@node1 ~]# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:06:a6:49 brd ff:ff:ff:ff:ff:ff
    inet 172.16.18.7/16 brd 172.16.255.255 scope global eth0
    inet6 fe80::20c:29ff:fe06:a649/64 scope link
       valid_lft forever preferred_lft forever
###############备节点##########
[root@node2 keepalived]# ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:12:c8:b5 brd ff:ff:ff:ff:ff:ff
    inet 172.16.18.9/16 brd 172.16.255.255 scope global eth0
    inet 172.16.18.100/32 scope global eth0
    inet6 fe80::20c:29ff:fe12:c8b5/64 scope link
       valid_lft forever preferred_lft forever
```

测试结果：这样是成立的，若node1重新上线会立即将VIP获取走。

如何使用keepalived调用外部脚本或手动执行命令实现VIP转移？？

思路：通过addr_script（脚本）定义检测机制；然后通过track_script在实例中追踪这个脚本。

```
vrrp_script chk_mantaince_down {             # chk_mantaince_down定义脚本的名称，可随意取
   script "[[ -f /etc/keepalived/down ]] && exit 1 || exit 0"   #命令（其实这里可以是自己定义好的脚本路径也可以是判断命令）#这里的意思是如果在这个文件下有down这个文件就表示期望这 个节点为备用状态。
   interval 1           #每隔1秒钟执行一次
   weight -2            #一旦命令执行失败，权重降低2个
}
```

如：下面这个检测机制

（1）将此此检查机制应用到我们的示例中测试实现过程：

```
#############主节点###########
[root@node1 keepalived]# vim keepalived.conf
 router_id LVS_DEVEL
}                                                                                                                                                      
vrrp_script chk_main {      脚本
   script "[[ -f /etc/keepalived/down ]] && exit 1 || exit 0"
   interval 1
   weight -2
}                                                                                                                                                 
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 157       #如果环境中操作者比较多，尽量在每次更改配置文件之后改变一下这个值，从而实现ARPs快速转接。
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        172.16.18.100
    }
    track_script {        #追踪脚本
        chk_main
    }
}
##############备节点############
[root@node2 keepalived]# vim keepalived.conf
smtp_connect_timeout 30
   router_id LVS_DEVEL
}
                                                     
vrrp_script chk_main {
   script "[[ -f /etc/keepalived/down ]] && exit 1 || exit 0"
   interval 1
   weight -2
}                                                                                                                                    
vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 157
    priority 99
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        172.16.18.100
    }
    track_script {
        chk_main
    }
}
```

（2）测试

```
#######未添加文件之前：节点一########
[root@node1 keepalived]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:06:a6:49 brd ff:ff:ff:ff:ff:ff
    inet 172.16.18.7/16 brd 172.16.255.255 scope global eth0
    inet 172.16.18.100/32 scope global eth0
    inet6 fe80::20c:29ff:fe06:a649/64 scope link
       valid_lft forever preferred_lft forever
#######未添加文件之前：节点二########
[root@node2 keepalived]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:12:c8:b5 brd ff:ff:ff:ff:ff:ff
    inet 172.16.18.9/16 brd 172.16.255.255 scope global eth0
    inet6 fe80::20c:29ff:fe12:c8b5/64 scope link
       valid_lft forever preferred_lft forever
############添加文件########
[root@node1 keepalived]# touch down
###########查看状态#########
[root@node1 keepalived]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:06:a6:49 brd ff:ff:ff:ff:ff:ff
    inet 172.16.18.7/16 brd 172.16.255.255 scope global eth0
    inet6 fe80::20c:29ff:fe06:a649/64 scope link
       valid_lft forever preferred_lft forever
[root@node2 keepalived]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:12:c8:b5 brd ff:ff:ff:ff:ff:ff
    inet 172.16.18.9/16 brd 172.16.255.255 scope global eth0
    inet 172.16.18.100/32 scope global eth0
    inet6 fe80::20c:29ff:fe12:c8b5/64 scope link
       valid_lft forever preferred_lft forever
```

如何在状态转换时进行通知？？

（1）keepalive内部提供了两个配置指令详细参考man keepalived.conf，一般在vrrp_instance或者vrrp_sync_group中使用中使用：

第一类指令：

```
# to MASTER transition
notify_master /path/to_master.sh     #转换为master状态时使用此脚本通知
# to BACKUP transition
notify_backup /path/to_backup.sh     #转换为backup状态时使用此脚本通知
# FAULT transition
notify_fault "/path/fault.sh VG_1"   #如果变成了fault就是用此脚本通知，如果脚本带有参数也就是有空格必须使用引号
```

第二类指令：使用notify直接引用

```
# $1 = "GROUP"|"INSTANCE"            #参数1：必须能够指定接受组或实例
# $2 = name of group or instance     #这个组或实例的名称
# $3 = target state of transition    #指定转换成哪个状态进行通知的
#     ("MASTER"|"BACKUP"|"FAULT")
notify /path/notify.sh     notify    #脚本的路径（自行写）
```

（2）通知脚本定义：

```
[root@node1 keepalived]# vim notify.sh
#!/bin/bash
#
vip=172.16.18.100           #指定VIP
contact='root@localhost'    #通知给谁
thisip=`ifconfig eth0 | awk '/inet addr:/{print $2}' | awk -F: '{print $2}'`                       #获取当前节点IP地址
notify() {
    mailsubject="$thisip to be $1: $vip floating"
    mailbody="`date '+%F %H:%M:%S'`: vrrp transition, $thisip changed to be $1"
    echo $mailbody | mail -s "$mailsubject" $contact
}
case "$1" in
    master)
        notify master
        exit 0
    ;;
    backup)
        notify backup
        exit 0
    ;;
    fault)
        notify fault
        exit 0
    ;;
    *)
        echo 'Usage: `basename $0` {master|backup|fault}'
        exit 1
    ;;
esac
##########赋予此脚本执行权限###########
[root@node1 keepalived]# chmod +x notify.sh
###########测试脚本####################
[root@node1 keepalived]# ./notify.sh master
###########查看邮件信息################
[root@node1 keepalived]# mail
Heirloom Mail version 12.4 7/29/08.  Type ? for help.
"/var/spool/mail/root": 1 message 1 new
>N  1 root                  Wed Sep 25 22:24  18/693   "172.16.18.7 to be master: 172.16.18.100 flo"
& 1        #第一封邮件
Message  1:
From root@node1.magedu.com  Wed Sep 25 22:24:40 2013
Return-Path: <root@node1.magedu.com>
X-Original-To: root@localhost
Delivered-To: root@localhost.magedu.com
Date: Wed, 25 Sep 2013 22:24:39 +0800
To: root@localhost.magedu.com
Subject: 172.16.18.7 to be master: 172.16.18.100 floating
User-Agent: Heirloom mailx 12.4 7/29/08
Content-Type: text/plain; charset=us-ascii
From: root@node1.magedu.com (root)
Status: R
2013-09-25 22:24:39: vrrp transition, 172.16.18.7 changed to be master #内容
###########使用quit退出邮件##############
```

```
#############配置测试状态转换##############
[root@node1 keepalived]# vim keepalived.conf
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 157
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        172.16.18.100
    }
    track_script {
        chk_main
    }
    notify_master "/etc/keepalived/notify.sh master" #指定切换到Master状态时执行的脚本
    notify_backup "/etc/keepalived/notify.sh backup" #指定切换到Backup状态时执行的脚本
    notify_fault "/etc/keepalived/notify.sh fault" #指定切换到Mfault状态时执行的脚本
}
##########注意将上面此代码写入备节点中############
notify_master "/etc/keepalived/notify.sh master" #指定切换到Master状态时执行的脚本
notify_backup "/etc/keepalived/notify.sh backup" #指定切换到Backup状态时执行的脚本
notify_fault "/etc/keepalived/notify.sh fault"   #指定切换到Mfault状态时执行的脚本
##########脚本同样在备节点中存在##################
[root@node1 keepalived]# scp notify.sh 172.16.18.9:/etc/keepalived/
```

（3）测试

```
[root@node1 keepalived]# touch down
[root@node1 keepalived]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:06:a6:49 brd ff:ff:ff:ff:ff:ff
    inet 172.16.18.7/16 brd 172.16.255.255 scope global eth0
    inet6 fe80::20c:29ff:fe06:a649/64 scope link
       valid_lft forever preferred_lft forever
[root@node1 keepalived]# mail       #节点1上
Heirloom Mail version 12.4 7/29/08.  Type ? for help.
"/var/spool/mail/root": 3 messages 2 unread
    1 root                  Wed Sep 25 22:24  19/704   "172.16.18.7 to be master: 172.16.18.100 flo"
>U  2 root                  Wed Sep 25 22:47  19/703   "172.16.18.7 to be master: 172.16.18.100 flo"
 U  3 root                  Wed Sep 25 22:47  19/703   "172.16.18.7 to be backup: 172.16.18.100 flo"
&
[root@node2 keepalived]# mail      #节点2上
Heirloom Mail version 12.4 7/29/08.  Type ? for help.
"/var/spool/mail/root": 3 messages 3 new
>N  1 root                  Wed Sep 25 22:46  18/693   "172.16.18.9 to be backup: 172.16.18.100 flo"
 N  2 root                  Wed Sep 25 22:47  18/693   "172.16.18.9 to be backup: 172.16.18.100 flo"
 N  3 root                  Wed Sep 25 22:47  18/693   "172.16.18.9 to be master: 172.16.18.100 flo"
&
```

关于ipvs配置生成规则实现负载均衡和web服务器实现高可用等更多关于keepalived高级应用将在后续博客中持续更新，请继续关注！！谢谢！！

原文地址:http://pangge.blog.51cto.com/6013757/1301878