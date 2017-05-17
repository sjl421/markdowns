# Docker 常用命令

## 1. 查看docker信息（version、info）

查看docker版本

```shell
1. $docker version  
2.   
3. # 显示docker系统的信息  
4. $docker info  
```

## 2. 对image的操作（search、pull、images、rmi、history）

```shell
1. # 检索image  
2. $docker search image_name  
3.   
4. # 下载image  
5. $docker pull image_name  
6.   
7. # 列出镜像列表; -a, --all=false Show all images; --no-trunc=false Don't truncate output; -q, --quiet=false Only show numeric IDs  
8. $docker images  
9.   
10. # 删除一个或者多个镜像; -f, --force=false Force; --no-prune=false Do not delete untagged parents  
11. $docker rmi image_name  
12.   
13. # 显示一个镜像的历史; --no-trunc=false Don't truncate output; -q, --quiet=false Only show numeric IDs  
14. $docker history image_name  
```

## 3. 启动容器（run）

[Docker](http://lib.csdn.net/base/docker)容器可以理解为在沙盒中运行的进程。这个沙盒包含了该进程运行所必须的资源，包括文件系统、系统类库、shell 环境等等。但这个沙盒默认是不会运行任何程序的。你需要在沙盒中运行一个进程来启动某一个容器。这个进程是该容器的唯一进程，所以当该进程结束的时候，容器也会完全的停止。

```shell
1. # 在容器中运行"echo"命令，输出"hello word"  
2. $docker run image_name echo "hello word"  
3.   
4. # 交互式进入容器中  
5. $docker run -i -t image_name /bin/bash  
6.   
7.   
8. # 在容器中安装新的程序  
9. $docker run image_name apt-get install -y app_name  
```

Note：  在执行apt-get 命令的时候，要带上-y参数。如果不指定-y参数的话，apt-get命令会进入交互模式，需要用户输入命令来进行确认，但在docker环境中是无法响应这种交互的。apt-get 命令执行完毕之后，容器就会停止，但对容器的改动不会丢失。

如果在运行的时候需要指定使用的资源，在`docker run`的命令中使用下面参数：

对docker使用资源的限制：

```shell
--cpu-period int              Limit CPU CFS (Completely Fair Scheduler) period
--cpu-quota int               Limit CPU CFS (Completely Fair Scheduler) quota
-c, --cpu-shares int          CPU shares (relative weight)
--cpuset-cpus string          CPUs in which to allow execution (0-3, 0,1)
```

参数cpu-sets，指定容器使用的核心。使用上述测试容器测试，指定容器使用0，3核

```shell
--cpuset-cpus                   CPUs in which to allow execution (0-3, 0,1) 限定使用的cpu
--cpuset-mems                   MEMs in which to allow execution (0-3, 0,1) 限定使用的mem
```

可以通过lscpu命令来显示当前系统中能够使用的cpu：

```shell
[root@localhost system]# lscpu 
...
NUMA node0 CPU(s):     0-7,16-23
NUMA node1 CPU(s):     8-15,24-31
```

然后就可以通过如下的命令来启动docker了:

```shell
docker run --cpuset-cpus=5 --cpuset-mems=0 -it  docker.registry.dthink.io/library/centos6.7-jdk1.7.0_80 /bin/bash
```

限定内存的使用：

```shell
docker run --cpuset-cpus=5 --cpuset-mems=0 -m 1G -it  docker.registry.dthink.io/library/centos6.7-jdk1.7.0_80 /bin/bash
```

详细的关于docker使用资源的限制可以参考下面的链接：

http://blog.csdn.net/horsefoot/article/details/51731543

## docker exec : 在运行的容器中执行命令

```
-d :分离模式: 在后台运行
-i :即使没有附加也保持STDIN 打开
-t :分配一个伪终端
```

## 4. 查看容器（ps）

```shell
1. # 列出当前所有正在运行的container  
2. $docker ps  
3. # 列出所有的container  
4. $docker ps -a  
5. # 列出最近一次启动的container  
6. $docker ps -l  
```

## 5. 保存对容器的修改（commit）

当你对某一个容器做了修改之后（通过在容器中运行某一个命令），可以把对容器的修改保存下来，这样下次可以从保存后的最新状态运行该容器。

```shell
1. # 保存对容器的修改; -a, --author="" Author; -m, --message="" Commit message  
2. $docker commit ID new_image_name  
```

Note：  image相当于类，[Container](http://lib.csdn.net/base/docker)相当于实例，不过可以动态给实例安装新软件，然后把这个container用commit命令固化成一个image。

## 6. 对容器的操作（rm、stop、start、kill、logs、diff、top、cp、restart、attach）

```shell
1. # 删除所有容器  
2. $docker rm docker ps -a -q  
3.   
4. # 删除单个容器; -f, --force=false; -l, --link=false Remove the specified link and not the underlying container; -v, --volumes=false Remove the volumes associated to the container  
5. $docker rm Name/ID  
6.   
7. # 停止、启动、杀死一个容器  
8. $docker stop Name/ID  
9. $docker start Name/ID  
10. $docker kill Name/ID  
11.   
12. # 从一个容器中取日志; -f, --follow=false Follow log output; -t, --timestamps=false Show timestamps  
13. $docker logs Name/ID  
14.   
15. # 列出一个容器里面被改变的文件或者目录，list列表会显示出三种事件，A 增加的，D 删除的，C 被改变的  
16. $docker diff Name/ID  
17.   
18. # 显示一个运行的容器里面的进程信息  
19. $docker top Name/ID  
20.   
21. # 从容器里面拷贝文件/目录到本地一个路径  
22. $docker cp Name:/container_path to_path  
23. $docker cp ID:/container_path to_path  
24.   
25. # 重启一个正在运行的容器; -t, --time=10 Number of seconds to try to stop for before killing the container, Default=10  
26. $docker restart Name/ID  
27.   
28. # 附加到一个运行的容器上面; --no-stdin=false Do not attach stdin; --sig-proxy=true Proxify all received signal to the process  
29. $docker attach ID  
```

Note： attach命令允许你查看或者影响一个运行的容器。你可以在同一时间attach同一个容器。你也可以从一个容器中脱离出来，是从CTRL-C。

## 7. 保存和加载镜像（save、load）

当需要把一台机器上的镜像迁移到另一台机器的时候，需要保存镜像与加载镜像。

```shell
1. # 保存镜像到一个tar包; -o, --output="" Write to an file  
2. $docker save image_name -o file_path  
3. # 加载一个tar包格式的镜像; -i, --input="" Read from a tar archive file  
4. $docker load -i file_path  
5.   
6. # 机器a  
7. $docker save image_name > /home/save.tar  
8. # 使用scp将save.tar拷到机器b上，然后：  
9. $docker load < /home/save.tar  
```

## 8、 登录registry server（login）

```shell`
1. # 登陆registry server; -e, --email="" Email; -p, --password="" Password; -u, --username="" Username  
2. $docker login  
```

## 9. 发布image（push）

```shell
1. # 发布docker镜像  
2. $docker push new_image_name  
```

## 10.  根据Dockerfile 构建出一个容器

```shell
1. #build  
2.       --no-cache=false Do not use cache when building the image  
3.       -q, --quiet=false Suppress the verbose output generated by the containers  
4.       --rm=true Remove intermediate containers after a successful build  
5.       -t, --tag="" Repository name (and optionally a tag) to be applied to the resulting image in case of success  
6. $docker build -t image_name Dockerfile_path  
```