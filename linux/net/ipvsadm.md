#ipvsadm

> ipvsadm - Linux Virtual Server administration

事情的缘由是由于集群的内的rm模块没有开放集群外访问的接口,所以在项目的debug的过程中在集群外是没有办法通过rest 来访问rm的,解决的办法(其实有更好的解决办法):

1. 先获取rm模块的pod的podIp:podPort(10.244.1.4:8080);
2. 找一个集群中的能在集群外访问的ip地址,这里用的租户的VIP;
3.  然后通过ipvsadm将一个能在集群外访问的一个ip地址(192.168.143.141) 和一个没有使用的端口号hostPort:31555;
4. 通过ipvsadm将建立VIP:hostPort -> podIp:podPort之间映射;

这是最开始用到的两条命令:

添加映射关系:

```
ipvsadm -A -t 192.168.143.141:31555 -s rr -p 600
ipvsadm -a -t 192.168.143.141:31555 -r 10.244.1.4:8080 -m
```

删除映射关系:

```
ipvsadm -d -t 192.168.143.102:31555 -r 10.244.1.4:8080 
```

查看映射关系:

```
ipvsadm -l -n -c 
```

```
COMMANDS
ipvsadm(8) recognises the commands described below. Upper-case commands maintain virtual services. Lower-case commands maintain real servers that are associated with  a  virtual
service.

-A, --add-service
Add  a  virtual  service.  A service address is uniquely defined by a triplet: IP address, port number, and protocol. Alternatively, a virtual service may be defined by a
firewall-mark.

-E, --edit-service
Edit a virtual service.

-D, --delete-service
Delete a virtual service, along with any associated real servers.

-C, --clear
Clear the virtual server table.

-R, --restore
Restore Linux Virtual Server rules from stdin. Each line read from stdin will be treated as the command line options to a separate invocation of ipvsadm. Lines read  from
stdin can optionally begin with "ipvsadm".  This option is useful to avoid executing a large number or ipvsadm  commands when constructing an extensive routing table.

-S, --save
Dump the Linux Virtual Server rules to stdout in a format that can be read by -R|--restore.

-a, --add-server
Add a real server to a virtual service.

-e, --edit-server
Edit a real server in a virtual service.

-d, --delete-server
Remove a real server from a virtual service.

-L, -l, --list
List  the  virtual server table if no argument is specified. If a service-address is selected, list this service only. If the -c option is selected, then display the con‐
nection table. The exact output is affected by the other arguments given.

-Z, --zero
Zero the packet, byte and rate counters in a service or all services.

--set tcp tcpfin udp
Change the timeout values used for IPVS connections. This command always takes 3 parameters,  representing  the  timeout  values (in seconds) for TCP sessions,  TCP  ses‐
sions after receiving a  FIN packet, and  UDP  packets, respectively.  A timeout value 0 means that the current timeout value of the  corresponding  entry  is preserved.

--start-daemon state
Start the connection synchronization daemon. The state is to indicate that the daemon is started as master or backup. The connection synchronization daemon is implemented
inside the Linux kernel. The master daemon running at the primary load balancer multicasts changes of connections periodically, and  the  backup  daemon  running  at  the
backup  load  balancers  receives  multicast  message  and  creates  corresponding connections. Then, in case the primary load balancer fails, a backup load balancer will
takeover, and it has state of almost all connections, so that almost all established connections can continue to access the service.

The sync daemon currently only supports IPv4 connections.

--stop-daemon
Stop the connection synchronization daemon.

-h, --help
Display a description of the command syntax.
```

```
PARAMETERS
The commands above accept or require zero or more of the following parameters.

-t, --tcp-service service-address
Use TCP service. The service-address is of the form host[:port].  Host may be one of a plain IP address or a hostname. Port may be either a plain port number or the  ser‐
vice  name of port. The Port may be omitted, in which case zero will be used. A Port  of zero is only valid if the service is persistent as the -p|--persistent option, in
which case it is a wild-card port, that is connections will be accepted to any port.


-r, --real-server server-address
Real  server  that an associated request for service may be assigned to.  The server-address is the host address of a real server, and may plus port. Host can be either a
plain IP address or a hostname.  Port can be either a plain port number or the service name of port.  In the case of the masquerading method, the host address is  usually
an  RFC  1918  private IP address, and the port can be different from that of the associated service. With the tunneling and direct routing methods, port must be equal to
that of the service address. For normal services, the port specified  in the service address will be used if port is not specified. For fwmark services, port may be omit‐
ted, in which case  the destination port on the real server will be the destination port of the request sent to the virtual service.

[packet-forwarding-method]

  -g, --gatewaying  Use gatewaying (direct routing). This is the default.

  -i, --ipip  Use ipip encapsulation (tunneling).

  -m, --masquerading  Use masquerading (network access translation, or NAT).

  Note:  Regardless of the packet-forwarding mechanism specified, real servers for addresses for which there are interfaces on the local node will be use the local forward‐
  ing method, then packets for the servers will be passed to upper layer on the local node. This cannot be specified by ipvsadm, rather it set by the kernel as real servers
  are added or modified.

-p, --persistent [timeout]
Specify that a virtual service is persistent. If this option is specified, multiple requests from a client are redirected to the same real server selected for  the  first
request.   Optionally, the timeout of persistent sessions may be specified given in seconds, otherwise the default of 300 seconds will be used. This option may be used in
conjunction with protocols such as SSL or FTP where it is important that clients consistently connect with the same real server.

Note: If a virtual service is to handle FTP connections then persistence must be set for the virtual service if Direct Routing or Tunnelling is  used  as  the  forwarding
mechanism. If Masquerading is used in conjunction with an FTP service than persistence is not necessary, but the ip_vs_ftp kernel module must be used.  This module may be
manually inserted into the kernel using insmod(8).
```

