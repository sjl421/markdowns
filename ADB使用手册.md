# ADB使用手册

## 概述
DTCube ADB产品是数梦工场自研的分析型数据库引擎，最小规模为2台控制节点+2台计算节点，可以根据客户的数据规模和性能需求扩充计算节点的数目，计算节点的扩容步长为2。在小规模低成本场景下，本产品具有明显的竞争优势。
DTCube ADB产品主要应用于大数据场景下的分析性数据库(OLAP)，业界流行的BI软件都可以直接使用它进行在线业务分析。DTCube ADB采用通用的MPP并行处理架构，增加节点就可以线性提高系统的存储容量和处理能力，在扩展节点时操作简单，在很短时间内就能完成数据的重新分布。支持行存储和列存储混合模式，提高分析性能；同时提供数据压缩技术，降低存储成本。该引擎提供了用户友好的管理界面，包括系统安装、系统及集群配置、运维监控及预警等多方面的一站式支持。产品架构具有高可用性和快速故障恢复能力，其底层存储系统的支撑技术保证了数据的持久化和冗余复制，确保整个大数据处理系统的高可用性。该产品支持多租户隔离，根据用户角色进行权限控制，支持细粒度访问控制、列级安全及数据加密及解密等安全措施。
ADB的优势：
- 支持标准的SQL以及SQL优化
- MPP架构，扩展能力强，且扩展业务不中断
- 支持行或列混合存储
- 支持高可用HA及数据库安全
- 支持的硬件机型多，成本可控，安装部署及运维操作易用
- 支持海量存储及毫秒级快速响应，以及高并发
- 内置函数丰富，支持多种语言及多种插件
- 数据加载及导出快速

## ADB总体架构

![](attachment/gp_user_arch1.PNG)

### Greenplum架构
Greenplum是一种基于postgresql（开源数据库）的分布式数据库。其采用shared nothing架构（MPP），主机，操作系统，内存，存储都是自我控制的，不存在共享。主要由master host，segment host，interconnect三大部分组成。

![](attachment/gp_user_arch2.PNG)

Master Host

1. 建立与客户端的会话连接和管理；
2. SQL的解析并形成分布式的执行计划；
3. 将生成好的执行计划分发到每个Segment上执行；
4. 收集Segment的执行结果；
5. 不存储业务数据，只存储数据字典；
6. 可以一主一备，分布在两台机器上，为了提高性能，最好单独占用一台机器。

Segment Host
1. 业务数据的存储和存取；
2. 执行由Master分发的SQL语句；
3. 对于Master来说，每个Segment都是对等的，负责对应数据的存储和计算；
4. 每一台机器上可以配置一到多个Segment，因此建议采用相同的机器配置。

Interconnect
1. 是GP数据库的网络层，在每个Segment中起到一个IPC作用；
2. 推荐使用千兆以太网交换机做Interconnect；
3. 支持UDP和TCP两种协议，推荐使用UDP协议，因为其高可靠性、高性能以及可扩展性；而TCP协议最高只能使用1000个Segment实例。

### ADB硬件架构
ADB就是采用的Greenplum架构，因此硬件最小规模为2台控制节点+2台计算节点，可以根据客户的数据规模和性能需求扩充计算节点的数目。为了满足高可靠性，计算节点的扩容步长为2。

### ADB软件架构
ADB软件包括：ambari部署平台、ops运维平台、script脚本、mysql存储以及Greenplum Core。
- ambari部署平台：用于ADB及其所有组件的部署，ambari server 安装在2台控制节点的master一台主机上，ambari agent 在每台主机上都有安装。
- ops运维平台：只在2台控制节点的master一台主机上安装运行。
- script脚本：用于管理控制GP以及扩展GP的功能。安装时在2台控制节点主机上都有脚本，但是只需要在master一台主机上运行脚本。
- mysql存储：用于ambari部署平台和ops运维平台的数据存储，放在2台控制节点主机上，做了mysql HA同步数据。
- Greenplum Core：按照标准Greenplum的要求安装，2台控制节点分别作为master和standby主机一主一备，2台计算节点作为segment主机用于存储和计算。每台segment主机上的segment数需要配置为偶数个，为了满足高可靠性和可扩展性的同时主机负载均衡。关于segment主机及segment数的扩展请参考ADB扩容章节。



## ADB安装部署
参考《DTCube ADB分析型数据库部署手册》

## ADB运维及使用
### ADB运维平台
参考《DTCube ADB分析型数据库运维平台用户手册》

### 客户连接方式
- psql命令行连接
示例使用gpadmin用户登录指定ip的tpc库
```
psql -h 192.168.103.31 -p 5432 -U gpadmin -d tpc
```

- java通过jdbc连接
需要依赖postgresql的jdbc。maven版本如下
```
			<groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>9.4.1209.jre7</version>
```
```
public class TestJDBC {

    public static void main(String[] args) throws ClassNotFoundException, SQLException {
        Class.forName("org.postgresql.Driver");
        Connection conn = DriverManager.getConnection("jdbc:postgresql://192.168.103.31:5432/tpc","gpadmin","123456");
        Statement st = conn.createStatement();
        ResultSet rs = st.executeQuery("select * from item limit 1");
        while (rs.next()) {
            System.out.println(rs.getString(1));
        }
        rs.close();
        st.close();
    }
}
```


### 数据导入导出
参考[数据导出](doc/modules/GP数据导出.md)

### ADB扩容
ADB扩容有两种，一种是在集群中的原有segment主机上添加segment计算节点数，另一种是新增segment主机。
扩容的步骤有三步：
1. 将主机加入集群（如果在原有segment主机上扩展节点数，则不需要这一步）
2. 初始化segment节点并加入集群
3. 重分布表






