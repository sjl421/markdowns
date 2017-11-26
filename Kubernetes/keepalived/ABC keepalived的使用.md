# ABC keepalived的使用

实现场景:

1. 集群中运行两个keepalived pod,

   ```
   keepalived-master-cvhzs          1/1       Running            2          3d        192.168.143.65   m2.adb.g1.com
   keepalived-slave-512mt           1/1       Running            0          3d        192.168.143.60   s1.adb.g1.com
   ```

2. master

   ```
   [root@m1 k8s]# kubectl get po  keepalived-master-cvhzs -o yaml 
   apiVersion: v1
   kind: Pod
   metadata:
     annotations:
       kubernetes.io/created-by: |
         {"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicationController","namespace":"default","name":"keepalived-master","uid":"22695f6f-c5ef-11e7-919d-6c92bf127a73","apiVersion":"v1","resourceVersion":"1077"}}
     creationTimestamp: 2017-11-10T08:14:12Z
     generateName: keepalived-master-
     labels:
       unit: keepalived-master
     name: keepalived-master-cvhzs
     namespace: default
     ownerReferences:
     - apiVersion: v1
       controller: true
       kind: ReplicationController
       name: keepalived-master
       uid: 22695f6f-c5ef-11e7-919d-6c92bf127a73
     resourceVersion: "1269"
     selfLink: /api/v1/namespaces/default/pods/keepalived-master-cvhzs
     uid: 226a1d56-c5ef-11e7-919d-6c92bf127a73
   spec:
     containers:
     - env:
       - name: VIP
         valueFrom:
           configMapKeyRef:
             key: mysql.vip
             name: mysql-configmap
       - name: VRID
         valueFrom:
           configMapKeyRef:
             key: keepalived.vrid
             name: mysql-configmap
       - name: INTERFACE
         valueFrom:
           configMapKeyRef:
             key: interface
             name: mysql-configmap
       - name: PRIORITY
         value: "100"
       - name: REAL_IP
         valueFrom:
           configMapKeyRef:
             key: master
             name: mysql-configmap
       image: docker.dtdream.com/dtdream/keepalived:v1.2.13
       imagePullPolicy: IfNotPresent
       name: keepalived-master
       resources: {}
       securityContext:
         privileged: true
       terminationMessagePath: /dev/termination-log
       volumeMounts:
       - mountPath: /lib/modules
         name: modules
         readOnly: true
       - mountPath: /dev
         name: dev
       - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
         name: default-token-sjrj5
         readOnly: true
     dnsPolicy: ClusterFirst
     hostNetwork: true
     nodeName: m2.adb.g1.com
     nodeSelector:
       mysql: master
     restartPolicy: Always
     securityContext: {}
     serviceAccount: default
     serviceAccountName: default
     terminationGracePeriodSeconds: 30
     volumes:
     - hostPath:
         path: /lib/modules
       name: modules
     - hostPath:
         path: /dev
       name: dev
     - name: default-token-sjrj5
       secret:
         defaultMode: 420
         secretName: default-token-sjrj5
   ```

   slave: keepalived-slave-512mt

   ```
   [root@m1 k8s]# kubectl get po  keepalived-slave-512mt  -o yaml 
   apiVersion: v1
   kind: Pod
   metadata:
     annotations:
       kubernetes.io/created-by: |
         {"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicationController","namespace":"default","name":"keepalived-slave","uid":"2270c7ef-c5ef-11e7-919d-6c92bf127a73","apiVersion":"v1","resourceVersion":"1085"}}
     creationTimestamp: 2017-11-10T08:14:12Z
     generateName: keepalived-slave-
     labels:
       unit: keepalived-slave
     name: keepalived-slave-512mt
     namespace: default
     ownerReferences:
     - apiVersion: v1
       controller: true
       kind: ReplicationController
       name: keepalived-slave
       uid: 2270c7ef-c5ef-11e7-919d-6c92bf127a73
     resourceVersion: "1400"
     selfLink: /api/v1/namespaces/default/pods/keepalived-slave-512mt
     uid: 22715766-c5ef-11e7-919d-6c92bf127a73
   spec:
     containers:
     - env:
       - name: VIP
         valueFrom:
           configMapKeyRef:
             key: mysql.vip
             name: mysql-configmap
       - name: VRID
         valueFrom:
           configMapKeyRef:
             key: keepalived.vrid
             name: mysql-configmap
       - name: INTERFACE
         valueFrom:
           configMapKeyRef:
             key: interface
             name: mysql-configmap
       - name: PRIORITY
         value: "90"						<===
       - name: REAL_IP
         valueFrom:
           configMapKeyRef:
             key: slave
             name: mysql-configmap
       image: docker.dtdream.com/dtdream/keepalived:v1.2.13
       imagePullPolicy: IfNotPresent
       name: keepalived-slave
       resources: {}
       securityContext:
         privileged: true
       terminationMessagePath: /dev/termination-log
       volumeMounts:
       - mountPath: /lib/modules
         name: modules
         readOnly: true
       - mountPath: /dev
         name: dev
       - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
         name: default-token-sjrj5
         readOnly: true
     dnsPolicy: ClusterFirst
     hostNetwork: true
     nodeName: s1.adb.g1.com
     nodeSelector:
       mysql: slave
     restartPolicy: Always
     securityContext: {}
     serviceAccount: default
     serviceAccountName: default
     terminationGracePeriodSeconds: 30
     volumes:
     - hostPath:
         path: /lib/modules
       name: modules
     - hostPath:
         path: /dev
       name: dev
     - name: default-token-sjrj5
       secret:
         defaultMode: 420
         secretName: default-token-sjrj5
   ```

   mysql-configmap:

   ```
   [root@m1 k8s]# kubectl get cm mysql-configmap -o yaml 
   apiVersion: v1
   data:
     interface: bond0
     keepalived.vrid: "32"
     master: 192.168.143.65
     mysql.vip: 192.168.143.32
     slave: 192.168.143.60
   kind: ConfigMap
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

问题:

1. 在租户中的vip:keepalived的功能只用到了下vip的功能,并没有用到选举的功能, 那么如果出问题了是如何切换的?
2. mysql中的keepalived应该是有选举切换功能的?
3. ABC是如何保证OPS,DMS服务平台容器的HA的?

### ABC 租户keepalived的使用

```
root@m1:/# cat /etc/keepalived/keepalived.conf 
global_defs {
  vrrp_version 3
  vrrp_iptables KUBE-KEEPALIVED-VIP

  lvs_timeouts tcp 7200 tcpfin 120 udp 300
  lvs_sync_daemon bond0 vips
}

vrrp_instance vips {
  state BACKUP
  interface bond0
  virtual_router_id 33
  priority 100
  nopreempt
  advert_int 1
  
  track_interface {
    bond0
  }

  virtual_ipaddress { 
    192.168.143.33
  }
}

# Service: default/t1-vip
virtual_server 192.168.143.33 5432 {
  delay_loop 5
  lvs_sched wlc
  lvs_method NAT
  persistence_timeout 7200
  protocol TCP

  real_server 10.244.0.6 5432 {
    weight 1
    # close TCP_CHECK for adb. Maybe TCP_CHECK should be open for others.
    #TCP_CHECK { 
    #  connect_port 5432
    #  connect_timeout 3
    #  retry 10
    #  delay_before_retry 3
    #}
  }
}
```

http://freeloda.blog.51cto.com/2033581/1280962

