# Some fitfalls about Runtime.getRuntime().exec();

这里略过基本的Runtime.getRuntime().exec()相关知识，只总结一些平时使用的过程中可能犯的错误。



## 1. 读写输出流导致子进程阻塞

由于Runtime.getRuntime.exec()所创建子进程的三个标准IO：stdin，stdout，stderr，父进程使用这些流来提供到子进程的输入和获取从子进程的输出。

