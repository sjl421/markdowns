#Linux Coredump配置

Core dump的配置主要是两部分:

1. 配置Core dump 文件的生成位置和文件的命名规则
2. 设置生成的core dump 文件的大小

### 一. 设置生成文件的位置和文件命名规则

####1. 临时修改

修改`/proc/sys/kernel/core_pattern`文件,但/proc目录本身是动态加载的，每次系统重启都会重新加载，因此这种方法只能作为临时修改。

```
echo '/var/log/%e.core.%p' > /proc/sys/kernel/core_pattern
```

####2. 永久修改

可以通过在/etc/sysctl.conf文件中，对sysctl变量kernel.core_pattern的设置。

```
#在/etc/sysctl.conf文件中添加下面两句话;
#注: 需要先手动创建/var/core目录, 并且目标程序需要拥有对该目录的写权限
kernel.core_pattern = /var/core/core_%e_%p 
kernel.core_uses_pid = 1
保存后退出。

#使修改结果马上生效
sysctl –p /etc/sysctl.conf
```

### 设置Core dump文件的大小

通过`ulimit -c` 查看系统设置的Core dump文件的大小, 如果该配置项为0, 表示不生成core dump文件;

```
-c <core文件上限> 　设定core文件的最大值，单位为区块。 
```

```
# ulimit -c 为0 表示,不生成core dump文件;
[root@m1 ~]# ulimit -c
0
```

设置Core dump 文件的大小

####1. 临时修改

 `ulimit -c 文件大小` 来设置core dump文件的大小. 

`ulimit -c unlimited` 设置core文件大小为不限制大小

```
[root@m1 ~]# ulimit -c 1000
[root@m1 ~]# ulimit -c
1000

[root@m1 ~]# ulimit -c unlimited
[root@m1 ~]# ulimit -c
unlimited
```

#### 2. 永久修改

通过修改limit.conf里使修改永久生效

```
vim /etc/security/limits.conf
#添加以下设置,文件中有配置格式的说明,其中value为KB
#<domain> <type> <item> <value>
gpadmin soft core 131072
gpadmin hard core 131072

#退出登录后切换到gpadmin用户,执行 
ulimit -a (查看是否有以下输出)
core file size          (blocks, -c) 131072
```

关于配置项中的soft/hard选项的解释:

```
A hard limit is the maximum allowed to a user, set by the superuser/root. This value is set in the file /etc/security/limits.conf. Think of it as an upper bound or ceiling or roof.
A soft limit is the effective value right now for that user. The user can increase the soft limit on their own in times of needing more resources, but cannot set the soft limit higher than the hard limit.
```

####3. 测试: foo.cpp

```
#include <iostream>
using namespace std;
int main() {
    *(char *)1=1;    
	return 0;
}
```

```
#1. 编译
g++ -g foo.cpp
#2. 运行测试用例,产生core dump
./a.out
#3. 去/var/core/ 文件夹下查看是否有coredump文件生成
```

