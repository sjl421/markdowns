# Java中阻塞队列的区别

### 实现方式上

- ArrayBlockingQueue:一个有数组组结构组成的有界阻塞队列;
- LinkedBlockingQueue:一个有链表组成的有界阻塞队列;
- PriorityBlockingQueue:一个支持优先级排序的无界阻塞队列;
- DelayQueue:一个使用优先级队列实现的无界阻塞队列;
- SynchronousQueue:一个不存储元素的阻塞队列;
- LinkedTransferQueue:一个有链表组结构组成的无界阻塞队列;

### 特点



###LinkedBlockingQueue

LinkedBlockingQueue 是通过双向链表的实现的阻塞队列，支持的最大的长度Integer.MaxValue, 	同事构造器中可以指定队列的最大长度。

双向队列避免了队列的“过度的膨胀”， 通过链表的实现方式可以再未知队列具体的容量时可以采用LinkedBlockingQueue作为折中的选择。

双端队列的大部分操作都运行在常数的时间范围内（不包含本质上需要遍历队列的操作）

LinkedBlockingQueue可以工作在工作窃取模式中；