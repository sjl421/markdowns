# keepalived原理

1.Keepalived 定义

​       Keepalived 是一个基于VRRP协议来实现的**LVS服务高可用**方案，可以利用其来避免单点故障。一个LVS服务会有2台服务器运行Keepalived，一台为主服务器（MASTER），一台为备份服务器（BACKUP），但是对外表现为一个虚拟IP，主服务器会发送特定的消息给备份服务器，当备份服务器收不到这个消息的时候，即主服务器宕机的时候， 备份服务器就会接管虚拟IP，继续提供服务，从而保证了高可用性。Keepalived是VRRP的完美实现，因此在介绍keepalived之前，先介绍一下VRRP的原理。

Keepalived使用的vrrp协议方式，虚拟路由冗余协议 (Virtual Router Redundancy Protocol，简称VRRP), Keepalived的目的是模拟路由器的高可用([文章](http://freeloda.blog.51cto.com/2033581/1280962) 中还提到了服务高可用的概念,用来和这里的路由器高可用做对比理解)

1. Keepalived 定义

​       Keepalived 是一个基于VRRP协议来实现的LVS服务高可用方案，可以利用其来避免单点故障。一个LVS服务会有2台服务器运行Keepalived，一台为主服务器（MASTER），一台为备份服务器（BACKUP），但是对外表现为一个虚拟IP，主服务器会发送特定的消息给备份服务器，当备份服务器收不到这个消息的时候，即主服务器宕机的时候， 备份服务器就会接管虚拟IP，继续提供服务，从而保证了高可用性。Keepalived是VRRP的完美实现，因此在介绍keepalived之前，先介绍一下VRRP的原理。

2. VRRP 协议简介

在现实的网络环境中，两台需要通信的主机大多数情况下并没有直接的物理连接。对于这样的情况，它们之间路由怎样选择？主机如何选定到达目的主机的下一跳路由，这个问题通常的解决方法有二种：

- 在主机上使用动态路由协议(RIP、OSPF等)
- 在主机上配置静态路由

很明显，在主机上配置动态路由是非常不切实际的，因为管理、维护成本以及是否支持等诸多问题。配置静态路由就变得十分流行，但路由器(或者说默认网关default gateway)却经常成为单点故障。VRRP的目的就是为了解决静态路由单点故障问题，VRRP通过一竞选(election)协议来动态的将路由任务交给LAN中虚拟路由器中的某台VRRP路由器。

3. VRRP 工作机制

​       在一个VRRP虚拟路由器中，有多台物理的VRRP路由器，但是这多台的物理的机器并不能同时工作，而是由一台称为MASTER的负责路由工作，其它的都是BACKUP，MASTER并非一成不变，VRRP让每个VRRP路由器参与竞选，最终获胜的就是MASTER。MASTER拥有一些特权，比如，拥有虚拟路由器的IP地址，我们的主机就是用这个IP地址作为静态路由的。拥有特权的MASTER要负责转发发送给网关地址的包和响应ARP请求。

​       VRRP通过竞选协议来实现虚拟路由器的功能，所有的协议报文都是通过IP多播(multicast)包(多播地址224.0.0.18)形式发送的。虚拟路由器由VRID(范围0-255)和一组IP地址组成，对外表现为一个周知的MAC地址。所以，在一个虚拟路由 器中，不管谁是MASTER，对外都是相同的MAC和IP(称之为VIP)。客户端主机并不需要因为MASTER的改变而修改自己的路由配置，对客户端来说，这种主从的切换是透明的。

​       在一个虚拟路由器中，只有作为MASTER的VRRP路由器会一直发送VRRP通告信息(VRRPAdvertisement message)，BACKUP不会抢占MASTER，除非它的优先级(priority)更高。当MASTER不可用时(BACKUP收不到通告信息)， 多台BACKUP中优先级最高的这台会被抢占为MASTER。这种抢占是非常快速的(<1s)，以保证服务的连续性。由于安全性考虑，VRRP包使用了加密协议进行加密。

4. VRRP 工作流程

(1).初始化：    

​	路由器启动时，如果路由器的优先级是255(最高优先级，路由器拥有路由器地址)，要发送VRRP通告信息，并发送广播ARP信息通告路由器IP地址对应的MAC地址为路由虚拟MAC，设置通告信息定时器准备定时发送VRRP通告信息，转为MASTER状态；否则进入BACKUP状态，设置定时器检查定时检查是否收到MASTER的通告信息。

(2).Master

- 设置定时通告定时器；
- 用VRRP虚拟MAC地址响应路由器IP地址的ARP请求；
- 转发目的MAC是VRRP虚拟MAC的数据包；
- 如果是虚拟路由器IP的拥有者，将接受目的地址是虚拟路由器IP的数据包，否则丢弃；
- 当收到shutdown的事件时删除定时通告定时器，发送优先权级为0的通告包，转初始化状态；
- 如果定时通告定时器超时时，发送VRRP通告信息；
- 收到VRRP通告信息时，如果优先权为0，发送VRRP通告信息；否则判断数据的优先级是否高于本机，或相等而且实际IP地址大于本地实际IP，设置定时通告定时器，复位主机超时定时器，转BACKUP状态；否则的话，丢弃该通告包；

(3).Backup

- 设置主机超时定时器；
- 不能响应针对虚拟路由器IP的ARP请求信息；
- 丢弃所有目的MAC地址是虚拟路由器MAC地址的数据包；
- 不接受目的是虚拟路由器IP的所有数据包；
- 当收到shutdown的事件时删除主机超时定时器，转初始化状态；
- 主机超时定时器超时的时候，发送VRRP通告信息，广播ARP地址信息，转MASTER状态；
- 收到VRRP通告信息时，如果优先权为0，表示进入MASTER选举；否则判断数据的优先级是否高于本机，如果高的话承认MASTER有效，复位主机超时定时器；否则的话，丢弃该通告包；

5.ARP查询处理

​       当内部主机通过ARP查询虚拟路由器IP地址对应的MAC地址时，MASTER路由器回复的MAC地址为虚拟的VRRP的MAC地址，而不是实际网卡的 MAC地址，这样在路由器切换时让内网机器觉察不到；而在路由器重新启动时，不能主动发送本机网卡的实际MAC地址。如果虚拟路由器开启的ARP代理 (proxy_arp)功能，代理的ARP回应也回应VRRP虚拟MAC地址；好了VRRP的简单讲解就到这里，我们下来讲解一下Keepalived的案例。



## keepalived + LVS配置

示例1. 

参考地址:http://blog.csdn.net/wngua/article/details/54668794

> 总结:
>
> 1. keepalived 底层依赖了LVS, 但是并不是安装ipvsadm来配置LVS,而是用配置文件来代替ipvsadm来配置LVS;
> 2. ​

```
如果你没有配置LVS+keepalived那么无需配置这段区域，里如果你用的是nginx来代替LVS，这无限配置这款，这里的LVS配置是专门为keepalived+LVS集成准备的。
注意了，这里LVS配置并不是指真的安装LVS然后用ipvsadm来配置他，而是用keepalived的配置文件来代替ipvsadm来配置LVS，这样会方便很多，一个配置文件搞定这些，维护方便，配置方便是也！

这里LVS配置也有两个配置
一个是虚拟主机组配置
一个是虚拟主机配置

1，虚拟主机组配置文件详解
这个配置是可选的，根据需求来配置吧，这里配置主要是为了让一台realserver上的某个服务可以属于多个Virtual Server，并且只做一次健康检查

virtual_server_group <STRING> {
# VIP port
<IPADDR> <PORT>
<IPADDR> <PORT>
fwmark <INT>
}

2，虚拟主机配置

virtual server可以以下面三种的任意一种来配置
1. virtual server IP port
2. virtual server fwmark int
3. virtual server group string
复制代码
下面以第一种比较常用的方式来配详细解说一下

virtual_server 192.168.1.2 80 {                     #设置一个virtual server: VIP:Vport
delay_loop 3                                                  # service polling的delay时间，即服务轮询的时间间隔

lb_algo rr|wrr|lc|wlc|lblc|sh|dh                        #LVS调度算法
lb_kind NAT|DR|TUN                                      #LVS集群模式                      
persistence_timeout 120                                #会话保持时间（秒为单位），即以用户在120秒内被分配到同一个后端realserver
persistence_granularity <NETMASK>              #LVS会话保持粒度，ipvsadm中的-M参数，默认是0xffffffff，即每个客户端都做会话保持
protocol TCP                                                  #健康检查用的是TCP还是UDP
ha_suspend                                                   #suspendhealthchecker’s activity
virtualhost <string>                                       #HTTP_GET做健康检查时，检查的web服务器的虚拟主机（即host：头）

sorry_server <IPADDR> <PORT>                 #备用机，就是当所有后端realserver节点都不可用时，就用这里设置的，也就是临时把所有的请求都发送到这里啦

real_server <IPADDR> <PORT>                    #后端真实节点主机的权重等设置，主要，后端有几台这里就要设置几个
{
weight 1                                                         #给每台的权重，0表示失效(不知给他转发请求知道他恢复正常)，默认是1
inhibit_on_failure                                            #表示在节点失败后，把他权重设置成0，而不是冲IPVS中删除

notify_up <STRING> | <QUOTED-STRING>  #检查服务器正常(UP)后，要执行的脚本
notify_down <STRING> | <QUOTED-STRING> #检查服务器失败(down)后，要执行的脚本

HTTP_GET                                                     #健康检查方式
{
url {                                                                #要坚持的URL，可以有多个
path /                                                             #具体路径
digest <STRING>                                            
status_code 200                                            #返回状态码
}
connect_port 80                                            #监控检查的端口

bindto <IPADD>                                             #健康检查的IP地址
connect_timeout   3                                       #连接超时时间
nb_get_retry 3                                               #重连次数
delay_before_retry 2                                      #重连间隔
} # END OF HTTP_GET|SSL_GET


#下面是常用的健康检查方式，健康检查方式一共有HTTP_GET|SSL_GET|TCP_CHECK|SMTP_CHECK|MISC_CHECK这些
#TCP方式
TCP_CHECK {
  connect_port 80
  bindto 192.168.1.1
  connect_timeout 4
} # TCP_CHECK

# SMTP方式，这个可以用来给邮件服务器做集群
SMTP_CHECK
host {
  connect_ip <IP ADDRESS>
  connect_port <PORT>                                     #默认检查25端口
  14 KEEPALIVED
  bindto <IP ADDRESS>
}
connect_timeout <INTEGER>
  retry <INTEGER>
  delay_before_retry <INTEGER>
  # "smtp HELO"ž|·-ëê§Œà"
  helo_name <STRING>|<QUOTED-STRING>
} #SMTP_CHECK

#MISC方式，这个可以用来检查很多服务器只需要自己会些脚本即可
MISC_CHECK
{
misc_path <STRING>|<QUOTED-STRING> #外部程序或脚本
misc_timeout <INT>                                    #脚本或程序执行超时时间

misc_dynamic                                               #这个就很好用了，可以非常精确的来调整权重，是后端每天服务器的压力都能均衡调配，这个主要是通过执行的程序或脚本返回的状态代码来动态调整weight值，使权重根据真实的后端压力来适当调整，不过这需要有过硬的脚本功夫才行哦
#返回0：健康检查没问题，不修改权重
#返回1：健康检查失败，权重设置为0
#返回2-255：健康检查没问题，但是权重却要根据返回代码修改为返回码-2，例如如果程序或脚本执行后返回的代码为200，#那么权重这回被修改为 200-2
}
} # Realserver
} # Virtual Server

配置文件到此就讲完了，下面是一份未加备注的完整配置文件
global_defs
{
	notification_email
	{
		admin@example.com
	}
	notification_email_from admin@example.com
	smtp_server 127.0.0.1
	stmp_connect_timeout 30
	router_id node1
}
notification_email
{
  admin@example.com
  admin@ywlm.net
}

static_ipaddress
{
	192.168.1.1/24 brd + dev eth0 scope global
	192.168.1.2/24 brd + dev eth1 scope global
}
static_routes
{
	src $SRC_IP to $DST_IP dev $SRC_DEVICE
	src $SRC_IP to $DST_IP via $GW dev $SRC_DEVICE
}

vrrp_sync_group VG_1 {
  group {
      http
      mysql
  }
  notify_master /path/to/to_master.sh
  notify_backup /path_to/to_backup.sh
  notify_fault "/path/fault.sh VG_1"
  notify /path/to/notify.sh
  smtp_alert
}
group {
  http
  mysql
}
vrrp_script check_running {
   script "/usr/local/bin/check_running"
   interval 10
   weight 10
}

vrrp_instance http {
  state MASTER
  interface eth0
  dont_track_primary
  track_interface {
    eth0
    eth1
  }
  mcast_src_ip <IPADDR>
  garp_master_delay 10
  virtual_router_id 51
  priority 100
  advert_int 1
  authentication {
  auth_type PASS
  autp_pass 1234
}
virtual_ipaddress {
  #<IPADDR>/<MASK> brd <IPADDR> dev <STRING> scope <SCOPT> label <LABEL>
  192.168.200.17/24 dev eth1
  192.168.200.18/24 dev eth2 label eth2:1
}
virtual_routes {
  # src <IPADDR> [to] <IPADDR>/<MASK> via|gw <IPADDR> dev <STRING> scope <SCOPE> tab
  src 192.168.100.1 to 192.168.109.0/24 via 192.168.200.254 dev eth1
  192.168.110.0/24 via 192.168.200.254 dev eth1
  192.168.111.0/24 dev eth2
  192.168.112.0/24 via 192.168.100.254
}
track_script {
	check_running weight 20
}

nopreempt
preemtp_delay 300
debug
}

virtual_server_group <STRING> {
  # VIP port
  < IPADDR> <PORT>
  < IPADDR> <PORT>
  fwmark <INT>
}
virtual_server 192.168.1.2 80 {
delay_loop 3
lb_algo rr|wrr|lc|wlc|lblc|sh|dh
lb_kind NAT|DR|TUN
persistence_timeout 120
persistence_granularity <NETMASK>
protocol TCP
ha_suspend
virtualhost <string>

sorry_server <IPADDR> <PORT>

real_server <IPADDR> <PORT>
{
weight 1
inhibit_on_failure 
notify_up <STRING> | <QUOTED-STRING>
notify_down <STRING> | <QUOTED-STRING>

#HTTP_GET方式
HTTP_GET | SSL_GET
{
url { 
path / 
digest <STRING>                                            
status_code 200
}
connect_port 80 

bindto <IPADD>
connect_timeout   3
nb_get_retry 3
delay_before_retry 2
} 
}
}
```

示例2:

参考地址:http://www.linuxidc.com/Linux/2013-07/86889.htm

```
下面是keepalived详细配置文件解析：

[root@localhost kernels]# cat /etc/keepalived/keepalived.conf 
! Configuration File for keepalived 
global_defs { 
#  notification_email { 
#    acassen@firewall.loc 
#    failover@firewall.loc 
#    sysadmin@firewall.loc 
#  } 
#  notification_email_from Alexandre.Cassen@firewall.loc 
#  smtp_server 192.168.200.1
#  smtp_connect_timeout 30
router_id LVS_DEVEL    //负载均衡器标识，同一网段内，可以相同 
} 
vrrp_sync_group VGM {  //定义一个vrrp组 
  group { 
  	VI_1 
  } 
} 
vrrp_instance VI_1 {    //定义vrrp实例 
state MASTER        	//主LVS是MASTER,从的BACKUP 
interface eth0      	//LVS监控的网络接口 
virtual_router_id 51  	//同一实例下virtual_router_id必须相同 
priority 100            //定义优先级，数字越大，优先级越高 
advert_int 5          	//MASTER与BACKUP负载均衡器之间同步检查的时间间隔，单位是秒 
  authentication {      //验证类型和密码 
  					  //认证类型有PASS和AH（IPSEC），通常使用的类型为PASS，同一vrrp实例MASTER
  					  //与BACKUP 使用相同的密码才能正常通信
    auth_type PASS 
    auth_pass 1111
  } 
  virtual_ipaddress {    //虚拟IP ##可以有多个VIP地址，每个地址占一行，不需要指定子网掩码，必须与RealServer上设定的VIP相一致
    192.168.1.8
    #192.168.1.9    //如果有多个，往下加就行了 
    #192.168.1.7
  } 
} 
virtual_server 192.168.1.8 80 {    //定义虚拟服务器 
  delay_loop 6                  //健康检查时间，单位是秒 
  lb_algo rr              //负载调度算法，这里设置为rr，即轮询算法 
  lb_kind DR              //LVS实现负载均衡的机制，可以有NAT、TUN和DR三个模式可选 
  persistence_timeout 50        //会话保持时间，单位是秒（可以适当延长时间以保持session） 
  protocol TCP                  //转发协议类型，有tcp和udp两种 
  sorry_server 127.0.0.1 80      //web服务器全部失败，vip指向本机80端口 
  real_server 192.168.1.16 80 {  //定义WEB服务器 
  weight 1                  //权重 
  TCP_CHECK {                //通过tcpcheck判断RealServer的健康状态 
    connect_timeout 5      //连接超时时间 
    nb_get_retry 3        //重连次数 
    delay_before_retry 3  //重连间隔时间 
    connect_port 80        //检测端口 
  } 
} 
  real_server 192.168.1.17 80 { 
    weight 1
    TCP_CHECK { 
      connect_timeout 5
      nb_get_retry 3
      delay_before_retry 3
      connect_port 80
    } 
  } 
}
```

ABC的配置:(主)

```
! Configuration File forkeepalived
global_defs {
    smtp_server 127.0.0.1
    smtp_connect_timeout 30
    router_id MYSQL_HA        #标识，不同keepalived实例router_id不能相同
}
vrrp_instance VI_1 {		 #定义vrrp实例 
    state BACKUP              #两台都设置BACKUP
    interface bond0           #虚IP要绑定的端口
    virtual_router_id 32      #主备相同，不同实例该值不能相同, //sun:同一实例下virtual_router_id必须相同 
    priority 100              #优先级，backup设置90
    advert_int 1			 //MASTER与BACKUP负载均衡器之间同步检查的时间间隔，单位是秒 
    nopreempt                 #不主动抢占资源
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.143.32        #虚IP
    }
}
virtual_server VIP 3306 {
    delay_loop 2			 # service polling的delay时间，即服务轮询的时间间隔
    #lb_algo rr               #LVS算法，用不到，我们就关闭了
    #lb_kind DR               #LVS模式，如果不关闭，备用服务器不能通过VIP连接主MySQL
    persistence_timeout 50    #同一IP的连接60秒内被分配到同一台真实服务器
    protocol TCP			 #健康检查用的是TCP还是UDP
    real_server 192.168.143.65 3306 {   #检测本地mysql，backup也要写检测本地mysql; //sun:此时mysql的master容器的节点的ip地址是192.168.143.65
            weight 3		#sun:权重
            notify_down /etc/keepalived/mysql.sh    #当mysq服务down时，执行此脚本，杀死keepalived实现切换
            TCP_CHECK {
                connect_timeout 3    #连接超时
                #nb_get_retry 3       #重试次数
                #delay_before_retry 3 #重试间隔时间
    }
}
```

备:

```
! Configuration File forkeepalived
global_defs {
    smtp_server 127.0.0.1
    smtp_connect_timeout 30
    router_id MYSQL_HA        #标识，不同keepalived实例router_id不能相同
}
vrrp_instance VI_1 {
    state BACKUP              #两台都设置BACKUP
    interface bond0            #虚IP要绑定的端口
    virtual_router_id 32      #主备相同，不同实例该值不能相同
    priority 90              #优先级，backup设置90
    advert_int 1
    nopreempt                 #不主动抢占资源
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.143.32        #虚IP
    }
}
virtual_server VIP 3306 {
    delay_loop 2
    #lb_algo rr               #LVS算法，用不到，我们就关闭了
    #lb_kind DR              #LVS模式，如果不关闭，备用服务器不能通过VIP连接主MySQL
    persistence_timeout 50  #同一IP的连接60秒内被分配到同一台真实服务器
    protocol TCP
    real_server 192.168.143.60 3306 {   #检测本地mysql，backup也要写检测本地mysql
            weight 3
            notify_down /etc/keepalived/mysql.sh    #当mysq服务down时，执行此脚本，杀死keepalived实现切换
            TCP_CHECK {
                connect_timeout 3    #连接超时
                #nb_get_retry 3       #重试次数
                #delay_before_retry 3 #重试间隔时间
    }
}
```

经过对比,发现,两份配置文件的区别的地方就是:

1. priority不同, 主的priority 是100,备的priority是90;
2. real_server对应的ip地址不同,  主的对应的是mysql-master pod所在的物理机ip:143.64,而备对应的msql-slave pod所在的物理机的ip地址:143.60;

http://freeloda.blog.51cto.com/2033581/1280962