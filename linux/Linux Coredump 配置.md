# Linux Coredump 配置

1. 查看当前系统Coredump文件格式

```
cat /proc/sys/kernel/core_pattern
#默认值是/usr/libexec/abrt-hook-ccpp %s %c %p %u %g %t e %P %I
```

2. ​

```
ulimit -c
```

3. ​

```
sysctl –p /etc/sysctl.conf
```



### 修改linux Core 设置

方法1：临时修改：修改/proc/sys/kernel/core_pattern文件，但/proc目录本身是动态加载的，每次系统重启都会重新加载，因此这种方法只能作为临时修改。

```
/proc/sys/kernel/core_pattern
例：echo ‘/var/log/%e.core.%p’ > /proc/sys/kernel/core_pattern
```

​	方法2：永久修改：可以通过在/etc/sysctl.conf文件中，对sysctl变量kernel.core_pattern的设置。

```
#vi /etc/sysctl.conf 然后，在sysctl.conf文件中添加下面两句话：
kernel.core_pattern = /var/core/core_%e_%p
kernel.core_uses_pid = 1
保存后退出。

#可以使用以下命令，使修改结果马上生效。
sysctl -p /etc/sysctl.conf
```



设置core dump文件的文件名:

```
%% 单个%字符
%p 所dump进程的进程ID
%u 所dump进程的实际用户ID
%g 所dump进程的实际组ID
%s 导致本次core dump的信号
%t core dump的时间 (由1970年1月1日计起的秒数)
%h 主机名
%e 程序文件名
%c 转储文件的大小上限
```



```
/etc/security/limits.conf  文件配置示例

## 说明各个列的含义
##<domain>      <type>   <item>    <value>
##........其他账户配置
##root的配置
@root                  soft         core        unlimited
@root                  hard        core       unlimited

配置好后，reboot重启服务器，这样在root组下的用户，其配置生效；其他组的用户不生效
参考文章http://www.jbxue.com/LINUXjishu/1250.html
```

```
前阵子，我要用到使LInux的文件打开数为65534个，而且需要永久生效，于是将配置写到了：
vim /etc/security/limits.conf
* soft nofile 65534
* hard nofile 65534
重新登录后limit.conf的配置都不生效，后来发现，ubuntu有个bug，root用户必须注明用户
root soft nofile 65534
root hard nofile 65534
也就是写成上面那样，重新登录，不需要重启，ulimit -a可以看到文件打开数已经是65534了，这就是limits.conf不生效的原因，注意ubuntu一定不能直接用*
```

```
vim /etc/security/limits.conf
添加一下设置
gpadmin soft core 131072
gpadmin hard core 131072
#退出登录后切换到gpadmin用户,执行 
ulimit -a
core file size          (blocks, -c) 131072
```

