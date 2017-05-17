 

# Docker 从安装到部署一个web应用(go、java)方式

说明: 
1.权限是root，不是则先提升权限

------

# 一:安装docker

1.[https://docs.docker.com/engine/installation/binaries/](https://docs.docker.com/engine/installation/binaries/) 
下载[Docker](http://lib.csdn.net/base/docker)最新版二进制tar.gz

```
linux下:
wget https://get.docker.com/builds/Darwin/x86_64/docker-1.11.0.tgz1212
```

2.丢到 $path中

```
mv docker /usr/local/sbin11
```

3.启动

```
docker daemon &11
```

------

# 二.在容器上运行tomcat

docker官方镜像仓库由于有墙，所以下载的很慢。目前我用的是时速云的镜像。

第一步:拉取镜像到本地 
`docker pull index.tenxcloud.com/tenxcloud/tomcat`

第二步:为镜像添加一个别名 
`docker tag index.tenxcloud.com/tenxcloud/tomcat tomcat-1`

第二步:启动tomcat 
`docker run -p 5000:8080 --name container1 tomcat-1` 
如此一来，tomcat就启动了，-p 5000:8080的意思是把容器tomcat的8080端口隐射到宿主机的端口上，这样外网访问5000就能访问到我们的container1的8080 tomcat上面了.

如此一来，一个简单的tomcat就跑起来了.

此处容器container1 和 镜像tomcat-1，我的理解是镜像就是一个模板，container1就是根据这个模板创造的一个真正的盆子，这个盆子里面就跑着我们的tomcat. 所以我们可以用同一个镜像创建许多[Container](http://lib.csdn.net/base/docker)。

# 三.在tomcat上面部署我们的应用

接下来我们要部署我们的应用上去，思路是进入到container1里面去，此时可以把container1想象为一个新的机器，我们只需要到tomcat的webapp丢war，然后重启就行了.

### 1.进入容器内部

`docker exec -it container2 /bin/bash`

### 2.查看tomcat webapp路径

/tomcat/webapps

## 3.传war

把war丢到宿主机 在丢到container里面丢到tomcat/webapps

`docker cp DemoOne.war container2:tomcat/webapps`

太TM惊喜了，docker本身就支持啊！！！666666.

## 4.重启容器

不需要了。。。docker自动帮你部署了 
![这里写图片描述](http://img.blog.csdn.net/20160513214503969)

## 5.访问应用

![这里写图片描述](http://img.blog.csdn.net/20160513214616516)

------

至此，一个完整的docker部署tomcat及上线一个[Java ](http://lib.csdn.net/base/java)web应用流程就走通了. 
说实话，走通后才发现是这么的简单。之前概念上面不懂的地方这下也基本通了。 
不得不说很Nice，和预想中的完全一样，就把dokcer给你创建的container当成一个新的[Linux](http://lib.csdn.net/base/linux)用就行啦！

------

# 使用docker部署一套应用系统

接下来部署一套完整的系统，包括如下组件： 
负载均衡：Haproxy 
[Java](http://lib.csdn.net/base/javase)工：tomcat 
缓存：[Redis](http://lib.csdn.net/base/redis) Master、Slave

流程是Java开一个restful接口，为redis写入一个数据， 
再开一个restful接口，从redis读取一个数据。

系统结构如图： 
![这里写图片描述](http://img.blog.csdn.net/20160516100043406)

步骤： 
1.准备java工程，并打包成war 
2.拉取haproxy镜像，并运行

```
//注意 --name不能放在最后，6555:80 80不可更改，是haproxy本身的端口
docker run -d -p 6555:80 --link container2:container2 --name haproxy-1 haproxy1212
```

![这里写图片描述](http://img.blog.csdn.net/20160516142942872) 
可以看到，haproxy已经成功实现了代理的功能. 
目前的镜像不知道为什么不能通过修改haproxy.cfg的方式来支持，后续研究

------

之后再补上golang镜像及应用部署的流程