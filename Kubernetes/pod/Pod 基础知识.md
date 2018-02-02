# Pod 基础知识

1. 用户发送删除pod的命令，默认优雅删除时期是30秒；
2. 在Pod超过该优雅删除期限后API server就会更新Pod的状态为“dead”；
3. 在客户端命令行上显示的Pod状态为“terminating”；
4. 跟第三步同时，当kubelet发现pod被标记为“terminating”状态时，开始停止pod
  进程：
  i. 如果在pod中定义了preStop hook，在停止pod前会被调用。如果在优雅删
  除期限过期后，preStop hook依然在运行，第二步会再增加2秒的优雅时
  间；
  ii. 向Pod中的进程发送TERM信号；
5. 跟第三步同时，该Pod将从该service的端点列表中删除，不再是replication
  controller的一部分。关闭的慢的pod将继续处理load balancer转发的流量；
6. 过了优雅周期后，将向Pod中依然运行的进程发送SIGKILL信号而杀掉进程。
7. Kublete会在API server中完成Pod的的删除，通过将优雅周期设置为0（立即删
  除） 。Pod在API中消失，并且在客户端也不可见。 

