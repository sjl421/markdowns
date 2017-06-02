# GreenPlum 基础知识

​	GREENPLUM的高可用性是通过master和segment的镜像来实现的这种镜像是基于服务器级别的镜像。因此，不仅可以提供存储级别的保护，还可以防止其它PC服务器硬件的损坏。

​        Segment级别的镜像是通过把primary segment对应的mirror segment放置到不同的物理主机实现的。正常情况下，只有primary segment的instance处于工作状态，所有primary segment上的变化通过文件块的复制技术拷贝到mirror segment。因此，存放mirror segment的主机上只有复制用的进程，而不存在mirror segment instance。一旦primary segment出现故障，mirror segment的复制进程停止，并启动instance，保证数据库的操作继续。

### Master

Master也就是所谓的计算节点，为了高可用性设计，一个master 会对应一个standby，并且一定位于不同的主机上面；

master镜像就是master的在线standby，因为master上面只保存目录表和系统日志，因此在standby上面有一个事务重复记录进程（gpsyncagent）连接master以同步，一旦master连不上即坏掉，该进程停止工作，并让standby从master上一个成功事务操作之后开始接管系统（PS:master出故障前，standby上面只有gpsyncagent一个进程运行）。

### Segment

Segment Host 就是所谓的计算节点

* 每一台机器上可以配置一到多个Segment，因此建议采用相同的机器配置。

Segment primary

Segment mirror

master上只保存全局目录表，而数据都保存在segment上面，对于每一个表在创建时都要制定distributed策略，将满足不同策略的rows分别保存在不同的segment上面。

```wiki
Master Host
1)        建立与客户端的会话连接和管理；
2)        SQL的解析并形成分布式的执行计划；
3)        将生成好的执行计划分发到每个Segment上执行；
4)        收集Segment的执行结果；
5)        不存储业务数据，只存储数据字典；
6)        可以一主一备，分布在两台机器上，为了提高性能，最好单独占用一台机器
Segment Host
1)        业务数据的存储和存取；
2)        执行由Master分发的SQL语句；
3)        对于Master来说，每个Segment都是对等的，负责对应数据的存储和计算；
4)        每一台机器上可以配置一到多个Segment，因此建议采用相同的机器配置。
Interconnect
1)        是GP数据库的网络层，在每个Segment中起到一个IPC作用；
2)        推荐使用千兆以太网交换机做Interconnect；
3)        支持UDP和TCP两种协议，推荐使用UDP协议，因为其高可靠性、高性能以及可扩展性；而TCP协议最高只能使用1000个Segment实例。
```

### Shared Nothing vs. Shared Disk

   GreenPlum数据库是Shared Nothing[架构](http://lib.csdn.net/base/architecture)，因为每个Segement拥有自己的CPU、内存、硬盘来管理部份数据库。相反，基于共享磁盘的Shared Disk(或Shared Everything)架构的分布数据库管理系统拥有多个数据库服务实例管理单个数据库实例。Shared Nothing与Shared Disk架构有不同的优缺点。

* 在磁盘共享系统中所有数据存储在本地数据库服务端，不需要通过网络发送数据到另一服务器执行连表查询；然而网络磁盘存储解决方案和软件磁盘共享限制数据与数据库服务器数量添加到数据库集群。昂贵服务器和网络附属存储软件需要增加容量和保持可接受的查询响应时间。
* Shared Disk架构中, 每个CPU都有自己的内存, 但是所有CPU共享一组硬盘, 这些硬盘以SAN或者NAS的形式组织在一起。
* SD架构的缺点

​       1. 连接CPU和硬盘驱动的连接会成为系统的瓶颈.

​	2. 因为各个CPU都有自己的内存, 所以没有一个地方可以放置锁表(lock table)或者缓存池(buffer pool). 为了		设置锁, 只能在一个CPU上设置一个公共的锁管理器或者使用复杂的分布式锁协议. 当CPU数量增多时, 上述两种两种方法的可扩展性都不是很好。

* Shared Nothing架构中, 每个CPU有自己的内存和硬盘. 数据按行被水平划分, 这样不同节点上存储的是不同行的数据. 每个节点只负责处理自己硬盘上的数据. 每个节点有自己的锁表和缓存池, 这样就避免了复杂的分布式锁机制，SN的可扩展性非常好。