# docker 镜像仓库

公司的docker 镜像的服务器:103.46



每次是在本地package完了之后会在本地生成一个最新的本地的镜像;

所以通过docker image命令每次看到的应该最新的镜像;

本地镜像仓库的和远程镜像仓库的概念就像git之间的关系;



我们的几个平台的版本都是0.1的, adb_control的版本是1.4



/docker 目录先会有build 脚本, 可以看一下脚本;

```
docker save -o adb-dms:v0.1.tar docker.dtdream.com/adb/adb-dms:v0.1
```

```
docker save -o adb-ops:v0.1.tar docker.dtdream.com/adb/adb-ops:v0.1
```

```
docker save -o adb-rm:v0.1.tar docker.dtdream.com/adb/adb-rm:v0.1
```

```
docker save -o adb-control:v1.4.tar docker.dtdream.com/adb/adb-control:v1.4
```

```
docker build -t docker.dtdream.com/adb/adb-control:v1.4 . 
```



```

```

