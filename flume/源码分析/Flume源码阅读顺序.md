#Flume源码阅读顺序

### 宏观整体

Flume-NG源码分析-整体结构及配置载入分析

Flume(NG)架构设计要点及配置实践（直接看原链接）



### 简单的概念

- Event：一个数据单元，带有一个可选的消息头
- Flow：Event从源点到达目的点的迁移的抽象
- Client：操作位于源点处的Event，将其发送到Flume Agent
- Agent：一个独立的Flume进程，包含组件Source、Channel、Sink
- Source：用来消费传递到该组件的Event
- Channel：中转Event的一个临时存储，保存有Source组件传递过来的Event
- Sink：从Channel中读取并移除Event，将Event传递到Flow Pipeline中的下一个Agent（如果有的话）

搞清楚Flume中都有哪些基础的组件，这些时候构成flume整体的核心。

### Configuration：

搞清楚Flume是如何配置的；

###Channel：

channel作为连接source和sink的桥梁，可以先搞清楚channel的是如何