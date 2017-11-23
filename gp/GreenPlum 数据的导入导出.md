# GreenPlum 数据的导入导出

```
\copy callrecord  TO '/tmp/a.csv' with csv;
```

导出数据库中某张表的数据到sql文件中

```
pg_dump -h 192.168.143.231 -Ugpadmin -t callrecord db1 > dtmall.sql
```

从sql文件中导入到数据库中的table中, 需要先创建好数据库

```
psql -h 192.168.143.87 -Ugpadmin  -d db1 -f dtmall.sql
```

```
\copy tablename  from '/tmp/a.csv' with csv;
```

```
\copy dxt_ao_column from '/home/sun/data_1k/dxt.csv' with csv DELIMITER '|' HEADER;
```



负责，今天启动测试，进度日报知会邮件列表人员

 由于有coredump的开发工作，部分投入

过程中databridge的使用请   友情指导下

目的：

给出databridge、java编码的白盒导入测试，性能结果

给出大小集群（4计算、10计算）的导入差异

使用版本：

v1.1.0 pre-release分支版本（databridge使用现归档的v2.1.2版本）

databridge的bug只记录，ADB的bug需要定位复现（先提单跟踪），重点关注性能和实现方式是否合理

使用ftp标准数据（1类tpc数据，1类单行1k字节数据）

测试内容参考：

<http://confluence.dtdream.com/pages/viewpage.action?pageId=84989693>

过程考虑：

多线程并发、内存限制带来的影响

考虑8segment、32segment（4台计算最大租户）、80segment（10台计算最大租户）、160segment（10台计算修改模板后最大租户）的差异

考虑databridge的实现方式和白盒测试结果的差异分析

考虑gpfdist模型和copyin的性能差异





```
更新内容：  

昨日进展（整体进度： ）：

1. 正常进度，无阻塞事件；

2. 未有针对测试结果的问题需要验证处理；

进展详情：

1. 完成大集群 80 Segment 租户的 AO表 + Tpc数据的Java代码导入测试和databridge的ADB加载测试， 共24个测试例；

2. 完成大集群 80 Segment 租户的Heap表 + Tpc数据的Java代码导入测试和databridge的ADB加载测试， 共24个测试例；

3. 完成大集群下实时场景下Update方式的测试文件（Sql文件）的生成工作；

```



更新内容：  

昨日进展（整体进度： ）：

1. 正常进度，无阻塞事件；

2. 未有针对测试结果的问题需要验证处理；

进展详情：

1. 完成大集群 80 Segment 租户的 AO表 + Tpc数据的Java代码导入测试和databridge的ADB加载测试， 共24个测试例；

2. 完成大集群 80 Segment 租户的Heap表 + Tpc数据的Java代码导入测试和databridge的ADB加载测试， 共24个测试例；

3. 完成大集群下实时场景下Update方式的测试文件（Sql文件）的生成工作；

给部分测试结论：

#### 更新内容：  

#### 昨日进展（整体进度： ）：

#### 1. 正常进度，无阻塞事件；

#### 2. 未有针对测试结果的问题需要验证处理；

#### 进展详情：

#### 1. 完成大集群 80 Segment 租户的 AO表 + Tpc数据的Java代码导入测试和databridge的ADB加载测试， 共24个测试例；

#### 2. 完成大集群 80 Segment 租户的Heap表 + Tpc数据的Java代码导入测试和databridge的ADB加载测试， 共24个测试例；

#### 3. 完成大集群下实时场景下Update方式的测试文件（Sql文件）的生成工作；