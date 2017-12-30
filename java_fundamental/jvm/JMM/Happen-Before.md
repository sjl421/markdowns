# Happen-Before

# happen—before规则介绍

​    Java语言中有一个“先行发生”（happen—before）的规则，它是Java内存模型中定义的两项操作之间的偏序关系，如果操作A先行发生于操作B，其意思就是说，在发生操作B之前，操作A产生的影响都能被操作B观察到，“影响”包括修改了内存中共享变量的值、发送了消息、调用了方法等，它与时间上的先后发生基本没有太大关系。这个原则特别重要，它是判断数据是否存在竞争、线程是否安全的主要依据。

​    举例来说，假设存在如下三个线程，分别执行对应的操作:

\---------------------------------------------------------------------------

线程A中执行如下操作：i=1

线程B中执行如下操作：j=i

线程C中执行如下操作：i=2

\---------------------------------------------------------------------------

​    假设线程A中的操作”i=1“ happen—before线程B中的操作“j=i”，那么就可以保证在线程B的操作执行后，变量j的值一定为1，即线程B观察到了线程A中操作“i=1”所产生的影响；现在，我们依然保持线程A和线程B之间的happen—before关系，同时线程C出现在了线程A和线程B的操作之间，但是C与B并没有happen—before关系，那么j的值就不确定了，线程C对变量i的影响可能会被线程B观察到，也可能不会，这时线程B就存在读取到不是最新数据的风险，不具备线程安全性。

​    下面是Java内存模型中的八条可保证happen—before的规则，它们无需任何同步器协助就已经存在，可以在编码中直接使用。如果两个操作之间的关系不在此列，并且无法从下列规则推导出来的话，它们就没有顺序性保障，虚拟机可以对它们进行随机地重排序。

​    1、程序次序规则：在一个单独的线程中，按照程序代码的执行流顺序，（时间上）先执行的操作happen—before（时间上）后执行的操作。

​    2、管理锁定规则：一个unlock操作happen—before后面（时间上的先后顺序，下同）对同一个锁的lock操作。

​    3、volatile变量规则：对一个volatile变量的写操作happen—before后面对该变量的读操作。

​    4、线程启动规则：Thread对象的start（）方法happen—before此线程的每一个动作。

​    5、线程终止规则：线程的所有操作都happen—before对此线程的终止检测，可以通过Thread.join（）方法结束、Thread.isAlive（）的返回值等手段检测到线程已经终止执行。

​    6、线程中断规则：对线程interrupt（）方法的调用happen—before发生于被中断线程的代码检测到中断时事件的发生。

​    7、对象终结规则：一个对象的初始化完成（构造函数执行结束）happen—before它的finalize（）方法的开始。

​    8、传递性：如果操作A happen—before操作B，操作B happen—before操作C，那么可以得出A happen—before操作C。

补充（http://www.importnew.com/23519.html）：

1. 将一个元素放入一个线程安全的队列的操作Happens-Before从队列中取出这个元素的操作
2. 将一个元素放入一个线程安全容器的操作Happens-Before从容器中取出这个元素的操作
3. 在CountDownLatch上的倒数操作Happens-Before CountDownLatch#await()操作
4. 释放Semaphore许可的操作Happens-Before获得许可操作
5. Future表示的任务的所有操作Happens-Before Future#get()操作
6. 向Executor提交一个Runnable或Callable的操作Happens-Before任务开始执行操作

# 时间上先后顺序和happen—before原则

​    ”时间上执行的先后顺序“与”happen—before“之间有何不同呢？

​    1、**首先来看操作A在时间上先与操作B发生，是否意味着操作A happen—before操作B？**

​    一个常用来分析的例子如下：

```
private int value = 0;  
  
public int get(){  
    return value;  
}  
public void set(int value){  
    this.value = value;  
}   
```

 假设存在线程A和线程B，线程A先（时间上的先）调用了setValue（3）操作，然后（时间上的后）线程B调用了同一对象的getValue（）方法，那么线程B得到的返回值一定是3吗？

​    对照以上八条happen—before规则，发现没有一条规则适合于这里的value变量，从而我们可以判定线程A中的setValue（3）操作与线程B中的getValue（）操作不存在happen—before关系。因此，尽管线程A的setValue（3）在操作时间上先于操作B的getvalue（），但无法保证线程B的getValue（）操作一定观察到了线程A的setValue（3）操作所产生的结果，也即是getValue（）的返回值不一定为3（有可能是之前setValue所设置的值）。这里的操作不是线程安全的。

​    因此，”一个操作时间上先发生于另一个操作“并不代表”一个操作happen—before另一个操作“。

​    解决方法：可以将setValue（int）方法和getValue（）方法均定义为synchronized方法，也可以把value定义为volatile变量（value的修改并不依赖value的原值，符合volatile的使用场景），分别对应happen—before规则的第2和第3条。注意，只将setValue（int）方法和getvalue（）方法中的一个定义为synchronized方法是不行的，必须对同一个变量的所有读写同步，才能保证不读取到陈旧的数据，仅仅同步读或写是不够的 。

​    2、**其次来看，操作A happen—before操作B，是否意味着操作A在时间上先与操作B发生？**

​    看有如下代码：

```
x = 1；  
y = 2;  
```

​    假设同一个线程执行上面两个操作：操作A：x=1和操作B：y=2。根据happen—before规则的第1条，操作A happen—before 操作B，但是由于编译器的

指令重排序（Java语言规范规定了JVM线程内部维持顺序化语义，也就是说只要程序的最终结果等同于它在严格的顺序化环境下的结果，那么指令的执行顺序就可能与代码的顺序不一致。这个过程通过叫做指令的重排序。指令重排序存在的意义在于：JVM能够根据处理器的特性（CPU的多级缓存系统、多核处理器等）适当的重新排序机器指令，使机器指令更符合CPU的执行特点，最大限度的发挥机器的性能。在没有同步的情况下，编译器、处理器以及运行时等都可能对操作的执行顺序进行一些意想不到的调整）

等原因，操作A在时间上有可能后于操作B被处理器执行，但这并不影响happen—before原则的正确性。

​    因此，”一个操作happen—before另一个操作“并不代表”一个操作时间上先发生于另一个操作“。

​    最后，一个操作和另一个操作必定存在某个顺序，要么一个操作或者是先于或者是后于另一个操作，或者与两个操作同时发生。同时发生是完全可能存在的，特别是在多CPU的情况下。而两个操作之间却可能没有happen-before关系，也就是说有可能发生这样的情况，操作A不happen-before操作B，操作B也不happen-before操作A，用数学上的术语happen-before关系是个偏序关系。两个存在happen-before关系的操作不可能同时发生，一个操作A happen-before操作B，它们必定在时间上是完全错开的，这实际上也是同步的语义之一（独占访问）。



原文地址；http://blog.csdn.net/ns_code/article/details/17348313

推荐阅读：

1. http://ifeve.com/easy-happens-before/
2. http://blog.csdn.net/ns_code/article/details/17348313
3. http://www.importnew.com/23519.html