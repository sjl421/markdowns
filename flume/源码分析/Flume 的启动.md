# Flume 的启动

 启动的入口为　apache-flume/bing/flume-ng，这是一个shell脚本，该脚本中读取了通过文件配置的jvm的一些启动参数，然后也提示除了启动的入口位置是flume的Application类，想搞清楚flume是如何启动的，看懂这个脚本就可以了。