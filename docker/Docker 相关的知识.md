# Docker 相关的知识

## 1. 什么是Docker

Docker是一个开源的应用容器引擎，让开发者可以打包他们的应用及依赖包到一个可移植的容器中，然后发布到任何流行的Linux机器中，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。

### Docker的设计哲学

Docker设想是交付运行环境如同海运，OS如同一个货轮，每一个在OS基础上的软件都如同一个集装箱，用户可以通过标准化手段自由组装运行环境，同时集装箱的内容可以由用户自定义，也可以由专业人员制造。这样，交付一个软件，就是一些列的标准话组件的集合的交付，如同可高积木，用户只需要选择合适的继母组合，并且在最顶端署上自己的名字。

### Docer架构

* Docker使用客户端-服务器的架构模式，使用远程API来管理和创建Docker容器。Docer容器通过Docker镜像来创建。
* Docker采用C/S架构，Docker daemon作为服务端接受来自用户的请求，并处理这些请求（创建，运行，分发容器）。
* Docker daemon 一般在宿主机后台运行，等待接受来自客户端的消息。Docker客户端侧则为用户提供了一些列的可执行命令，用户用这些命令实现跟Docker daemon交互。

### Docker 底层实现

* Docker是基于LXC的轻量级虚拟化的，docker相比于KVM之类最明显的特点就是启动快，资源占用小。
* 由于LXC轻量级的特点，其启动快，而且docker能够只加载每个container变化的部分，这样占用资源小，能够在单机环境中下与KVM之类的虚拟化方案相比能够更加快速和占用更少资源。

### Docker的局限

* docker是基于Linux 64bit的，无法在32bit的环境下使用；
* LXC是基于cgroup等linux kernel功能的，因此container的guest系统只能是linux base的；
* 隔离性相比KVM之类的虚拟化方案还是有些欠缺，所有container公用一部分的运行库。
* 网络管理相对简单，主要是基于namespace隔离；

### Docker的原理

Docker核心解决的问题是利用LXC来实现类似VM的功能，从而利用更加节省的硬件资源提供给用户更多的计算资源。同VM的方式不同，LXC并不是一套硬件虚拟化方法、无法归属到全虚拟化、部分虚拟化和半虚拟化中的任意一个，而是一个操作系统级虚拟化方法、理解起来可能并不像VM那样直观。

* 隔离性-每个用户实例之间相互隔离、互不影响。硬件虚拟化方法给出的方法是VM，LXC给出的方法是container，更细一点的是kernel namespace。

* 可配额/可度量-每个用户实例可以按需提供期计算资源，所使用的资源可以被计量。硬件虚拟化方法因为虚拟了CPU，memory可以方便实现，LXC主要是利用cgroup来控制资源。

  ### Control Groups（cgroups）

  cgroups实现了对资源的配额和度量。cgroups的使用非常简单，提供类似文件的接口，在/cgroup目录下新建一个文件夹即可创建一个group，在此文件夹中新建task文件，并将pid写入该文件，即可实现对该进程的资源控制。

  ### Linux容器（LXC）

  借助于namesapce的隔离机制和cgroup限额功能，LXC提供了一套统一的API工具来简历和管理container。

  LXC 旨在提供一个共享kernel的OS级虚拟化方法，在执行时不用重复加载kernel，且container的kernel与host共享，因此可以大大加快container的启动过程，并且显著减少内存消耗。基于LXC的虚拟化方法的IO和CPU性能几乎是接近baremetal的性能。

  ### AUFS

  Docker对container的使用基本是建立在LXC基础之上的，然而LXC存在的问题是难以移动 - 难以通过标准化的模板制作、重建、复制和移动 container。

  在以VM为基础的虚拟化手段中，有image和snapshot可以用于VM的复制、重建以及移动的功能。想要通过container来实现快速的大规模部署和更新, 这些功能不可或缺。

  Docker 正是利用AUFS来实现对container的快速更新 - 在docker0.7中引入了storage driver, 支持AUFS, VFS, device mapper, 也为[BTRFS](http://baike.baidu.com/item/BTRFS)以及ZFS引入提供了可能。 但除了AUFS都未经过dotcloud的线上使用，因此我们还是从AUFS的角度介绍。

  AUFS(AnogherUnionFS)是一宗UniionFS, 简单来说就是支持将不同根目录挂在店奥同一个虚拟文件系统下（unite serveral directories into a single virtual filesystem）的文件系统，更进一步地，AUFS支持为每一个成员目录（AKA branch）设定‘readonly‘，’readwrite’和‘whiteout-able’权限，同时AUFS里有一个类似分层的概念，对readonly权限的branch可以逻辑上进行修改（增量地，不影响readonly部分的）。

  #### 结论

  采用AUFS作为docker的container的文件系统，能够提供如下好处：

  1. 节省存储空间 - 多个container可以共享base image存储
  2. 快速部署 - 如果要部署多个container，base image可以避免多次拷贝
  3. 内存更省 - 因为多个container共享base image, 以及OS的disk缓存机制，多个container中的进程命中缓存内容的几率大大增加
  4. 升级更方便 - 相比于 copy-on-write 类型的FS，base-image也是可以挂载为可writeable的，可以通过更新base image而一次性更新其之上的container
  5. 允许在不更改base-image的同时修改其目录中的文件 - 所有写操作都发生在最上层的writeable层中，这样可以大大增加base image能共享的文件内容。

  ### GRSEC

  grsec 是linux kernel安全相关的patch，用于保护host防止非法入侵。



