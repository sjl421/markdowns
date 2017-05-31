# service 命令

service命令用于对系统服务进行管理，比如启动（start）、停止（stop）、重启（restart）、查看状态（status）等。相关的命令还包括chkconfig、ntsysv等，chkconfig用于查看、设置服务的运行级别，ntsysv用于直观方便的设置各个服务是否自动启动。service命令本身是一个shell脚本，它在/etc/init.d/目录查找指定的服务脚本，然后调用该服务脚本来完成任务。

看看下面的手册页可能更加清楚的了解service的内幕：service运行指定服务（称之为System V初始脚本）时，把大部分环境变量去掉了，只保留LANG和TERM两个环境变量，并且把当前路径置为/，也就是说是在一个可以预测的非常干净的环境中运行服务脚本。这种脚本保存在/etc/init.d目录中，它至少要支持start和stop命令。