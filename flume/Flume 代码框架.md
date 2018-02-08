# Flume 代码框架

需要梳理清楚几条线：

1. flume启动时是如何如何读取配置信息并生效的？
2. channel框架
3. sink框架
4. source框架
5. channel，sink，source组合起立的框架





Application.java

主要来控制所有components的生命启动和停止，配置文件的读取，解析启动参数；

Application的主要的启动流程是：

`public static void main(String[] args) ` 是整个Application的main方法，也是真个flume的入口方法，主要实现的功能有：

1. 解析命令行传入的启动参数；
2. （省略不重要的），从制定的配置文件中读取配置信息，这里只关注了