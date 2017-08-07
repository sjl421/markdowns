# Java 通过Runtime.exec()执行Shell时需要注意的问题

java中通常通过Runtime.exec()来执行shell语句或者脚本,出了网上很多帖子提到的需要注意的一些问题之外,这里记录一个我自己在开发的过程中遇到的一个坑, 是一个自己之前并没有想到的坑,同时由于自己的debug的过程中没有详尽的获取所有的bug线索,导致解这个bug也花费了自己很长的一段时间, 记录在此, 警醒自己.

### 过程

要实现的功能是在java中ssh到某一个机器然后执行一句shell语句,获取输出结果,执行的纯的shell语句如下:

```shell
ssh m1.adb.g1.com "ps -ef | grep '/usr/local/adb_core/bin/postgres' |  grep -v 'grep' | grep '\-M master' | awk '{print \$2}'"
或者:
ssh gpadmin@m1.adb.g1.com "ps -ef | grep 'primary process' | grep '5432' | grep -v 'grep' | awk '{print \$2}'"
```

这两句shell语句直接的终端下执行是没有问题的,但是最开始的时候一放到java中通过`Runtime.getRuntime.exec()`来执行的时候就是不成功, 忧桑的是这个不成功之间有好几个部分的不成功:

`Process p = Runtime.getRuntime().exec(cmd)` 来运行;

1. 调用p.getexitValue() 来获取结果

   shell语句一提交后就直接调用getexitValue()来获取程序的执行结束状态,这样导致的结果就是如果在shell语句还没有执行结束的时候就调用getexitValue()来获取执行结果,会导致`process hasn't exit`(好像是这么一个提示), 正确的做法是应该调用`p.waitfor()`来获取进程的执行状态,`p.waitfor()` 在shell语句没有执行结束之前会阻塞等待,直到shell语句执行结束;

   ---

2. 上面的`1`. 还只是一个一个小错误,最让自己懊恼的是第二个错误, 在我解掉第一个错误的时候程序运行还是不对,在我的程序执行的过程中,我通过一个线程来读取shell执行过程中的输出流和错误流(这是必须的做法,否则会容易出错),而我的获取结果的输出流中确实始终没有信息输出,程序的运行状态也不对,这就有点懊恼了,而此刻,最让我生气的是我竟然没有想到过去看看错误流中有没有什么信息, 直到一个偶然的机会让我看到了错误流中的错误信息,这才让我知道了到底是哪里出了问题;

   虽然我的shell语句在终端下是可以执行成功地,但是终端下有一些依赖的东西是配置好的,比如说:环境变量,我直接拿终端下的语句咋java中跑, 这样依赖的环境不对,语句自然也就执行的不对了,上面的shell语句提到的错误信息就是执行的命令找不到,然后通过在shell语句中执行完整路径的命令才得意解决;

   以上的说法都是错误的;

   ---