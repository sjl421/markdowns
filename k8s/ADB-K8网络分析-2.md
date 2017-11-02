#ADB-K8网络分析-2

```
[root@m1 home]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.143.142:5432 wlc persistent 7200
  -> 10.244.0.23:5432             Masq    1      2          4 
```



通过`iptables -t nat -L` 命令查看可以得到其中的输出有:

这条规则说明了什么? 

```
Chain KUBE-SEP-FYEGPKK3GAGYCKOA (1 references)
target     prot opt source               destination         
KUBE-MARK-MASQ  all  --  10.244.0.23          anywhere             /* default/t1-vip:adb */
DNAT       tcp  --  anywhere             anywhere             /* default/t1-vip:adb */ tcp to:10.244.0.23:5432
```



Adb 网络问题汇总:

1. ipvsadm -Ln 可以看到租户vip 到一个地址的映射, 映射后地址是什么地址?

   感觉像是一个随便的地址,然后又在iptables里面做了对应的规则;

2. iptables里的规则是怎么设定的?

3. vip pod(里面是 keepalived), 作用是什么? 

   在用户访问虚ip时是vip pod  起的的是什么作用?

4. 看到vip pod的在启动的时候传了两个参数进去

   --services-configmap=default/tenant3-vip-cm 

   --vrid=144

   这两个参数是做什么用的? (猜测是keepalived需要用的)

   keepalived 的工作原理什么样的? 

5. keepalived 镜像的 dockerfile是什么样的?

6. 物理机上的iptables规则是谁配置的? 

