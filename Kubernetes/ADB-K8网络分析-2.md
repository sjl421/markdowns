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


## 集群环境

```
[mysql]
KEEP_VIP=192.168.143.32

[master]
192.168.143.64 = m1.adb.g1.com
192.168.143.65 = m2.adb.g1.com

[segment]
192.168.143.60 = s1.adb.g1.com
192.168.143.61 = s2.adb.g1.com
192.168.143.62 = s3.adb.g1.com
192.168.143.63 = s4.adb.g1.com
192.168.143.68 = s5.adb.g1.com
192.168.143.69 = s6.adb.g1.com
192.168.143.70 = s7.adb.g1.com
192.168.143.71 = s8.adb.g1.com
192.168.143.72 = s9.adb.g1.com
192.168.143.73 = s10.adb.g1.com
192.168.143.74 = s11.adb.g1.com
192.168.143.75 = s12.adb.g1.com
192.168.143.76 = s13.adb.g1.com
192.168.143.77 = s14.adb.g1.com
192.168.143.78 = s15.adb.g1.com
192.168.143.79 = s16.adb.g1.com
192.168.143.202 = s17.adb.g1.com
192.168.143.203 = s18.adb.g1.com
```

## Service网络分析

**service: adbops-service**

```
metadata:
  creationTimestamp: 2017-11-01T12:22:25Z
  name: adbops-service
  namespace: default
  resourceVersion: "3073"
  selfLink: /api/v1/namespaces/default/services/adbops-service
  uid: 518a51b1-beff-11e7-a0a3-6c92bf127a73
spec:
  clusterIP: 10.100.92.62
  ports:
  - name: http
    nodePort: 31505
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: adbops
  sessionAffinity: None
  type: NodePort
status:
  loadBalancer: {}
```

**Chain PREROUTING **

```
Chain PREROUTING (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination
1    1859K  207M KUBE-SERVICES  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kubernetes service portals */
2     348K   64M DOCKER     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL
```

添加语句:

```
-A PREROUTING -m comment --comment "kubernetes service portals" -j KUBE-SERVICES
```

**Chain KUBE-SERVICES (2 references)**

```
num   pkts bytes target     prot opt in     out     source               destination
2        0     0 KUBE-SVC-AZHJEWGPJBDBI6EL  tcp  --  *      *       0.0.0.0/0            10.96.0.20           /* default/uim-service: cluster IP */ tcp dpt:80
3        0     0 KUBE-SVC-RURZADBCFXFVXU2V  tcp  --  *      *       0.0.0.0/0            10.104.247.168       /* default/adb-apiservice:http cluster IP */ tcp dpt:8080
6        0     0 KUBE-SVC-S6ICJGUJQRHVNB2G  tcp  --  *      *       0.0.0.0/0            10.109.61.173        /* default/adbdms-service:http cluster IP */ tcp dpt:8080
11       0     0 KUBE-SVC-LUUCXDEVIXJDP2EU  tcp  --  *      *       0.0.0.0/0            10.110.48.70         /* default/biggg1-vip:adb cluster IP */ tcp dpt:5432
15       0     0 KUBE-SVC-2FTH5DXWURNCCRFT  tcp  --  *      *       0.0.0.0/0            10.108.62.13         /* default/mysql-vip:mysql cluster IP */ tcp dpt:3306
---
16       0     0 KUBE-SVC-TXDRYUK3PIVY2MYX  tcp  --  *      *       0.0.0.0/0            10.100.92.62         /* default/adbops-service:http cluster IP */ tcp dpt:8080
---
17       0     0 KUBE-SVC-LBOBEJMUGRVNA7T3  tcp  --  *      *       0.0.0.0/0            10.102.158.120       /* default/mysql-healthz-service:http cluster IP */ tcp dpt:8080
18       0     0 KUBE-NODEPORTS  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* kubernetes service nodeports; NOTE: this must be the last rule in this chain */ ADDRTYPE match dst-type LOCAL
```

```
-A KUBE-SERVICES -d 10.100.92.62/32 -p tcp -m comment --comment "default/adbops-service:http cluster IP" -m tcp --dport 8080 -j KUBE-SVC-TXDRYUK3PIVY2MYX

-A KUBE-SERVICES -m comment --comment "kubernetes service nodeports; NOTE: this must be the last rule in this chain" -m addrtype --dst-type LOCAL -j KUBE-NODEPORTS
```

**KUBE-NODEPORTS**

```
-A KUBE-NODEPORTS -p tcp -m comment --comment "default/adbops-service:http" -m tcp --dport 31505 -j KUBE-SVC-TXDRYUK3PIVY2MY
```

**Chain KUBE-SVC-TXDRYUK3PIVY2MYX**

```
Chain KUBE-SVC-TXDRYUK3PIVY2MYX (2 references)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 KUBE-SEP-VGX56CCCCWZBBO2O  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/adbops-service:http */
```

对应的添加规则是:

```
-A KUBE-SVC-TXDRYUK3PIVY2MYX -m comment --comment "default/adbops-service:http" -j KUBE-SEP-VGX56CCCCWZBBO2O
```

**Chain KUBE-SEP-VGX56CCCCWZBBO2O**

```
Chain KUBE-SEP-VGX56CCCCWZBBO2O (1 references)
num   pkts bytes target     prot opt in     out     source               destination
1        0     0 KUBE-MARK-MASQ  all  --  *      *       10.244.10.2          0.0.0.0/0            /* default/adbops-service:http */
2        0     0 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/adbops-service:http */ tcp to:10.244.10.2:8080
```

对应添加规则是:

```
-A KUBE-SEP-VGX56CCCCWZBBO2O -s 10.244.10.2/32 -m comment --comment "default/adbops-service:http" -j KUBE-MARK-MASQ 
-A KUBE-SEP-VGX56CCCCWZBBO2O -p tcp -m comment --comment "default/adbops-service:http" -m tcp -j DNAT --to-destination 10.244.10.2:8080
```

```
[root@m1 sun]# kubectl get po -o wide
NAME                             READY     STATUS    RESTARTS   AGE       IP               NODE
adb-apiservice-977715860-ppscz   1/1       Running   0          2d        10.244.7.2       s6.adb.g1.com
adbdms-3663664677-khkkd          1/1       Running   0          2d        10.244.9.2       s8.adb.g1.com
adbops-131222870-fw4n4           1/1       Running   1          2d        10.244.10.2      s9.adb.g1.com
adbrm-2599076652-n03lm           1/1       Running   0          2d        10.244.8.2       s7.adb.g1.com
```

 **endpoints**

```
[root@m1 sun]# kubectl get endpoints
NAME                    ENDPOINTS                                 AGE
adb-apiservice          10.244.7.2:8080                           2d
adbdms-service          10.244.9.2:8080                           2d
adbops-service          10.244.10.2:8080                          2d
adbrm-service           10.244.8.2:8080                           2d
```

可以看到

验证:

外网访问: 任一NodeIp + nodePort

由于adb-ops服务使用了NodePort类型的Service, 所以在集群中任一Node的Ip+nodePort就可以访问到adb-ops服务, 并且经过验证也确实如此;