# Pod调度

目的:

​	通过nodeSelector实现pod的定向调度

**1.查看当前apiservice pod所在的节点**

当前apiservice 的pod 是在m1.adb.g1.com上的

```
[root@m1 ADBStart]# kubectl get po -o wide
NAME                             READY     STATUS    RESTARTS   AGE       IP                NODE
adb-apiservice-977715860-0nhkl   1/1       Running   0          9m        10.244.0.8        m1.adb.g1.com
```

**2. 添加Node 的label**

```
#在Node m2.adb.g1.com 上添加role=standby的label
kubectl label node m2.adb.g1.com role=standby
```

**3. 修改Pod的yaml文件:**

在配置文件中添加nodeSelector, 将其label设置为`role=stadnby` 

```
kind: Service
apiVersion: v1
metadata:
  name: adb-apiservice
spec:
  ports:
    - name: http
      protocol: TCP
      port: 8080
      targetPort: 8080
      nodePort: 31507
  type: NodePort
  selector:
    app: adb-apiservice
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: adb-apiservice
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: adb-apiservice
    spec:
      volumes:
        - name: logs
          hostPath:
            path: /opt/tomcat-logs/apiservice
        - name: keystore-volume
          secret:
            secretName: apiservice-keystore
      nodeSelector:
        role: standby
      containers:
        - name: adb-apiservice
          image: docker.dtdream.com/adb/adb-apiservice:v1.1.0
          imagePullPolicy: IfNotPresent
          resources:
            limits:
              cpu: 1
              memory: 2Gi
            requests:
              cpu: 1
              memory: 2Gi
          volumeMounts:
            - name: logs
              mountPath: /opt/tomcat/logs
            - name: keystore-volume
              mountPath: /mnt/keystore
              readOnly: true
          command: ["/bin/sh", "-c", "/entry/entry-point.sh"]
          env:
            - name: MEMORY_LIMIT
              value: "2048"
```

**4. 重新创建pod后**

```
[root@m1 ADBStart]# kubectl get po -o wide
NAME                              READY     STATUS    RESTARTS   AGE       IP                NODE
adb-apiservice-3048720302-2sjx5   1/1       Running   0          5s        10.244.1.9        m2.adb.g1.com
```

**5. 删除Node label**

```
#如果Node m2.adb.g1.com 中存在key为role的label则删除
kubectl label node m2.adb.g1.com role-
```

**6. 在不修改yaml文件的前提下再重新创建pod**

发现此时pod虽然创建成功但是pod的状态一直是`Pending`

```
[root@m1 ADBStart]# kubectl get po 
NAME                              READY     STATUS    RESTARTS   AGE
adb-apiservice-3048720302-mr5ww   0/1       Pending   0          5s
```

这就说明了如果在pod的resource文件中设置了`nodeSelector` ,如果此时集群中没有符合nodeSelector 的节点存在,即便有其他的node存在,该pod也无法被调度(**并不是无法被创建**);

**7.继续修改资源文件,将nodeSelector删除掉之后pod就可以在成功创建并调度了**



**错误总结:**

1. 本来想通过kubectl edit 在线编辑pod的resource 文件, 发现kubectl edit 命令值允许修改image (好像还有个什么的东西),但是不允许更新添加nodeSelector
2. 后继续尝试使用`kubectl patch` 命令在增加nodeSelector字段,发现命令可以执行成功但是无法生效;
3. 最后才通过将pod删除掉-> 修改yaml文件-> 重新创建pod 才成功;