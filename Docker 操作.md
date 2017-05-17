# Docker 操作

## 初始化

安装docker

```
blah, blah, blah...

```

安装数梦仓库 [http://confluence.dtdream.com/pages/viewpage.action?pageId=42371412](http://confluence.dtdream.com/pages/viewpage.action?pageId=42371412)

```
./docker_registry_dthink.sh
```

下载镜像，回头换成centos7的

```
docker search docker.registry.dthink.io/
docker pull docker.registry.dthink.io/library/centos6.7-jdk1.7.0_80
docker pull docker.registry.dthink.io/centos:7.2.1511

```

### 测试资源限制

#### 占用的cpu核

```
--cpuset-cpus                   CPUs in which to allow execution (0-3, 0,1)
--cpuset-mems                   MEMs in which to allow execution (0-3, 0,1)

```

不同类型的cpu，numa的数量不一定都是0，譬如

```
[root@localhost system]# lscpu 
...
NUMA node0 CPU(s):     0-7,16-23
NUMA node1 CPU(s):     8-15,24-31

```

```
docker run --cpuset-cpus=5 --cpuset-mems=0 -it  docker.registry.dthink.io/library/centos6.7-jdk1.7.0_80 /bin/bash

```

#### 内存

限制1G内存

```
docker run --cpuset-cpus=5 --cpuset-mems=0 -m 1G -it  docker.registry.dthink.io/library/centos6.7-jdk1.7.0_80 /bin/bash
```