#JMM-顺序一致性模型

### 未同步的程序的执行特性

未同步的程序在JMM和顺序一致性模型中的执行特写差异

1. 顺序一致性模型保证了单线程内的操作会（严格）按照程序的顺序执行，而JMM并不保证线程内的操作会按程序的顺序执行（比如重排序，即便是正确同步了的程序在临界区内也是会发生重排序的）；
2. 顺序一致性模型保证所有线程只能看到一致的操作顺序，而JMM不保证所有线程能看到一致的执行顺序（比如说在多线程在写回数据的时候，又是只是将数据写回到自己的缓存中，并没有写回到主内存中，这样其他进程就无法看到这个改动）；
3. JMM不保证对64位的long型和double型变量的读写操作具有原子性，而顺序一致性模型保证对所有的内存读写操作都具有原子性(在JSR-133 中规定了把一个64位的long/double型数据的**写操作**拆分成两个32位的**写操作**来执行，任意的读操作都必须具有原子性)