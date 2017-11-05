# ADB-Kubernetes实践

## 多租户

1. 首先是通过ipvsadm做的从外网到k8s内网的转换

```
[root@m1 ~]# ipvsadm -Ln
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.143.144:5432 wlc persistent 7200
  -> 10.244.0.19:5432             Masq    1      0          0     
```

其中, `192.168.143.144:5432` 是多租户的Vip;

`10.244.0.19:5432` 是租户的内网的

###分析

```
vip的 pod 是一个keepalived pod
```



2. Service :tenant3-vip

```yaml
[root@m1 ~]# kubectl get svc  tenant3-vip
NAME          CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
tenant3-vip   10.102.183.176   <none>        5432/TCP   3d
[root@m1 ~]# kubectl get svc  tenant3-vip -o yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: 2017-10-27T09:12:09Z
  labels:
    app: tenant3-vip
  name: tenant3-vip
  namespace: default
  resourceVersion: "1644454"
  selfLink: /api/v1/namespaces/default/services/tenant3-vip
  uid: e95c82d5-baf6-11e7-9da9-6c92bf4ab1a8
spec:
  clusterIP: 10.102.183.176
  ports:
  - name: adb
    port: 5432
    protocol: TCP
    targetPort: 5432
  selector:
    app: adb
    role: master
    tenancy: tenant3
  sessionAffinity: None
  type: ClusterIP
```

3. Pod:tenant3-vip-c5ss6

```
[root@m1 ~]# kubectl get po  tenant3-vip-c5ss6 -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubernetes.io/created-by: |
      {"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicationController","namespace":"default","name":"tenant3-vip","uid":"e952da90-baf6-11e7-9da9-6c92bf4ab1a8","apiVersion":"v1","resourceVersion":"1644445"}}
    scheduler.alpha.kubernetes.io/tolerations: |
      [
            {
            "key": "dedicated",
            "operator": "Equal",
            "value": "master",
            "effect": "NoSchedule"
          }
      ]
  creationTimestamp: 2017-10-27T09:12:09Z
  generateName: tenant3-vip-
  labels:
    adb: tenant3-vip
  name: tenant3-vip-c5ss6
  namespace: default
  ownerReferences:
  - apiVersion: v1
    controller: true
    kind: ReplicationController
    name: tenant3-vip
    uid: e952da90-baf6-11e7-9da9-6c92bf4ab1a8
  resourceVersion: "1644486"
  selfLink: /api/v1/namespaces/default/pods/tenant3-vip-c5ss6
  uid: e953604e-baf6-11e7-9da9-6c92bf4ab1a8
spec:
  containers:
  - args:
    - --services-configmap=default/tenant3-vip-cm
    - --vrid=144
    env:
    - name: POD_NAME
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: metadata.name
    - name: POD_NAMESPACE
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: metadata.namespace
    image: docker.dtdream.com/dtdream/kube-keepalived-vip:0.9
    imagePullPolicy: IfNotPresent
    name: tenant3-vip
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
      name: default-token-pgs6m
      readOnly: true
  dnsPolicy: ClusterFirst
  hostNetwork: true
  nodeName: m1.adb.g1.com
  nodeSelector:
    tenancy-tenant3: master
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
  - name: default-token-pgs6m
    secret:
      defaultMode: 420
      secretName: default-token-pgs6m
```

1. ConfigMap: tenant3-vip-cm

```
[root@m1 ~]# kubectl get cm tenant3-vip-cm -o yaml
apiVersion: v1
data:
  192.168.143.144: default/tenant3-vip
kind: ConfigMap
metadata:
  creationTimestamp: 2017-10-27T09:12:09Z
  name: tenant3-vip-cm
  namespace: default
  resourceVersion: "1644444"
  selfLink: /api/v1/namespaces/default/configmaps/tenant3-vip-cm
  uid: e94f74f2-baf6-11e7-9da9-6c92bf4ab1a8
```

3. ReplicationController: tenant3-vip 

```yaml
[root@m1 ~]# kubectl get rc tenant3-vip -o yaml
apiVersion: v1
kind: ReplicationController
metadata:
  creationTimestamp: 2017-10-27T09:12:09Z
  generation: 1
  labels:
    adb: tenant3-vip
  name: tenant3-vip
  namespace: default
  resourceVersion: "1644487"
  selfLink: /api/v1/namespaces/default/replicationcontrollers/tenant3-vip
  uid: e952da90-baf6-11e7-9da9-6c92bf4ab1a8
spec:
  replicas: 1
  selector:
    adb: tenant3-vip
  template:
    metadata:
      annotations:
        scheduler.alpha.kubernetes.io/tolerations: |
          [
                {
                "key": "dedicated",
                "operator": "Equal",
                "value": "master",
                "effect": "NoSchedule"
              }
          ]
      creationTimestamp: null
      labels:
        adb: tenant3-vip
    spec:
      containers:
      - args:
        - --services-configmap=default/tenant3-vip-cm
        - --vrid=144
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
        image: docker.dtdream.com/dtdream/kube-keepalived-vip:0.9
        imagePullPolicy: IfNotPresent
        name: tenant3-vip
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
      dnsPolicy: ClusterFirst
      hostNetwork: true
      nodeSelector:
        tenancy-tenant3: master
      restartPolicy: Always
      securityContext: {}
      terminationGracePeriodSeconds: 30
      volumes:
      - hostPath:
          path: /lib/modules
        name: modules
      - hostPath:
          path: /dev
        name: dev
```

4. Service: tenant3-vip

```yaml
[root@m1 ~]# kubectl get svc tenant3-vip -o yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: 2017-10-27T09:12:09Z
  labels:
    app: tenant3-vip
  name: tenant3-vip
  namespace: default
  resourceVersion: "1644454"
  selfLink: /api/v1/namespaces/default/services/tenant3-vip
  uid: e95c82d5-baf6-11e7-9da9-6c92bf4ab1a8
spec:
  clusterIP: 10.102.183.176
  ports:
  - name: adb
    port: 5432
    protocol: TCP
    targetPort: 5432
  selector:
    app: adb
    role: master
    tenancy: tenant3
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}

```

5. Pod: tenant3-master-0

```yaml
[root@m1 ~]# kubectl get po tenant3-master-0 -o yaml
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubernetes.io/created-by: |
      {"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"StatefulSet","namespace":"default","name":"tenant3-master","uid":"e9225f23-baf6-11e7-9da9-6c92bf4ab1a8","apiVersion":"apps","resourceVersion":"1644425"}}
    pod.beta.kubernetes.io/hostname: tenant3-master-0
    pod.beta.kubernetes.io/subdomain: tenant3-master
    scheduler.alpha.kubernetes.io/tolerations: |
      [
            {
            "key": "dedicated",
            "operator": "Equal",
            "value": "master",
            "effect": "NoSchedule"
          }
      ]
  creationTimestamp: 2017-10-27T09:12:09Z
  generateName: tenant3-master-
  labels:
    app: adb
    role: master
    tenancy: tenant3
  name: tenant3-master-0
  namespace: default
  resourceVersion: "1644473"
  selfLink: /api/v1/namespaces/default/pods/tenant3-master-0
  uid: e9234827-baf6-11e7-9da9-6c92bf4ab1a8
spec:
  containers:
  - command:
    - /bin/sh
    - -c
    - /entry/entry-point.sh
    env:
    - name: MEMORY_LIMIT
      value: "2048"
    - name: MASTER
      value: tenant3-master
    - name: STANDBY
      value: tenant3-standby
    - name: RM_ADDR
      valueFrom:
        configMapKeyRef:
          key: rm.addr
          name: adbops-cm
    - name: RM_PORT
      valueFrom:
        configMapKeyRef:
          key: rm.port
          name: adbops-cm
    - name: ADB_ADMIN
      valueFrom:
        secretKeyRef:
          key: adb.admin
          name: adb-connect-secret
    - name: ADB_PASSWORD
      valueFrom:
        secretKeyRef:
          key: adb.password
          name: adb-connect-secret
    - name: MYSQL_ADDR
      valueFrom:
        configMapKeyRef:
          key: mysql.vip
          name: mysql-configmap
    - name: MYSQL_USER
      valueFrom:
        secretKeyRef:
          key: user
          name: mysql-connect
    - name: MYSQL_PASSWORD
      valueFrom:
        secretKeyRef:
          key: password
          name: mysql-connect
    image: docker.dtdream.com/adb/adb-control:v1.1.0
    imagePullPolicy: IfNotPresent
    name: tenant3-master
    resources:
      limits:
        memory: 16Gi
      requests:
        memory: 16Gi
    terminationMessagePath: /dev/termination-log
    volumeMounts:
    - mountPath: /home/gpadmin/data
      name: data
    - mountPath: /home/gpadmin/gpAdminLogs
      name: adminlog
    - mountPath: /opt/root-ssh
      name: root-ssh-volume
      readOnly: true
    - mountPath: /opt/gpadmin-ssh
      name: gpadmin-ssh-volume
      readOnly: true
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-pgs6m
      readOnly: true
  dnsPolicy: ClusterFirst
  hostname: tenant3-master
  nodeName: m1.adb.g1.com
  nodeSelector:
    tenancy-tenant3: master
  restartPolicy: Always
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  volumes:
  - name: gpadmin-ssh-volume
    secret:
      defaultMode: 420
      items:
      - key: authorized_keys
        mode: 384
        path: authorized_keys
      - key: id_rsa
        mode: 384
        path: id_rsa
      - key: id_rsa.pub
        path: id_rsa.pub
      - key: known_hosts
        path: known_hosts
      secretName: gpadmin-ssh-key
  - hostPath:
      path: /data0/tenant3/data
    name: data
  - hostPath:
      path: /data0/tenant3/adminlog
    name: adminlog
  - name: root-ssh-volume
    secret:
      defaultMode: 420
      items:
      - key: authorized_keys
        mode: 384
        path: authorized_keys
      - key: id_rsa
        mode: 384
        path: id_rsa
      - key: id_rsa.pub
        path: id_rsa.pub
      - key: known_hosts
        path: known_hosts
      secretName: root-ssh-key
  - name: default-token-pgs6m
    secret:
      defaultMode: 420
      secretName: default-token-pgs6m
```



---

在通过java代码在创建租户的过程为:

1. createNodeLables

```
kubectl label node %s %s=%s", hostname, key, role.toLowerCase())
```

2. createControl

createService: ( 在真正create之前先删除掉可能存在的同名service)

```
String serviceJson = "{\n" +
"  \"apiVersion\": \"v1\",\n" +
"  \"kind\": \"Service\",\n" +
"  \"metadata\": {\n" +
"    \"name\": \"" + service_name + "\",\n" +
"    \"labels\": {\n" +
"      \"app\": \"adb\"\n" +
"    }\n" +
"  },\n" +
"  \"spec\": {\n" +
"    \"ports\": [\n" +
"      {\n" +
"        \"name\": \"adb\",\n" +
"        \"port\": 5432,\n" +
"        \"targetPort\": 5432\n" +
"      },\n" +
"      {\n" +
"        \"name\": \"ssh\",\n" +
"        \"port\": 22,\n" +
"        \"targetPort\": 22\n" +
"      }\n" +
"    ],\n" +
"    \"clusterIP\": \"None\",\n" +
"    \"selector\": {\n" +
"      \"tenancy\": \"" + tenancy + "\",\n" +
"      \"role\": \"" + role.toLowerCase() + "\",\n" +
"      \"app\": \"adb\"\n" +
"    }\n" +
"  }\n" +
"}";
```

createStatefulSet:

```
String podJson = "{\n" +
"  \"apiVersion\": \"apps/v1beta1\",\n" +
"  \"kind\": \"StatefulSet\",\n" +
"  \"metadata\": {\n" +
"    \"name\": \"" + statefulsetName + "\"\n" +
"  },\n" +
"  \"spec\": {\n" +
"    \"serviceName\": \"" + statefulsetName + "\",\n" +
"    \"replicas\": 1,\n" +
"    \"template\": {\n" +
"      \"metadata\": {\n" +
"        \"labels\": {\n" +
"          \"tenancy\": \"" + tenancy + "\",\n" +
"          \"role\": \"" + role.toLowerCase() + "\",\n" +
"          \"app\": \"adb\"\n" +
"        },\n" +
"        \"annotations\": {\n" +
"          \"scheduler.alpha.kubernetes.io/tolerations\": \"[\\n      {\\n      \\\"key\\\": \\\"dedicated\\\",\\n      \\\"operator\\\": \\\"Equal\\\",\\n      \\\"value\\\": \\\"master\\\",\\n      \\\"effect\\\": \\\"NoSchedule\\\"\\n    }\\n]\\n\"\n" +
"        }\n" +
"      },\n" +
"      \"spec\": {\n" +
"        \"nodeSelector\": {\n" +
"          \"tenancy-"+ tenancy + "\": \"" + role.toLowerCase() + "\"\n" +
"        },\n" +
"        \"hostname\": \"" + statefulsetName +  "\",\n" +
"        \"containers\": [\n" +
"          {\n" +
"            \"resources\": {\n" +
"              \"limits\": {\n" +
"                \"memory\": \"" + memory + "Gi\"\n" +
"              }\n" +
"            },\n" +
"            \"image\": \"" + CONTAINER_IMAGE + "\",\n" +
"            \"imagePullPolicy\": \"IfNotPresent\",\n" +
"            \"name\": \"" + statefulsetName + "\",\n" +
"            \"volumeMounts\": [\n" +
"              {\n" +
"                \"name\": \"data\",\n" +
"                \"mountPath\": \"/home/gpadmin/data\"\n" +
"              },\n" +
"              {\n" +
"                \"name\": \"adminlog\",\n" +
"                \"mountPath\": \"/home/gpadmin/gpAdminLogs\"\n" +
"              },\n" +
"              {\n" +
"                \"name\": \"tomcatlogs\",\n" +
"                \"mountPath\": \"/opt/tomcat/logs\"\n" +
"              },\n" +
"              {\n" +
"                \"name\": \"root-ssh-volume\",\n" +
"                \"mountPath\": \"/opt/root-ssh\",\n" +
"                \"readOnly\": true\n" +
"              },\n" +
"              {\n" +
"                \"name\": \"gpadmin-ssh-volume\",\n" +
"                \"mountPath\": \"/opt/gpadmin-ssh\",\n" +
"                \"readOnly\": true\n" +
"              }\n" +
"            ],\n" +
"            \"env\": [\n" +
"              {\n" +
"                \"name\": \"MEMORY_LIMIT\",\n" +
"                \"value\": \"2048\"\n" +
"              },\n" +
"              {\n" +
"                \"name\": \"MASTER\",\n" +
"                \"value\": \"" + env_master + "\"\n" +
"              },\n" +
"              {\n" +
"                \"name\": \"STANDBY\",\n" +
"                \"value\": \"" + env_standby + "\"\n" +
"              },\n" +
"              {\n" +
"                \"name\": \"RM_ADDR\",\n" +
"                \"valueFrom\": {\n" +
"                  \"configMapKeyRef\": {\n" +
"                    \"name\": \"adbops-cm\",\n" +
"                    \"key\": \"rm.addr\"\n" +
"                  }\n" +
"                }\n" +
"              },\n" +
"              {\n" +
"                \"name\": \"RM_PORT\",\n" +
"                \"valueFrom\": {\n" +
"                  \"configMapKeyRef\": {\n" +
"                    \"name\": \"adbops-cm\",\n" +
"                    \"key\": \"rm.port\"\n" +
"                  }\n" +
"                }\n" +
"              },\n" +
"              {\n" +
"                \"name\": \"ADB_ADMIN\",\n" +
"                \"valueFrom\": {\n" +
"                  \"secretKeyRef\": {\n" +
"                    \"name\": \"adb-connect-secret\",\n" +
"                    \"key\": \"adb.admin\"\n" +
"                  }\n" +
"                }\n" +
"              },\n" +
"              {\n" +
"                \"name\": \"ADB_PASSWORD\",\n" +
"                \"valueFrom\": {\n" +
"                  \"secretKeyRef\": {\n" +
"                    \"name\": \"adb-connect-secret\",\n" +
"                    \"key\": \"adb.password\"\n" +
"                  }\n" +
"                }\n" +
"              },\n" +
"              {\n" +
"                \"name\": \"MYSQL_ADDR\",\n" +
"                \"valueFrom\": {\n" +
"                  \"configMapKeyRef\": {\n" +
"                    \"name\": \"mysql-configmap\",\n" +
"                    \"key\": \"mysql.vip\"\n" +
"                  }\n" +
"                }\n" +
"              },\n" +
"              {\n" +
"                \"name\": \"MYSQL_USER\",\n" +
"                \"valueFrom\": {\n" +
"                  \"secretKeyRef\": {\n" +
"                    \"name\": \"mysql-connect\",\n" +
"                    \"key\": \"user\"\n" +
"                  }\n" +
"                }\n" +
"              },\n" +
"              {\n" +
"                \"name\": \"MYSQL_PASSWORD\",\n" +
"                \"valueFrom\": {\n" +
"                  \"secretKeyRef\": {\n" +
"                    \"name\": \"mysql-connect\",\n" +
"                    \"key\": \"password\"\n" +
"                  }\n" +
"                }\n" +
"              }\n" +
"            ],\n" +
"            \"command\": [\n" +
"              \"/bin/sh\",\n" +
"              \"-c\",\n" +
"              \"/entry/entry-point.sh\"\n" +
"            ]\n" +
"          }\n" +
"        ],\n" +
"        \"volumes\": [\n" +
"          {\n" +
"            \"name\": \"data\",\n" +
"            \"hostPath\": {\n" +
"              \"path\": \"" + host_dataPath +"\"\n" +
"            }\n" +
"          },\n" +
"          {\n" +
"            \"name\": \"adminlog\",\n" +
"            \"hostPath\": {\n" +
"              \"path\": \"" + host_logPath +"\"\n" +
"            }\n" +
"          },\n" +
"          {\n" +
"            \"name\": \"tomcatlogs\",\n" +
"            \"hostPath\": {\n" +
"              \"path\": \"" + host_logPath +"/tomcatlogs\"\n" +
"            }\n" +
"          },\n" +
"          {\n" +
"            \"name\": \"root-ssh-volume\",\n" +
"            \"secret\": {\n" +
"              \"secretName\": \"root-ssh-key\",\n" +
"              \"defaultMode\": 420,\n" +
"              \"items\": [\n" +
"                {\n" +
"                  \"key\": \"authorized_keys\",\n" +
"                  \"mode\": 384,\n" +
"                  \"path\": \"authorized_keys\"\n" +
"                },\n" +
"                {\n" +
"                  \"key\": \"id_rsa\",\n" +
"                  \"mode\": 384,\n" +
"                  \"path\": \"id_rsa\"\n" +
"                },\n" +
"                {\n" +
"                  \"key\": \"id_rsa.pub\",\n" +
"                  \"path\": \"id_rsa.pub\"\n" +
"                },\n" +
"                {\n" +
"                  \"key\": \"known_hosts\",\n" +
"                  \"path\": \"known_hosts\"\n" +
"                }\n" +
"              ]\n" +
"            }\n" +
"          },\n" +
"          {\n" +
"            \"name\": \"gpadmin-ssh-volume\",\n" +
"            \"secret\": {\n" +
"              \"secretName\": \"gpadmin-ssh-key\",\n" +
"              \"defaultMode\": 420,\n" +
"              \"items\": [\n" +
"                {\n" +
"                  \"key\": \"authorized_keys\",\n" +
"                  \"mode\": 384,\n" +
"                  \"path\": \"authorized_keys\"\n" +
"                },\n" +
"                {\n" +
"                  \"key\": \"id_rsa\",\n" +
"                  \"mode\": 384,\n" +
"                  \"path\": \"id_rsa\"\n" +
"                },\n" +
"                {\n" +
"                  \"key\": \"id_rsa.pub\",\n" +
"                  \"path\": \"id_rsa.pub\"\n" +
"                },\n" +
"                {\n" +
"                  \"key\": \"known_hosts\",\n" +
"                  \"path\": \"known_hosts\"\n" +
"                }\n" +
"              ]\n" +
"            }\n" +
"          }\n" +
"        ],\n" +
"        \"restartPolicy\": \"Always\"\n" +
"      }\n" +
"    }\n" +
"  }\n" +
"}";
```

createVip:

createVipCm:

```
String cmJson ="{\n" +
"  \"apiVersion\": \"v1\",\n" +
"  \"kind\": \"ConfigMap\",\n" +
"  \"metadata\": {\n" +
"    \"name\": \"" + cmName + "\"\n" +
"  },\n" +
"  \"data\": {\n" +
"    \"" + vip + "\": \"default/" + podName + "\"\n" +
"  }\n" +
"}";

String uri = String.format("https://%s:%s/api/v1/namespaces/default/configmaps", this.addr, this.port);
RestHttpClient restHttpClient = new RestHttpClient(uri, this.token);
restHttpClient.httpPost(configMap);
```

createReplicationController:

```
String controlloerJson = "{\n" +
"  \"apiVersion\": \"v1\",\n" +
"  \"kind\": \"ReplicationController\",\n" +
"  \"metadata\": {\n" +
"    \"name\": \"" + controllerName + "\"\n" +
"  },\n" +
"  \"spec\": {\n" +
"    \"replicas\": 1,\n" +
"    \"template\": {\n" +
"      \"metadata\": {\n" +
"        \"labels\": {\n" +
"          \"adb\": \"" + controllerName + "\"\n" +
"        },\n" +
"        \"annotations\": {\n" +
"          \"scheduler.alpha.kubernetes.io/tolerations\": \"[\\n      {\\n      \\\"key\\\": \\\"dedicated\\\",\\n      \\\"operator\\\": \\\"Equal\\\",\\n      \\\"value\\\": \\\"master\\\",\\n      \\\"effect\\\": \\\"NoSchedule\\\"\\n    }\\n]\\n\"\n" +
"        }\n" +
"      },\n" +
"      \"spec\": {\n" +
"        \"nodeSelector\": {\n" +
"          \"tenancy-" + tenancy + "\": \"" + curMasterOriginalRole.toLowerCase() + "\"\n" +
"        },\n" +
"        \"hostNetwork\": true,\n" +
"        \"containers\": [\n" +
"          {\n" +
"            \"image\": \"docker.dtdream.com/dtdream/kube-keepalived-vip:0.9\",\n" +
"            \"name\": \"" + controllerName + "\",\n" +
"            \"imagePullPolicy\": \"IfNotPresent\",\n" +
"            \"securityContext\": {\n" +
"              \"privileged\": true\n" +
"            },\n" +
"            \"volumeMounts\": [\n" +
"              {\n" +
"                \"mountPath\": \"/lib/modules\",\n" +
"                \"name\": \"modules\",\n" +
"                \"readOnly\": true\n" +
"              },\n" +
"              {\n" +
"                \"mountPath\": \"/dev\",\n" +
"                \"name\": \"dev\"\n" +
"              }\n" +
"            ],\n" +
"            \"env\": [\n" +
"              {\n" +
"                \"name\": \"POD_NAME\",\n" +
"                \"valueFrom\": {\n" +
"                  \"fieldRef\": {\n" +
"                    \"fieldPath\": \"metadata.name\"\n" +
"                  }\n" +
"                }\n" +
"              },\n" +
"              {\n" +
"                \"name\": \"POD_NAMESPACE\",\n" +
"                \"valueFrom\": {\n" +
"                  \"fieldRef\": {\n" +
"                    \"fieldPath\": \"metadata.namespace\"\n" +
"                  }\n" +
"                }\n" +
"              }\n" +
"            ],\n" +
"            \"args\": [\n" +
"              \"--services-configmap=default/" + cmName + "\",\n" +
"              \"--vrid=" + vrid + "\"\n" +
"            ]\n" +
"          }\n" +
"        ],\n" +
"        \"volumes\": [\n" +
"          {\n" +
"            \"name\": \"modules\",\n" +
"            \"hostPath\": {\n" +
"              \"path\": \"/lib/modules\"\n" +
"            }\n" +
"          },\n" +
"          {\n" +
"            \"name\": \"dev\",\n" +
"            \"hostPath\": {\n" +
"              \"path\": \"/dev\"\n" +
"            }\n" +
"          }\n" +
"        ]\n" +
"      }\n" +
"    }\n" +
"  }\n" +
"}";

String uri = String.format("https://%s:%s/api/v1/namespaces/default/replicationcontrollers", this.addr, this.port);
RestHttpClient restHttpClient = new RestHttpClient(uri, this.token);
restHttpClient.httpPost(controllerJson);
```

createVipService:

```
String serviceJson = "{\n" +
"  \"apiVersion\": \"v1\",\n" +
"  \"kind\": \"Service\",\n" +
"  \"metadata\": {\n" +
"    \"name\": \"" + tenancy + "-vip\",\n" +
"    \"labels\": {\n" +
"      \"app\": \"" + tenancy + "-vip\"\n" +
"    }\n" +
"  },\n" +
"  \"spec\": {\n" +
"    \"ports\": [\n" +
"      {\n" +
"        \"name\": \"adb\",\n" +
"        \"port\": 5432,\n" +
"        \"targetPort\": 5432\n" +
"      }\n" +
"    ],\n" +
"    \"selector\": {\n" +
"      \"tenancy\": \"" + tenancy + "\",\n" +
"      \"role\": \"" + role.toLowerCase() + "\",\n" +
"      \"app\": \"adb\"\n" +
"    }\n" +
"  }\n" +
"}";

String uri = String.format("https://%s:%s/api/v1/namespaces/default/services", this.addr, this.port);
RestHttpClient restHttpClient = new RestHttpClient(uri, this.token);
restHttpClient.httpPost(serviceJson.toString());
```



---

Pearl:

1. statefulset 控制起pod;

2. service: 控制ip访问;

3. vip:  service 选master的pod;

   由于ADB是主备形式, 对外的服务是通过vip的方式来控制访问的;

4. keepalived

   1. 下Vip( 就是通过ipvsadm来将访问的ip地址转换到service的vip)
   2. 选master(目前并没有这个功能)

实际的访问路径是:

```
143.144 -> keepalived -> cm -> service ->
第一步的映射是使用 ipvsadm 来实现的;
```



