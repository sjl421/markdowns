建议先阅读[十分钟带你理解Kubernetes核心概念](http://dockone.io/article/932)

以RM为例，先来看一眼RM的服务编排文件：
```
kind: Service
apiVersion: v1
metadata:
  name: adbrm-service
spec:
  ports:
    - name: http
      protocol: TCP
      port: 8080
      targetPort: 8080
  clusterIP: None
  selector:
    app: adbrm
---
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
  name: adbrm
spec:
  serviceName: "adbrm-service"
  replicas: 1
  template:
    metadata:
      labels:
        app: adbrm
    spec:
      volumes:
        - name: logs
          hostPath:
            path: /opt/tomcat-logs/adbrm
        - name: root-ssh-volume
          secret:
            secretName: root-ssh-key
            defaultMode: 420
            items:
              - key: authorized_keys
                mode: 384
                path: authorized_keys
              - key: id_rsa
               path: authorized_keys
              - key: id_rsa
                mode: 384
                path: id_rsa
              - key: id_rsa.pub
                path: id_rsa.pub
              - key: known_hosts
                path: known_hosts
      containers:
        - name: adb-rm
          image: docker.dtdream.com/adb/adb-rm:v0.1
          volumeMounts:
            - name: logs
              mountPath: /opt/apache-tomcat-7.0.78/logs
            - name: root-ssh-volume
              mountPath: /opt/root-ssh
              readOnly: true
          command: ["/bin/sh", "-c", "/entry/entry-point.sh"]
          env:
            - name: KUBE_API_SERVER_HOST
              valueFrom:
                secretKeyRef:
                  name: k8s-api
                  key: apiserver.host
            - name: KUBE_API_SERVER_PORT
              valueFrom:
                secretKeyRef:
                  name: k8s-api
                  key: apiserver.port
            - name: KUBE_API_SERVER_TOKEN
              valueFrom:
                secretKeyRef:
                  name: k8s-api
                  key: apiserver.token

            - name: MYSQL_ADDR
              valueFrom:
                configMapKeyRef:
                  name: mysql-configmap
                  key: mysql.vip
            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: mysql-connect
                  key: user
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-connect
                  key: password
```
上述编排文件包含一个service，一个StatefulSet。

service是一个抽象概念，它决定了路由规则。pod容器是易失的，并且可以调度到集群中任一机器，sevice通过selector指定的标签（键值对）选择路由到哪一个pod，例如这里的是标签是app:adbrm。这样不管pod调度到了哪一个机器上，对于应用来说是透明的，通过service总是能够访问到容器里的服务。

StatefulSet是一种有状态的pod，pod死掉之后可能被调度到其他集群上启动，StatefulSet则会在该机器上重新启动一个容器，replicas代表副本数，k8s总是保证有这么多个副本，这里为1。command是容器起来要执行的命令/脚本，执行完就退出容器。env是传入容器的变量，name是环境变量名，value直接指定值，也可以从configmap或secret获取信息。configmap是明文的键值对，账户密码等需要保密的信息通常使用secret。


容器的入口是 /entry/entry-point.sh, 镜像制作的时候已经将war包解压好了，/entry/entry-point.sh做的主要事情是：
1. 将挂载进来的ssh信息拷贝到/root/.ssh从而实现ssh打通
2. 脚本根据k8s传进来的环境变量生成war包需要的配置文件，并放到对应位置
3. catalina.sh run启动tomcat，该脚本与startup.sh的不同之处在于日志直接输出到控制台且不退出，Ctrl+C退出，而start.sh启动完tomcat就退出了。因为脚本执行退出，容器就退出了，所以这里采用catalina.sh run而不是start.sh

按Ctrl+C catalina.sh run退出后容器就退出了，由于容器的规则设为replicas=1，k8s会再次帮你启动一个容器

当需要在k8s里替换war包测试，又不想制作新镜像时，可以先删除当前资源，通过修改服务编排文件来更改容器入口创建一个持久的容器，再把war包拷贝进来，做自己想做的事情。
​    

具体步骤是：
1. 删除资源
```
kubectl delete -f /opt/ADBStart/k8s/adb/rm.yaml
```
2. 修改容器入口，将command中的/entry/entry-point.sh改为(while [ true ]; do sleep 3600; done) 这样容器永远不会退出
```
command: ["/bin/sh", "-c", "(while [ true ]; do sleep 3600; done)"]
```
3. 创建资源
```
kubectl create -f /opt/ADBStart/k8s/adb/rm.yaml
```
4、进入容器，将以前的war包以及解压后的文件夹删除
```
kubectl exec adbrm-0 -ti bash

rm -rf /opt/tomcat/webapps/resourcemanage.war /opt/tomcat/webapps/resourcemanage
```
5. 查看容器运行在哪个宿主机上，这里RM的宿主机是s1.adb.g1.com
```
[root@m1 tmp]# kubectl get pods -o wide
NAME                      READY     STATUS    RESTARTS   AGE       IP               NODE
adbdms-0                  1/1       Running   0          2h        10.244.3.2       s2.adb.g1.com
adbops-0                  1/1       Running   0          1h        10.244.3.3       s2.adb.g1.com
adbrm-0                   1/1       Running   0          2h        10.244.2.4       s1.adb.g1.com
cas                       1/1       Running   0          4h        10.244.1.2       m2.adb.g1.com
keepalived-master-bpks8   1/1       Running   2          4h        192.168.143.83   m2.adb.g1.com
keepalived-slave-tktbd    1/1       Running   0          4h        192.168.143.60   s1.adb.g1.com
manage                    1/1       Running   0          4h        10.244.5.2       s4.adb.g1.com
mysql-master-0            1/1       Running   0          4h        192.168.143.83   m2.adb.g1.com
mysql-slave-0             1/1       Running   0          4h        192.168.143.60   s1.adb.g1.com
tenant1-master            1/1       Running   0          26m       10.244.0.3       m1.adb.g1.com
tenant1-standby           1/1       Running   0          26m       10.244.1.4       m2.adb.g1.com
tenant1-vip               1/1       Running   0          26m       192.168.143.82   m1.adb.g1.com
uim                       1/1       Running   0          4h        10.244.2.3       s1.adb.g1.com
```
5. 将war包拷贝至宿主机s1.adb.g1.com上
6. 在宿主机s1.adb.g1.com上查看RM容器ID，这里是c1d432def382
```
[root@s1 ~]# docker ps
CONTAINER ID        IMAGE                                                          COMMAND                  CREATED             STATUS              PORTS               NAMES
c1d432def382        docker.dtdream.com/adb/adb-rm:v0.1                             "/bin/sh -c /entry/en"   2 hours ago         Up 2 hours                              k8s_adb-rm.3fa57266_adbrm-0_default_d5f85c77-7358-11e7-8c6b-6c92bf20da2b_f8bce27b
```
7. 将war包从宿主机s1.adb.g1.com拷贝至容器里
```
[root@s1 ~]# docker cp resourcemanage.war c1d432def382:/opt/tomcat/webapps
```
8. 进入容器，解压war包，运行/entry/entry-point.sh, 就会把配置文件都生成好了。由于/entry/entry-point.sh不再是容器入口，随意Ctrl+c并不会导致容器退出
```
kubectl exec adbrm-0 -ti bash

# 解压war包
unzip -oq /opt/tomcat/webapps/resourcemanage.war -d /opt/tomcat/webapps/resourcemanage

# 生成配置文件，并移动到目标位置
# 一定要先解压war包，因为生成好的配置文件是mv xxx /opt/tomcat/webapps/resourcemanage/xxx
/entry/entry-point.sh
```

#### 重要：配置文件更改请联系木溪，需要修改配置文件生成的脚本

我们k8s上的网络方案基本采用了nat(Flannel)方式，可以阅读这篇文章[一篇文章带你了解Flannel](http://dockone.io/article/618)
