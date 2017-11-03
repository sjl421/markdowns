# iptables 详解

## iptables/netfilter 介绍//TODO

##基本概念:

1. 防火墙工作在主机边缘:

对于进出本网络或者本主机的数据报文，根据事先设定好的检查规则对其检查，对形迹可疑的报文一律按照事先定义好的处理机制做出相应处理, 对linux而言tcp/ip协议栈是在内核当中，意味着报文的处理是在内核中处理的，也就是说防火墙必须在工作在内核中，防火墙必须在内核中完成tcp/ip报文所流进的位置，用规则去检查，才真正能工作起来。

iptables用来衡量tcp/ip报文的属性：源ip、目标ip、源端口、目标端口；

tcp标志位:   syn、syn+ack、ack、 fin、urg、psh、rst ；

2. 应用网关

众多代理服务器都是应用网关，比如squid（使用acl限制应用层）varish这一类代理服务等。

3. 入侵检测系统（IDS）：

* 网络入侵检测系统  NIDS
* 主机入侵检测系统  HIDS
* 对于IDS常用的检测服务有：snort等

4. 入侵防御系统（IPS），比如蜜罐

部署一套入侵检测系统是非常麻烦的，因为必须检测网络任意一个位置, 对于IPS常用的检测服务有： tripwire 等

##iptables基本概念

对linux来说，是能够实现主机防火墙的功能组件，如果部署在网络边缘，那么既可以扮演网络防火墙的角色，而且是纯软件的

网络数据走向：

请求报文 -> 网关 -> 路由 -> 应用程序（等待用户请求）-> 内核处理 -> 路由 -> 发送报文

###iptables规则功能

表:

filter主要和主机自身有关，主要负责防火墙功能 过滤本机流入流出的数据包是默认使用的表;

input   :负责过滤所有目标地址是本机地址的数据包，就是过滤进入主机的数据包;

forward  :负责转发流经主机但不进入本机的数据包，和NAT关系很大;

output   :负责处理源地址的数据包，就是对本机发出的数据包;

**NAT表:**

负责网络地址转换，即来源于目的IP地址和端口的转换，一般用于共享上网或特殊端口的转换服务

snat    :地址转换

dnat    :标地址转换

pnat    :标端口转换

**mangle 表：**

将报文拆开来并修改报文标志位，最后封装起来

**5个检查点（内置链）**

* PREROUTING

* INPUT

* FORWORD

* OUTPUT

* POSTROUTING

  多条链整合起来叫做表，比如，在input这个链，既有magle的规则也可能有fileter的规则。因此在编写规则的时候应该先指定表，再指定链. 

  netfilter主要工作在tcp/ip协议栈上的，主要集中在tcp报文首部和udp报文首部

  **(搞清楚表跟链的区别)**

**规则的属性定义：**

1.网络层协议

​	主要集中在ip协议报文上

2.传输层协议属性：

​	主要集中在 

*  tcp,
*  udp
*  icmp  icmp其并不是真正意义传输层的，而是工作在网络层和传输层之间的一种特殊的协议

3.ip报文的属性：

​	IP报文的属性为: 源地址.目标地址

---

iptables 规则太常用，之前一直没有好好学习规则格式，专门收集，整理了下，方便查询：

```
iptables定义规则:格式：iptables [-t table] COMMAND chain CRETIRIA -j ACTION
iptables [-t表名]  <链名> <动作>
-t table ：指定设置的表：filter nat mangle  不指定默认为filter
COMMAND：定义如何对规则进行管理
chain：指定你接下来的规则到底是在哪个链上操作的，当定义策略的时候，可以省略
CRETIRIA:指定匹配标准
-j ACTION :指定如何进行处理
```

```
COMMAND详解：
-P :设置默认策略（设定默认门是关着的还是开着的）   
     iptables-t fllter -P INPUT DROP #设置fllter表input链的默认规则为丢弃
-F: FLASH，清空规则链     
     iptables -t nat -F PREROUTING 
     iptables -t nat -F 清空nat表的所有规则
-Z：清空链   iptables -Z :清空
-A：追加，在当前链的最后新增一个规则
-I num : 插入，把当前规则插入为第几条  
     -I 3
-R num：Replays替换/修改第几条规则   
     iptables -R 3 …………
-D num：删除，明确指定删除第几条规则   
     iptables -D INPUT 12
-L 查看规则，下面包含子命令：
     -n：以数字的方式显示ip，它会将ip直接显示出来，如果不加-n，则会将ip反向解析成主机名。
     -v：显示详细信息
     --line-numbers : 显示规则的行号
     通常组合使用的命令是: -nvL
-N ： 新建一条自定义链（内置链不能删除，如果太多，可以自定义链）
      [root@s2 ~]# iptables -t filter -N MYLAN
      [root@s2 ~]# iptables -A FORWARD -s 192.168.1.0/24 -j MYLAN
      [root@s2 ~]# iptables -A FORWARD -d 192.168.1.0/24 -j MYLAN
      [root@s2 ~]# iptables -A MYLAN -p icmp -j DROP
-X : 删除自定义空链，如果链内有规则，则无法删除
-E ：重命名自定义链		
```

**iptables 规则管理**

```
CRETIRIA 详解：
1.通用匹配：源地址目标地址的匹配
     -s：指定作为源地址匹配，这里不能指定主机名称，必须是IP，地址可以取反，加一个“!”表示除了哪个IP之外
     -d：表示匹配目标地址
     -p：用于匹配协议的（这里的协议通常有3种，TCP/UDP/ICMP）
     -i eth0：从这块网卡流入的数据   （流入一般用在INPUT和PREROUTING上）
     -o eth0：从这块网卡流出的数据  （流出一般在OUTPUT和POSTROUTING上）
2.扩展匹配（隐含扩展：对协议的扩展 和显式扩展（-m））
     -p tcp :TCP协议的扩展
       --dport XX-XX：指定目标端口,不能指定多个非连续端口,只能指定单个端口
       --sport：指定源端口
       --tcp-fiags：TCP的标志位匹配（SYN,ACK，FIN,PSH，RST,URG）（使用相对较少）
       --tcpflags syn,ack,fin,rst syn  用于检测三次握手的第一次包，可简写为--syn
     -p udp：UDP协议的扩展  （与tcp相似）
     -p icmp：icmp数据报文的扩展
     -m multiport：表示启用多端口扩展  --dports 21,23,80
```

```
-j 	ACTION 详解：
    DROP：表示丢弃
    REJECT：明示拒绝
    ACCEPT：接受
    MASQUERADE：源地址伪装
    REDIRECT：重定向：主要用于实现端口重定向
    RETURN：返回：在自定义链执行完毕后使用返回，来返回原规则链
    DNAT: 目标地址转换
	SNAT: 源地址转换
	MARK: 打标签,以便提供作为后续过滤的条件判断依据
	LOG  
	custom_chain：转向一个自定义的链
```

**对于严谨的规则，一般默认规则都是拒绝未知，允许已知**

示例:

```
[root@test3xtables-1.4.7]# iptables -A INPUT -s 10.0.10.0/24 -d 10.0.10.0/24 -j ACCEPT
[root@test3 xtables-1.4.7]# iptables -L -n -v
ChainINPUT (policy ACCEPT 10 packets, 1029 bytes)

pkts bytestarget    prot opt  in    out      source                destination

22  1660    ACCEPT     all  --  *      *       10.0.10.0/24                10.0.10.0/24 

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
pkts bytes target     prot opt in    out       source                   destination 

Chain OUTPUT (policy ACCEPT 16 packets, 1536 bytes)
pkts bytes target     prot opt in    out       source                  destination    
```

各输出字段的含义:

```
pkts     被本机报文所匹配的个数
bytes   报文所有大小记起来之和
opt     额外的选项，--表示没有
target   处理机制
prot     放行哪种协议
source  源地址
destination  目标地址
```

**1. 扩展匹配** 

所有的扩展匹配表示要使用-m来指定扩展的名称来引用，而每个扩展模块一般都会有自己特有的专用选项，在这些选项中，有些是必备的：

```
#端口之间必须是连续的
-p tcp --sport | --dport 21-80
#取反，非21-80的端口
-p tcp --sport | --dport !21-80
#检测报文中的标志位
--tcp-flags SYN,ACK,RST,FIN, SYN

--tcp-flags ALL NONE   #表示所有标志位都检测，但是其中所有都为0
--tcp-flage ALL SYN,FIN #表示SYN,FIN都为1（即握手又断开）
```

例：放行本机对web的访问

```
[root@test3~]# iptables -A INPUT -d 10.0.10.62  -ptcp --dport 80 -j ACCEPT
[root@test3~]# iptables -L -n
ChainINPUT (policy DROP)
target     prot opt source               destination        
ACCEPT     all --  10.0.10.0/24         10.0.10.62          
ACCEPT     tcp --  0.0.0.0/0            10.0.10.62          tcp dpt:80
```

放行出去的报文，源端口为80

```
[root@test3~]# iptables -A OUTPUT -s 10.0.10.62 -p tcp --sport 80 -j ACCEPT
```

查看匹配规则:

```
[root@test3 ~]# iptables -L -n --line-number
ChainINPUT (policy DROP)
num  target    prot opt source              destination        
1    ACCEPT    all  --  10.0.10.0/24         10.0.10.62          
2    ACCEPT    tcp  --  0.0.0.0/0            10.0.10.62          tcp dpt:80

ChainFORWARD (policy DROP)
num  target    prot opt source              destination       
ChainOUTPUT (policy DROP)
num  target    prot opt source              destination        
1    ACCEPT    all  --  10.0.10.0/24         10.0.10.0/24        
2    ACCEPT    tcp  --  10.0.10.62           0.0.0.0/0           tcp spt:80
```

**2. 协议匹配**

通常对协议做匹配则使用 -p 参数 来指定协议即可

匹配UDP：UDP只有端口的匹配，没有任何可用扩展，格式如下:

```
-p udp--sport | --dport
```

匹配ICMP格式如下:

```
-picmp --icmp-[number]
icmp常见类型：请求为8（echo-request），响应为0(echo-reply)
```

例：默认规则input output 都为DROP,使其本机能ping（响应的报文）的报文出去\

通过此机器去ping网关10.0.10.1 ， 可结果却提示not permitted，使其能通10.0.10.0/24网段中的所有主机

```
[root@test3~]#iptables -A OUTPUT -s 10.0.10.62 -d 10.0.10.0/24 -p icmp --icmp-type8 -j ACCEPT
```

可看到无法响应：0表示响应进来的报文规则，并没有放行自己作为服务端的的角色规则

```
[root@test3~]# iptables -A INPUT -s 10.0.10.0/24 -d 10.0.10.62 -p icmp --icmp-type0 -j ACCEPT
#ping 10.0.10.x
```

允许类型为0（响应报文）出去

```
[root@test3~]# iptables -A OUTPUT -s 10.0.10.62 -d  10.0.10.0/24 -picmp --icmp-type 0 -j ACCEPT
```

例2：本机DNS服务器，要为本地客户端做递归查询；iptables的input output默认为drop 本机地址是10.0.10.62

```
[root@test3~]# iptables -A INPUT -d 10.0.10.62 -p udp --dprot 53 -j ACCEPT
[root@test3~]# iptables -A OUTPUT -S 10.0.10.62 -p udp --sprot 53 -j ACCEPT
```

客户端请求可以进来，响应也可以出去，但是自己作为客户端请求别人是没有办法出去的，所以：

```
[root@test3~]# iptables -A OUTPUT -s 10.0.10.62 -p udp --dport 53 -j ACCEPT
[root@test3~]# iptables -A INPUT -d 10.0.10.62 -p udp --sprot 53 -j ACCEPT
```

**2.3 TCP****协议的报文走向**

![img](http://s3.51cto.com/wyfs02/M00/12/00/wKiom1Lw0HCgK5tEAAI5zcbKr-8165.jpg?_=3540837)

TCP连接的建立

双方主机为了实现tcp的通信，所以首先三次握手

客户端主动发出了SYN，服务器端处于监听状态，随时等待客户端的请求信息；

服务器端接收到了SYN请求，从而回应用户的请求，发送SYN_ACK ，从而转换为SYN_REVIN

客户端在发出了请求，从发出的那一刻close状态转换为SYN_SENT状态

客户端在SYN_SENT状态中一旦收到了服务端发来的SYN_ACK 之后，转换为ESTABLISHED状态，这时便可以开始传送数据了，无论怎么传都是ESTABLISHED状态

而服务器端收到了对方的ACK，同样处于ESTABLISHED状态

 

数据传输结束之后

客户端从ESTABLEISHED状态，发起四次断开请求

客户端发起FIN请求，从而进入等待状态

服务端收到断开请求之后，便发起ACK请求

客户端收到服务端发来的ACK确认信息后，从而又发起FIN_2 请求

等待服务端发来的FIN请求之后，便确认

服务器端收到FIN并发送ACK之后，服务器端便处于CLOSE_WAIT便自己发送FIN，从而进入LAST ACK模式 ，

确认完后不能立刻断开，还需要等待一定的时间（大约240秒），确认报文是否传递给对方

于是转换为CLOSED

![img](http://s3.51cto.com/wyfs02/M02/11/FF/wKioL1Lw0GPTLv-bAAKAtO7srDI767.jpg?_=3540837)

iptables中有一个扩张参数--status

此扩展可以追踪tcp udp icmp等各种状态

其能够使用某种内核数据结构保持此前曾经建立的连接状态时间的功能，称为连接追踪

内核参数文件路径为：

[root@test3~]# ls /proc/sys/net/netfilter/

[root@test3~]# cat /proc/sys/net/netfilter/nf_conntrack_udp_timeout
30

以此为例，在其30秒钟内，曾经建立过的udp连接,这些连接都可以被追踪到的，可以明确知道在这期间哪个客户端曾经访问过，只要基于请求的序列，能跟此前保持会话信息，即可查询

 

**2.4****显式扩展**

在iptalbes中数据包和被跟踪连接的4种不同状态相关联，这四种状态分别是NEW、ESTABLISHED、RELATED及INVALID.

除了本机产生的数据包由NAT表的OUTPUT链处理外，所有连接跟踪都是在NAT表的PREROUTING链中进行处理的，也就是说iptables在NAT表的PREROUTING链里从新计算所有数据包的状态。如果发送一个流的初始化数据包，状态就会在NAT表的OUTPUT链里被设置为NEW，当收到回应的数据包时，状态就会在NAT表的PREROUTING链里被设置为ESTABLISHED，如果第一个数据包不是本机生成的，那就回在NAT表PREROUTING链里被设置为NEW状态，所以所有状态的改变和计算都是在NAT表中的表链和OUTPUT链里完成的。

使用-m来指定其状态并赋予匹配规则，语法如下

-mstate --state 状态

* NEW
* ESTABLISHED
* RELATED          
* INVALID

NEW：

NEW状态的数据包说明这个数据包是收到的第一个数据包。比如收到一个SYN数据包，它是连接的第一个数据包，就会匹配NEW状态。第一个包也可能不是SYN包，但它仍会被认为是NEW状态。

ESTABLISHED：

只要发送并接到应答，一个数据连接就从NEW变为ESTABLISHED,而且该状态会继续匹配这个连接后继数据包。

RELATED：

当一个连接和某个已处于ESTABLISHED状态的连接有关系时，就被认为是RELATED，也就是说，一个连接想要是RELATED的，首先要有个ESTABLISHED的连接，这个ESTABLISHED连接再产生一个主连接之外的连接，这个新的连接就是RELATED。

INVALID：

INVALID状态说明数据包不能被识别属于哪个连接或没有任何状态。

例:

对本机22端口做状态监测：

进来的请求状态为new，而出去的状态则为ESTABLISHED，如果自动连接别人 状态肯定为NEW，如果正常去响应别人那么状态肯定是ESTABLISHED

```
[root@test3~]# iptables -A OUTPUT -s 10.0.10.62 -d 10.0.10.0/24 -p tcp --dport 22 -m state--state ESTABLISHED -j ACCEPT

[root@test3~]# iptables -L -n
ChainINPUT (policy ACCEPT)
target     prot opt source               destination        
ACCEPT     tcp --  10.0.10.0/24         10.0.10.62          tcp dpt:22 state NEW,ESTABLISHED

ChainFORWARD (policy DROP)
target     prot opt source               destination        
ChainOUTPUT (policy ACCEPT)
target     prot opt source               destination        
ACCEPT     tcp  -- 10.0.10.62          10.0.10.0/24        tcp dpt:22state ESTABLISHED
```

**多端口规则匹配**

使用参数-m multiport 可以指定15个以内的非连续端口，比如21-22,80

```
-mmulitport  
--src-prots
--dst-ports
--prots
```

对多端口进行匹配，只要匹配以下端口，则全部放行:

```
[root@test3~]# iptables -A INPUT  -s 10.0.10.0/24 -d10.0.10.62 -p tcp -m state --state NEW  -m mulitport--destination-ports 21,22,80 -j ACCEPT
```

多IP匹配,指定匹配的IP地址范围：

   -miprange

   --src-range

   --dst-range

指定匹配的连续ip段

```
[root@test3~]# iptables -A INPUT -s  -m iprange --src-range 10.0.10.100-10.0.10.200
```

默认为每秒匹配3个报文，基于令牌桶算法

   -mlimit

   --limit             	#NUMBER，表示允许收集多少个空闲令牌

   --limit-burst          #RATE，允许放行多少个报文

比如：ssh一分钟之内只能建立20个链接，平均5秒一个，而一次性只能放行2个空闲令牌

   --limit 20/min

   --limit-burst 2

只有在大量空闲令牌存储的情况下，才可有limit-burst控制

例：控制NEW状态的请求

```
[root@test3~]# iptables -A INPUT -s 10.0.10.0/24 -d 10.0.10.62 -m state --state NEW -mlimit --limit 12/min --limit 12/min --limit-burst 2 -j ACCEPT
```

例2：每次只允许2个ping包进来

```
[root@test3~]# iptables -F
[root@test3~]# iptables -A INPUT -s 10.0.10.0/24 -d 10.0.10.62 -p icmp --icmp-type 8 -mlimit --limit 20/min --limit-burst 5 -j ACCEPT
新建立一终端，在其终端ping10.0.10.62可以看到效果，不再演示
```

**2.5****对应用层进行匹配**

对应用层编码字符串做相似匹配，常用算法使用--alog来指定 ，一般来讲算法一般为bm和kmp

```
-msrting  
--string ""
--algo {bm|kmp}
```

例：

·假如我们期望web站点页面中任何包含"hello"的字符串的页面，则禁止访问，其他则放行

·请求报文中不会包含hello，一般来讲只包含访问某个页面，那么请求内容无非包含了请求某个链接而已

·响应报文中会封装页面的内容信息，因此 会出现在响应报文中，而不是请求报文

```
启动httpd服务
[root@test3~]# /etc/init.d/httpd start
在web站点新建页面1.html，内容为"hello" ， 2.html内容为"word"
[root@test3domian]# echo hello > 1.html
[root@test3domian]# echo word > 2.html

在iptables的允许放行规则前面加一条更严谨的禁止规则：
[root@test3domian]# iptables -A OUTPUT -s 10.0.10.62 -p tcp --sport 80 -m string --string"hello" --algo kmp -j REJECT

再次访问
[root@test3domian]# curl -dump http://10.0.10.62/2.html
word
[root@test3domian]# curl -dump http://10.0.10.62/1html

#请求已发出去但是一直没有反应，我们来看一下防火墙规则是否被匹配到
[root@test3domian]# iptables -L -nv
ChainINPUT (policy ACCEPT 255 packets, 30024 bytes)
pkts bytes target     prot opt in     out    source               destination        
ChainFORWARD (policy DROP 0 packets, 0 bytes)
pkts bytes target     prot opt in     out    source              destination        
ChainOUTPUT (policy ACCEPT 201 packets, 29406 bytes)
pkts bytes target     prot opt in     out    source              destination        
  35 11209 REJECT     tcp --  *      *      10.0.10.62          0.0.0.0/0           tcp spt:80STRING match "hello" ALGO name kmp TO 65535 reject-withicmp-port-unreachable
```

**基于时间限定**

-m time

\#指定日期起止范围

   --datestart

   --datestop

\#指定时间的起止范围

   --timestart

   --timestop

\#指定星期x范围

   --weekdays

\#指定月份

   --monthdays

### 添加注释 -m comment --comment "xxx"

Comments appear as follows when in use. (Ex: /* allow SSH to this host from anywhere */ as seen below.)

```
$ sudo iptables -L
Chain INPUT (policy DROP)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             anywhere             state RELATED,ESTABLISHED /* allow inbound traffic for established and related connections */
fail2ban-ssh  tcp  --  anywhere             anywhere             multiport dports ssh
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:ssh /* allow SSH to this host from anywhere */
```

**添加规则并添加注释**

```
$ sudo iptables -A INPUT -p tcp -m tcp --dport 22 -m comment --comment "allow SSH to this host from anywhere" -j ACCEPT
```

```
$ sudo iptables -L
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:ssh /* allow SSH to this host from anywhere */
```

---

##关于IPTABLES 各种MARK 功能的用法

iptalbes 的有多个MARK 模块..用法各不相同..一直没有完全明白..希望高手解释一下各功能的使用及区别....

```
-m mark
-m connmark
-j MARK
-j CONNMARK
-j CONNSECMARK
-j SECMARK
解释：
小写的是数据包匹配模块，大写的是数据包修改模块。
带 CONN 的是连接的标记，不带的是标记数据包的。
带 SEC 的是用于处理 IPSEC 数据的，不带的是处理一般数据的。
```

```
CONNMARK target options:
  --set-xmark value[/ctmask]    Zero mask bits and XOR ctmark with value
  --save-mark [--ctmask mask] [--nfmask mask]
                                Copy ctmark to nfmark using masks
  --restore-mark [--ctmask mask] [--nfmask mask]
                                Copy nfmark to ctmark using masks
  --set-mark value[/mask]       Set conntrack mark value
  --save-mark [--mask mask]     Save the packet nfmark in the connection
  --restore-mark [--mask mask]  Restore saved nfmark value
  --and-mark value              Binary AND the ctmark with bits
  --or-mark value               Binary OR  the ctmark with bits
  --xor-mark value              Binary XOR the ctmark with bits
  
在这个模块中..--save-mark,--set-mark,--restore-mark 
这些常用的..如何用..有什么区别呢
--set-mark value[/mask]       Set conntrack mark value
--save-mark [--mask mask]     Save the packet nfmark in the connection
--restore-mark [--mask mask]  Restore saved nfmark value
```

虽然只匹配了每个连接的第一个包，但通过后面两个操作，使得这个连接的每个包都被设置成了 MARK 1

这个功能在 ipp2p 结合 tc 进行限速时特别有用

http://www.ipp2p.org/docu_en.html#example

对于限速来说...包肯定是在某一个连接中...
就象之前我用IPTABLES L7标记文件大小限制速度..那就应该用CONNMARK了..这是对整个连接的包进行限速成了...也正是我们需要的..
但白金大哥说过..L7本身就有连接跟踪的作用..而IP2PP没有...所以得通过上面的方式..转换成第个包...
作了个小节....呵

```
mark 打标记 用 mangle 表
iptables -t mangle -A PREROUTING -mttl --ttl-eq 64 -j MARK --set-mark 10
iptables -t mangle -A PREROUTING -mttl --ttl-eq 123 -j MARK --set-mark 20
iptables -t filter -A FORWARD -m mark--mark 10 -j ACCEPT
iptables -t filter -A FORWARD -m mark--mark 20 -j DROP
打标记的位置很重要
表优先级 mangle --------> nat ------>filter
```

##基于iptables实现NAT功能

**3.1 基于SNAT功能的实现**

考虑场景：为解决IP地址不足，所以用NAT功能来实现成本节约

SNAT：源地址转换（代理内部客户端访问外部网络）在POSTROUTING或OUTPUT链上来做规则限制

参数选项：

​    -j SNAT --to-source IP

​    -j MASQUERADE

DNAT ：目标地址转换（将内部服务器公开至外部网络）需在PREROUTING做限制

参数选项：

   -j DNAT --to-destination IP:prot

NAT不但可以转换目标地址，还可以映射目标端口

拓补图如下：

![img](http://s3.51cto.com/wyfs02/M00/11/FF/wKioL1Lw7qSRlOaVAACmAxfTfhM460.jpg?_=3540837)

假设iptables为网关服务器，192.168.0.0为内网地址段 10.0.10.0 为外网地址段

规划：

| 服务器角色       | 服务器内网IP地址                |
| ----------- | ------------------------ |
| iptables    | 10.0.10.62 、 192.168.0.4 |
| client      | 10.0.10.60               |
| web  server | 192.168.0.110            |

---

### 保存iptable

`iptables-save`  将iptable的输出保存到文件中

`iptables-resotre`  从文件中读取iptable规则

```
[root@test3~]# iptables-save > /tmp/iptables  
加载iptables文件规则
[root@test3~]# iptables-resotre < /tmp/iptables 
```



##常用规则：

1.仅允许指定某IP地址可以SSH连接到服务器：

```
[root@HK /]# iptables -A INPUT -s 192.168.3.7/32 -p tcp --dport 22 -j ACCEPT
[root@HK /]# iptables -A INPUT -p tcp --dport 22 -j DROP
注意最后需要有DROP ，否则无法生效，不要写成了 -I INPUT 
```

2.屏蔽某IP地址：

```
[root@HK /]# iptables -I INPUT -s 192.168.3.7 -j DROP
```

3.放行指定端口的请求：

```
[root@HK /]# iptables -A INPUT  -p tcp --dport 80 -j ACCEPT
如果设置了FTP被动模式，就需要添加被动模式端口进行放行：
-A INPUT -m state --state NEW -m tcp -p tcp --dport 20000:30000 -j ACCEPT
规则写好后记得最后需要拒绝其他的请求，然后保存规则，不执行保存重启服务器后规则失效
[root@HK /]# /etc/init.d/iptables save
```

NAT 规则：
SNAT基于原地址的转换： 常用于配置VPN 等服务

```
iptables -t nat -A POSTROUTING -s 192.168.3.0/24 -j SNAT --to-source 172.16.100.33
意思是将所有192.168.3.0 的请求转换为172.16.100.1  ，对外访问的地址就都变成了172.16.100.1 
```

DNAT目标地址转换：类似于端口映射

```
iptables -t nat -A PREROUTING -d 192.168.3.3 -p tcp --dport 80 -j DNAT --todestination 172.16.100.34
把172.16.100.34服务器80端口的请求转发到服务器192.168.3.3 上
```

\# 1. 删除所有现有规则

```
iptables -F
```

\# 2. 设置默认的 chain 策略

```
iptables -P INPUT DROPiptables -P FORWARD DROPiptables -P OUTPUT DROP
```

\# 3. 阻止某个特定的 IP 地址

```
#BLOCK_THIS_IP="x.x.x.x"#iptables -A INPUT -s "$BLOCK_THIS_IP" -j DRO
```

\# 4. 允许全部进来的（incoming）SSH

```
iptables -A INPUT -i eth0 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPTiptables -A OUTPUT -o eth0 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```

\# 5. 只允许某个特定网络进来的 SSH

```
iptables -A INPUT -i eth0 -p tcp -s 192.168.200.0/24 --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPTiptables -A OUTPUT -o eth0 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```

\# 6. 允许进来的（incoming）HTTP

```
iptables -A INPUT -i eth0 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPTiptables -A OUTPUT -o eth0 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT
```

\# 7. 多端口（允许进来的 SSH、HTTP 和 HTTPS）

```
iptables -A INPUT -i eth0 -p tcp -m multiport --dports 22,80,443 -m state --state NEW,ESTABLISHED -j ACCEPTiptables -A OUTPUT -o eth0 -p tcp -m multiport --sports 22,80,443 -m state --state ESTABLISHED -j ACCEPT
```

\# 8. 允许出去的（outgoing）SSH

```
iptables -A OUTPUT -o eth0 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPTiptables -A INPUT -i eth0 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```

\# 9. 允许外出的（outgoing）SSH，但仅访问某个特定的网络

```
iptables -A OUTPUT -o eth0 -p tcp -d 192.168.101.0/24 --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPTiptables -A INPUT -i eth0 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```

\# 10. 允许外出的（outgoing） HTTPS

```
iptables -A OUTPUT -o eth0 -p tcp --dport 443 -m state --state NEW,ESTABLISHED -j ACCEPTiptables -A INPUT -i eth0 -p tcp --sport 443 -m state --state ESTABLISHED -j ACCEPT
```

\# 11. 对进来的 HTTPS 流量做负载均衡

```
iptables -A PREROUTING -i eth0 -p tcp --dport 443 -m state --state NEW -m nth --counter 0 --every 3 --packet 0 -j DNAT --to-destination 192.168.1.101:443iptables -A PREROUTING -i eth0 -p tcp --dport 443 -m state --state NEW -m nth --counter 0 --every 3 --packet 1 -j DNAT --to-destination 192.168.1.102:443iptables -A PREROUTING -i eth0 -p tcp --dport 443 -m state --state NEW -m nth --counter 0 --every 3 --packet 2 -j DNAT --to-destination 192.168.1.103:443
```

\# 12. 从内部向外部 Ping

```
iptables -A OUTPUT -p icmp --icmp-type echo-request -j ACCEPTiptables -A INPUT -p icmp --icmp-type echo-reply -j ACCEPT
```

\# 13. 从外部向内部 Ping

```
iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPTiptables -A OUTPUT -p icmp --icmp-type echo-reply -j ACCEPT
```

\# 14. 允许环回（loopback）访问

```
iptables -A INPUT -i lo -j ACCEPTiptables -A OUTPUT -o lo -j ACCEPT
```

\# 15. 允许 packets 从内网访问外网

```
if eth1 is connected to external network (internet)if eth0 is connected to internal network (192.168.1.x)iptables -A FORWARD -i eth0 -o eth1 -j ACCEPT
```

\# 16. 允许外出的 DNS

```
iptables -A OUTPUT -p udp -o eth0 --dport 53 -j ACCEPTiptables -A INPUT -p udp -i eth0 --sport 53 -j ACCEPT
```

\# 17. 允许 NIS 连接

```
rpcinfo -p | grep ypbind ; This port is 853 and 850iptables -A INPUT -p tcp --dport 111 -j ACCEPTiptables -A INPUT -p udp --dport 111 -j ACCEPTiptables -A INPUT -p tcp --dport 853 -j ACCEPTiptables -A INPUT -p udp --dport 853 -j ACCEPTiptables -A INPUT -p tcp --dport 850 -j ACCEPTiptables -A INPUT -p udp --dport 850 -j ACCEPT
```

\# 18. 允许某个特定网络 rsync 进入本机

```
iptables -A INPUT -i eth0 -p tcp -s 192.168.101.0/24 --dport 873 -m state --state NEW,ESTABLISHED -j ACCEPTiptables -A OUTPUT -o eth0 -p tcp --sport 873 -m state --state ESTABLISHED -j ACCEPT
```

\# 19. 仅允许来自某个特定网络的 MySQL 的链接

```
iptables -A INPUT -i eth0 -p tcp -s 192.168.200.0/24 --dport 3306 -m state --state NEW,ESTABLISHED -j ACCEPTiptables -A OUTPUT -o eth0 -p tcp --sport 3306 -m state --state ESTABLISHED -j ACCEPT
```

\# 20. 允许 Sendmail 或 Postfix

```
iptables -A INPUT -i eth0 -p tcp --dport 25 -m state --state NEW,ESTABLISHED -j ACCEPTiptables -A OUTPUT -o eth0 -p tcp --sport 25 -m state --state ESTABLISHED -j ACCEPT
```

\# 21. 允许 IMAP 和 IMAPS

```
iptables -A INPUT -i eth0 -p tcp --dport 143 -m state --state NEW,ESTABLISHED -j ACCEPTiptables -A OUTPUT -o eth0 -p tcp --sport 143 -m state --state ESTABLISHED -j ACCEPTiptables -A INPUT -i eth0 -p tcp --dport 993 -m state --state NEW,ESTABLISHED -j ACCEPTiptables -A OUTPUT -o eth0 -p tcp --sport 993 -m state --state ESTABLISHED -j ACCEPT
```

\# 22. 允许 POP3 和 POP3S

```
iptables -A INPUT -i eth0 -p tcp --dport 110 -m state --state NEW,ESTABLISHED -j ACCEPTiptables -A OUTPUT -o eth0 -p tcp --sport 110 -m state --state ESTABLISHED -j ACCEPTiptables -A INPUT -i eth0 -p tcp --dport 995 -m state --state NEW,ESTABLISHED -j ACCEPTiptables -A OUTPUT -o eth0 -p tcp --sport 995 -m state --state ESTABLISHED -j ACCEPT
```

\# 23. 防止 DoS 攻击

```
iptables -A INPUT -p tcp --dport 80 -m limit --limit 25/minute --limit-burst 100 -j ACCEPT
```

\# 24. 设置 422 端口转发到 22 端口

```
iptables -t nat -A PREROUTING -p tcp -d 192.168.102.37 --dport 422 -j DNAT --to 192.168.102.37:22iptables -A INPUT -i eth0 -p tcp --dport 422 -m state --state NEW,ESTABLISHED -j ACCEPTiptables -A OUTPUT -o eth0 -p tcp --sport 422 -m state --state ESTABLISHED -j ACCEPT
```

\# 25. 为丢弃的包做日志（Log）

```
iptables -N LOGGINGiptables -A INPUT -j LOGGINGiptables -A LOGGING -m limit --limit 2/min -j LOG --log-prefix "IPTables Packet Dropped: " --log-level 7iptables -A LOGGING -j DROP
```

**参考**

1. http://www.cnblogs.com/davidwang456/p/3540837.html
2. https://www.cnyunwei.cc/archives/393
3. http://www.qinglin.net/1556.html