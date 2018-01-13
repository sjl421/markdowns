#Volatile

volatile 的 特点：

1. 可见性：对一个volatile变量的读，总能看到（任意线程)对这个变量最后的写入；
2. 原子性：对任意单个volatile变量的读写操作具有原子性 ，但类似volatile++ 这种复合操作不具有原子性；
3. 防止指令重排，对于使用volatile修饰的变量，volatile会阻止JVM对其相关代码进行指令重排，这样就能够按照既定的顺序指执行。

### volatile写-读内存的语义

volatile写内存的语义：

当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值刷新到主内存；

volatile读内存的语义：

当度一个volatile变量时，JMM会把该线程对应的本地内存置为无效。线程接下来将会从主内存中读取共享变量；

结合进程间通信的理解：

1.  线程A写一个volatile比那里那个，实质上是线程A向接下来将哟啊读这个volatile变量的某个线程发出了（其对共享变量所在修改的）消息；
2. 线程B读一个volatile变量，实质上是线程B接受了之前某个线程发出的（在写这个volatile变量之前对共享变量所做修改的）消息；
3. 线程A写一个volatile变量，随后线程B读这个volatile变量，这个过程实质上是线程A通过主内存向线程B发送消息；  

### volatile内存语义的实现

为了实现volatile的内存语义，编译器在生成字节码时，会在指令序列只能插入内存屏障来禁止特定类型的处理器排序。