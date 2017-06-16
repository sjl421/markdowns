# shell kill process

1. 查看指定的进程的进程pid

```shell
ps -ef | grep $processname | grep -v "grep" | awk '{print $2} '
```

其中, `grep -v "grep"` 命令是为了排除掉 grep 在查找制定的pid是会显示grep的进程pid;

`grep -v xxx`命令是查找所有语句中不出现这个xxx字段的行;

2. 如果要kill掉指定查找到的进程id,只需要在前面加上`kill -9`

```shell
kill -9 ps -ef | grep $processname | grep -v "grep" | awk '{print $2} '
```

```shell
kill -9 `ps -ef|grep "processname" | grep -v "grep"|awk '{print $2}'`
```

```shell
#!/bin/bash
pids=$(ps -ef|grep "tomcat" | grep -v "grep" | awk '{print $2}')
for pid in $pids; do
  echo "kill tomcat process: " $pid
  kill -9 $pid
done
```

