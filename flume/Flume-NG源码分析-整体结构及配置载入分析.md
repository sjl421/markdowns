# Flume-NG源码分析-整体结构及配置载入分析

>参考：
>
>http://www.jianshu.com/p/0187459831af

---



![img](http://upload-images.jianshu.io/upload_images/1049928-ff9cd3f6ac9d7de3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## flume的基本模块说明

* flume-ng-channels 

  里面主要包含了filechannel， jdbcchannel，kafakachannel，memorychannel通道的实现

* flume-ng-clients

  实现了几个log4j相关的几个Appender，是的log4j的日志输出可以直接发送给flume-agent；其中有一个LoadBalanccingLog4jAppender的实现，提供了多个flume-agent的load balance 和 ha功能，采用 flume作为日志收集的可以考虑将这Appender引入内部的log4j中。

* flume-ng-configuration

  这个主要就是flume配置信息相关的类，包括载入flume-config.properties配置文件并解析。其中包括了source的配置，sink的配置，channel的配置，在阅读远吗的时候推荐先梳理这部分关系再看其他部分的。

* flume-ng-core

  flume真个核心框架，包括了各个模块的接口以及逻辑关系的实现。其中instrumentation是flume内实现的的一套metric机制，metric的变化和维护，其核心也就是在MonitorCounterGroup中通过一个Map<key, AtuomicLong>来实现metric的计量。ng-core下几乎大部分代码仍然在集中在channel，，sink，，source几个子目录下，其他目录基本完成了一个util和辅助的功能。

* flume-ng-node

  实现启动flume的一些基本类，包括main函数的入口（Application.jaa)。在理解configuration之后，从application的main函数入手，可以较快的了解整个flume的代码。

  ​