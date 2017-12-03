# centos 修改hostname

1. 使用hostname 命令

   hostname 命令可以查看的当前主机名，但是这种修改是临时的，主机重启后就会回复为原来的主机名；

```shell
:$ hostname newhostname	# 修改主机名为newhostname
:$ hostname				# 查看修改后的主机名,
						#为了使终端中的提示符显示新的hostname，需要重新打开一个终端，或者退出并重新登录终端；
```

2. 永久修改hostname

主机的hostname是配置在 /etc/hostname 文件中的，手动修改这个文件可以达到永久修改hostname的目的，

```shell
[root@m1 ~]# cat /etc/hostname 
m1.adb.g1.com
```

但是为了让你的修改生效，需要重启主机；或者结合hostname命令，这样保证了当前的配置立马生效了，并且重启主机后会使/etc/hostname 文件中的配置永久的生效；

###  hosts文件与主机名修改无关

http://blog.csdn.net/forest_boy/article/details/5636696

> 一些网络文章中提出修改主机名还需修改Hosts文件，其实hosts文件和主机名修改无关。
>
> hosts文件是配本地主机名/域名解析的。
>
> 如我本机ip是192.168.11.116名字是zhh64.就可以直接访问主机名。
>
> zhouhh@zhh64:~$ ping zhh64
> PING zhh64 (192.168.11.116) 56(84) bytes of data.
> 64 bytes from zhh64 (192.168.11.116): icmp_seq=1 ttl=64 time=0.077 ms
>
> zhouhh@zhh64:~$ ping centdev
> PING centdev (192.168.12.14) 56(84) bytes of data.
> 64 bytes from centdev (192.168.12.14): icmp_seq=1 ttl=63 time=0.726 ms
>
> 如果是小型局域网，就可以将hosts文件机器配全了，拷贝到每个机器，然后在ssh访问时用主机名直接访问。

第二个解释：

http://www.ctohome.com/FuWuQi/1b/414.html

### 永久更改Linux的hostname

man hostname里有这么一句话，”The host name is usually set once at system startup in /etc/rc.d/rc.inet1 or /etc/init.d/boot (normally by reading the contents of a file which contains the host name, e.g. /etc/hostname).” RedHat里没有这个文件，而是由/etc/rc.d/rc.sysinit这个脚本负责设置系统的hostname，它读取/etc /sysconfig/network这个文本文件，RedHat的hostname就是在这个文件里设置。

所以，如果要永久修改RedHat的hostname，就修改/etc/sysconfig/network文件，将里面的HOSTNAME这一行修改成 HOSTNAME=NEWNAME，其中NEWNAME就是你要设置的hostname。

Debian发行版的hostname的配置文件是/etc/hostname。

修该配置文件后，重启系统就会读取配置文件设置新的hostname。

### hostname与/etc/hosts的关系

很过人一提到更改hostname首先就想到修改/etc/hosts文件，认为hostname的配置文件就是/etc/hosts。其实不是的。

hosts文件的作用相当如DNS，提供IP地址到hostname的对应。早期的互联网计算机少，单机hosts文件里足够存放所有联网计算机。 不过随着互联网的发展，这就远远不够了。于是就出现了分布式的DNS系统。由DNS服务器来提供类似的IP地址到域名的对应。具体可以man hosts。

Linux系统在向DNS服务器发出域名解析请求之前会查询/etc/hosts文件，如果里面有相应的记录，就会使用hosts里面的记录。/etc /hosts文件通常里面包含这一条记录

```
127.0.0.1    localhost.localdomain   localhost
```

hosts文件格式是一行一条记录，分别是IP地址 hostname aliases，三者用空白字符分隔，aliases可选。

127.0.0.1到localhost这一条建议不要修改，因为很多应用程序会用到这个，比如sendmail，修改之后这些程序可能就无法正常运行。

修改hostname后，如果想要在本机上用newhostname来访问，就必须在/etc/hosts文件里添加一条newhostname的记录。比如我的eth0的IP是192.168.1.61，我将hosts文件修改如下：

```shell
:$ hostname www.cthome.com
:$ cat /etc/hosts
127.0.0.1  localhost.localdomain localhost
192.168.1.61    www.cthome.com       blog
```

这样，我就可以通过blog或者www.cthome.com来访问本机。

从上面这些来看，/etc/hosts于设置hostname是没直接关系的，仅仅当你要在本机上用新的hostname来访问自己的时候才会用到 /etc/hosts文件。两者没有必然的联系。



---

# 如何在CentOS 7上修改主机名

http://www.jianshu.com/p/39d7000dfa47

在CentOS中，有三种定义的主机名:静态的（static），瞬态的（transient），和灵活的（pretty）。`静态`主机名也称为内核主机名，是系统在启动时从/etc/hostname自动初始化的主机名。`瞬态`主机名是在系统运行时临时分配的主机名，例如，通过DHCP或mDNS服务器分配。静态主机名和瞬态主机名都遵从作为互联网域名同样的字符限制规则。而另一方面，`灵活`主机名则允许使用自由形式（包括特殊/空白字符）的主机名，以展示给终端用户（如qqmm）。
在CentOS 7中，有个叫hostnamectl的命令行工具，它允许你查看或修改与主机名相关的配置。

1. 要查看主机名相关的设置：

   ```
   [root@localhost ~]# hostnamectl
   Static hostname: localhost.localdomain
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 21ff9d4ebdd94e949b9fd6cbdb1926c0
           Boot ID: 2a952e91c02841e3ae10de0d16dd3f01
    Virtualization: kvm
   Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-327.el7.x86_64
      Architecture: x86-64
   ```

   ```
   [root@localhost ~]# hostnamectl status
   Static hostname: localhost.localdomain
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 21ff9d4ebdd94e949b9fd6cbdb1926c0
           Boot ID: 2a952e91c02841e3ae10de0d16dd3f01
    Virtualization: kvm
   Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-327.el7.x86_64
      Architecture: x86-64
   ```

2. 只查看静态、瞬态或灵活主机名，分别使用`--static`，`--transient`或`--pretty`选项。

   ```
   [root@localhost ~]# hostnamectl --static
   localhost.localdomain
   [root@localhost ~]# hostnamectl --transient
   localhost.localdomain
   [root@localhost ~]# hostnamectl --pretty
   ```

3. 要同时修改所有三个主机名：静态、瞬态和灵活主机名：

   ```
   [root@localhost ~]# hostnamectl set-hostname qqmm
   [root@localhost ~]# hostnamectl --pretty
   [root@localhost ~]# hostnamectl --static
   qqmm
   [root@localhost ~]# hostnamectl --transient
   qqmm
   ```

   就像上面展示的那样，在修改静态/瞬态主机名时，任何特殊字符或空白字符会被移除，而提供的参数中的任何大写字母会自动转化为小写。
   一旦修改了静态主机名，`/etc/hostname` 将被自动更新。然而，`/etc/hosts` 不会更新以保存所做的修改，所以你每次在修改主机名后一定要手动更新`/etc/hosts`，之后再重启CentOS 7。否则系统再启动时会很慢。

4. 手动更新

   ```
   /etc/hosts
   ```

   ```
   vim /etc/hosts
   #127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
   127.0.0.1  qqmm
   #::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
   ::1        qqmm
   ```

5. 重启CentOS 7 之后（reboot -f ），

   ```
   [root@qqmm ~]# hostname
   qqmm
   [root@qqmm ~]# hostnamectl
   Static hostname: qqmm
        Icon name: computer-vm
          Chassis: vm
       Machine ID: 21ff9d4ebdd94e949b9fd6cbdb1926c0
          Boot ID: 2a952e91c02841e3ae10de0d16dd3f01
   Virtualization: kvm
   Operating System: CentOS Linux 7 (Core)
      CPE OS Name: cpe:/o:centos:centos:7
           Kernel: Linux 3.10.0-327.el7.x86_64
     Architecture: x86-64
   ```

6. 如果你只想修改特定的主机名（静态，瞬态或灵活），你可以使用

   ```
   --static
   ```

   ```
   --transient
   ```

   或

   ```
   --pretty
   ```

   选项。例如，要永久修改主机名，你可以修改静态主机名：

   ```
   [root@localhost ~]# hostnamectl --static set-hostname qqmm
   ```

   重启CentOS 7 之后(reboot -f),

   ```
   [root@localhost ~]# hostnamectl --static
   qqmm
   [root@localhost ~]# hostnamectl --transient
   qqmm
   [root@localhost ~]# hostnamectl --pretty
   qqmm
   [root@localhost ~]# hostname
   qqmm
   ```

   其实，你不必重启机器以激活永久主机名修改。上面的命令会立即修改内核主机名。

   注销并重新登入后在命令行提示来观察新的静态主机名