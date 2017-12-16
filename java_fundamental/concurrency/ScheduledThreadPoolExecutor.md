#ScheduledThreadPoolExecutor

* ScheduledThreadPoolExecutor是一个size固定的pool,corePoolSize是在创建的时候就固定了的;
* 分为 scheduleAtFixedRate 和 scheduleWithFixedDelay 两种方法;
* threadPoolExecutor 内部的任务队列是无界的;
* ScheduledFutureTask
* DelayedWorkQueue


###ScheduledThreadPoolExecutor介绍

...

![ScheduledThreadPoolExecutor](G:\snap\ScheduledThreadPoolExecutor.png)

### ScheduledExecutorService

继承了ExecutorService, 定义了让任务"延时执行" 和 "定期执行" 的方法.

```
// 创建并执行在给定延迟后启用的一次性操作。
public ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit);

// 创建并执行在给定延迟后启用的 ScheduledFuture。
<V> ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit)

// 创建并执行一个在给定初始延迟后首次启用的定期操作，后续操作具有给定的周期；也就是将在 initialDelay 后开始执行，然后在 initialDelay+period 后执行，接着在 initialDelay + 2 * period 后执行，依此类推。
说明:scheduleAtFixedRate ，是以上一个任务开始的时间计时，period时间过去后，检测上一个任务是否执行完毕，如果上一个任务执行完毕，则当前任务立即执行，如果上一个任务没有执行完毕，则需要等上一个任务执行完毕后立即执行。
public ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period,
											  TimeUnit unit);
// 创建并执行一个在给定初始延迟后首次启用的定期操作，随后,在每一次执行终止和下一次执行开始之间都存在给定的延迟。
public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay,long delay,                                                     TimeUnit unit);	
```

###ScheduledFuture接口

​	![ScheduledFuture](G:\snap\ScheduledFuture.png)

作为ScheduledExecutorService 的所调度任务的执行结果,  调用`ScheduledExecutorService` 接口的方法返回的结果,通过该结果可以后续取消锁调度任务的执行,也可以在任务执行结束之后获取任务执行的结果;继承了Future和Deleyed接口,这些特性都是都为了ScheduledExecutorService 提供服务的;

###构造器

```
    public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
                new DelayedWorkQueue());
    }

    public ScheduledThreadPoolExecutor(int corePoolSize,
                                       ThreadFactory threadFactory) {
        super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
                new DelayedWorkQueue(), threadFactory);
    }

    public ScheduledThreadPoolExecutor(int corePoolSize,
                                       RejectedExecutionHandler handler) {
        super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
                new DelayedWorkQueue(), handler);
    }


    public ScheduledThreadPoolExecutor(int corePoolSize,
                                       ThreadFactory threadFactory,
                                       RejectedExecutionHandler handler) {
        super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
                new DelayedWorkQueue(), threadFactory, handler);
    }
//super()类为:
public ThreadPoolExecutor(int corePoolSize,
						  int maximumPoolSize,
						  long keepAliveTime,
						  TimeUnit unit,
						  BlockingQueue<Runnable> workQueue) {
	this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
		 Executors.defaultThreadFactory(), defaultHandler);
}
```

> 当然我们一般都不会直接通过其构造函数来生成一个ScheduledThreadPoolExecutor对象（例如new ScheduledThreadPoolExecutor(10)之类的），而是通过Executors类（例如Executors.newScheduledThreadPool(int);）
>
> 在ScheduledThreadPoolExecutor的构造函数中，我们发现它都是利用ThreadLocalExecutor来构造的，唯一变动的地方就在于它所使用的阻塞队列变成了DelayedWorkQueue，而不是ThreadLocalhExecutor的LinkedBlockingQueue（通过Executors产生ThreadLocalhExecutor对象）。DelayedWorkQueue为ScheduledThreadPoolExecutor中的内部类，它其实和阻塞队列DelayQueue有点儿类似。DelayQueue是可以提供延迟的阻塞队列，它只有在延迟期满时才能从中提取元素，其列头是延迟期满后保存时间最长的Delayed元素。如果延迟都还没有期满，则队列没有头部，并且 poll 将返回 null。

###核心方法

ScheduledExecutorService 提供的核心的调服方法主要就是`ScheduledExecutorService`中定义的四个方法;

```
public ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit);
<V> ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit)
public ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period,
											  TimeUnit unit);
public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay,long delay,                                                     TimeUnit unit);	
```

```
    public ScheduledFuture<?> schedule(Runnable command,
                                       long delay,
                                       TimeUnit unit) {
        if (command == null || unit == null)
            throw new NullPointerException();
        RunnableScheduledFuture<?> t = decorateTask(command,
            new ScheduledFutureTask<Void>(command, null,
                                          triggerTime(delay, unit)));
        delayedExecute(t);
        return t;
    }
```

```
    public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
                                                  long initialDelay,
                                                  long period,
                                                  TimeUnit unit) {
        if (command == null || unit == null)
            throw new NullPointerException();
        if (period <= 0)
            throw new IllegalArgumentException();
        ScheduledFutureTask<Void> sft =
            new ScheduledFutureTask<Void>(command,
                                          null,
                                          triggerTime(initialDelay, unit),
                                          unit.toNanos(period));
        RunnableScheduledFuture<Void> t = decorateTask(command, sft);
        sft.outerTask = t;
        delayedExecute(t);
        return t;
    }
```

###DelayedWorkQueue

​	DelayedWorkQueue 是ScheduledThreadPool用来保存要执行的task的队列，内部采用使用数组表示的heap结构，队列中delay时间最短的元素会放在队列头；

​	内部主要通过siftUp()和siftDown()方法来实现元素在heap中的有序排列；

```
private static final int INITIAL_CAPACITY = 16;
private RunnableScheduledFuture<?>[] queue =
	new RunnableScheduledFuture<?>[INITIAL_CAPACITY];
private final ReentrantLock lock = new ReentrantLock();	
```

###获取元素

**offer()**

```
public boolean offer(Runnable x) {
    if (x == null)
        throw new NullPointerException();
    RunnableScheduledFuture<?> e = (RunnableScheduledFuture<?>)x;
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        int i = size;
        if (i >= queue.length)
            grow();
        size = i + 1;
        //队列为空，将元素插入到队列头
        if (i == 0) {
            queue[0] = e;
            setIndex(e, 0);
        } else {
            //队列不为空
            siftUp(i, e);
        }
        if (queue[0] == e) {
            leader = null;
            available.signal();
        }
    } finally {
        lock.unlock();
    }
    return true;
}
```

```
private void siftUp(int k, RunnableScheduledFuture<?> key) {
    //以上虑的方式在heap（数组）中寻找放置key的位置
    while (k > 0) {
        int parent = (k - 1) >>> 1;
        RunnableScheduledFuture<?> e = queue[parent];
        if (key.compareTo(e) >= 0)
            //找到了放置key的位置，退出循环
            break;
        queue[k] = e;
        //设置索引
        setIndex(e, k);
        k = parent;
    }
    //将key放置到位置中
    queue[k] = key;
    //设置索引
    setIndex(key, k);
}
```

```
private void siftDown(int k, RunnableScheduledFuture<?> key) {
    int half = size >>> 1;
    while (k < hal f) {
        int child = (k << 1) + 1;
        RunnableScheduledFuture<?> c = queue[child];
        int right = child + 1;
        if (right < size && c.compareTo(queue[right]) > 0)
            c = queue[child = right];
        if (key.compareTo(c) <= 0)
            break;
        queue[k] = c;
        setIndex(c, k);
        k = child;
    }
    queue[k] = key;
    setIndex(key, k);
}
```

我们举例说明：
[![21](http://www.liuinsect.com/wp-content/uploads/2014/11/21.png)](http://www.liuinsect.com/wp-content/uploads/2014/11/21.png)
这里的被移除元素分为两类，一类是被移除元素没有子节点的，一类是被移除元素有子节点的。
从表中，根据二叉树的性质可以知道，位置索引大于等于队列长度一半的，没有子节点，小于队列长度一半的有子节点。
因此，分两类来看：

第一种情况：
被移除元素没有子节点的：
[![22](http://www.liuinsect.com/wp-content/uploads/2014/11/22.png)](http://www.liuinsect.com/wp-content/uploads/2014/11/22.png)
这个时候，可知，siftDown方法中，k=6，key=[9]

1. 计算后half=5，k>half,说明被移除元素没有子节点。直接将[9]也就是原有队列的最后一个元素 9号元素放入索引位置为6的位置后，结束。

[![23](http://www.liuinsect.com/wp-content/uploads/2014/11/23.png)](http://www.liuinsect.com/wp-content/uploads/2014/11/23.png)
第二种情况：
被移除元素有子节点的。
[![24](http://www.liuinsect.com/wp-content/uploads/2014/11/24.png)](http://www.liuinsect.com/wp-content/uploads/2014/11/24.png)
这个时候，可知，k=3，key=[9]

1. 计算后可知，k<half,child=7，c=[7],right=8.

2. 进入第一个

   if (right < size && c.compareTo(queue[right]) > 0)
   c = queue[child = right];
   这句话可以理解为选出[7],[8]两个位置中下次执行时间（超时时间）较小的那个，赋值给c.

3. if (key.compareTo(c) <= 0)

  ​break;
  ​将c和key,比较，如果key比较小，直接将key放到[3]中，变成[7][8]的父节点
  [![25](http://www.liuinsect.com/wp-content/uploads/2014/11/25.png)](http://www.liuinsect.com/wp-content/uploads/2014/11/25.png)
  如果c比较小

```
queue[k] = c;
setIndex(c, k);
k = child;
```

将c放到位置3，然后在位置7,8中为key找合适的位置。

put， add方法的底层调用的都是offer()方法：

```
    public void put(Runnable e) {
        offer(e);
    }

    public boolean add(Runnable e) {
        return offer(e);
    }

    public boolean offer(Runnable e, long timeout, TimeUnit unit) {
        return offer(e);
    }
```

**poll()**

非阻塞的从队列中获取元素，如果队列为空，则返回null；

```
public RunnableScheduledFuture<?> poll() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        //取队列的首元素，判断delay时间是否到达，如果可以到达，则返回该元素
        RunnableScheduledFuture<?> first = queue[0];
        if (first == null || first.getDelay(NANOSECONDS) > 0)
            return null;
        else
            return finishPoll(first);
    } finally {
        lock.unlock();
    }
}

private RunnableScheduledFuture<?> finishPoll(RunnableScheduledFuture<?> f) {
    int s = --size;
    //取出队列的末尾元素，将其放入到heap的头部后再下虑
    RunnableScheduledFuture<?> x = queue[s];
    queue[s] = null;
    if (s != 0)
        siftDown(0, x);
    setIndex(f, -1);
    return f;
}
```

**take()**

以阻塞可中断的方式从DelayedWorkQueue中获取元素；

```
    public RunnableScheduledFuture<?> take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            for (;;) {
                RunnableScheduledFuture<?> first = queue[0];
                if (first == null)
                    available.await();
                else {
                    long delay = first.getDelay(NANOSECONDS);
                    if (delay <= 0)
                        return finishPoll(first);
                    first = null; // don't retain ref while waiting
                    if (leader != null)
                        available.await();
                    else {
                        Thread thisThread = Thread.currentThread();
                        leader = thisThread;
                        try {
                            available.awaitNanos(delay);
                        } finally {
                            if (leader == thisThread)
                                leader = null;
                        }
                    }
                }
            }
        } finally {
            if (leader == null && queue[0] != null)
                available.signal();
            lock.unlock();
        }
    }
```




###ScheduledFutureTask

用来描述可调度任务；

```
private class ScheduledFutureTask<V>
		extends FutureTask<V> implements RunnableScheduledFuture<V>;
```

Fields：

```
private final long sequenceNumber;	//序列号        
/* 用来描述任务的执行的方式是单次（ont-shot）还是fixed-rate 还是 fixed-delay;
 * 0： one-shot；
 * 正数：fixed-rate；
 * 负数：fixed-delay；
 */
private final long period;
//设置的任务执行的时间点
private long time;
//该任务在Delay Queue中的索引值，用来快速Cancell任务
int heapIndex;
```

构造器：

```
ScheduledFutureTask(Runnable r, V result, long ns) {
	super(r, result);
	this.time = ns;
	this.period = 0;	//氮气
	this.sequenceNumber = sequencer.getAndIncrement();
}

ScheduledFutureTask(Runnable r, V result, long ns, long period) {
	super(r, result);
	this.time = ns;
	this.period = period;
	this.sequenceNumber = sequencer.getAndIncrement();
}

ScheduledFutureTask(Callable<V> callable, long ns) {
	super(callable);
	this.time = ns;
	this.period = 0;
	this.sequenceNumber = sequencer.getAndIncrement();
}
```

**isPeriodic**

用来判断当前任务是否是periodic 任务；

```
public boolean isPeriodic() {
	return period != 0;
}
```

**run** 

重写了FutureTask的run方法， 在内部判断该任务是否是periodic的而进行不同的处理；

```
public void run() {
	boolean periodic = isPeriodic();
	if (!canRunInCurrentRunState(periodic))
		cancel(false);
	else if (!periodic)
		ScheduledFutureTask.super.run();
	else if (ScheduledFutureTask.super.runAndReset()) {
		setNextRunTime();
		reExecutePeriodic(outerTask);
	}
}
```

```
private void setNextRunTime() {
	long p = period;
	if (p > 0)
		time += p;
	else
		time = triggerTime(-p);
}
```

**poll()**

```
    public RunnableScheduledFuture<?> poll(long timeout, TimeUnit unit)
        throws InterruptedException {
        long nanos = unit.toNanos(timeout);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            for (;;) {
                RunnableScheduledFuture<?> first = queue[0];
                if (first == null) {
                    if (nanos <= 0)
                        return null;
                    else
                        nanos = available.awaitNanos(nanos);
                } else {
                    long delay = first.getDelay(NANOSECONDS);
                    if (delay <= 0)
                        return finishPoll(first);
                    if (nanos <= 0)
                        return null;
                    first = null; // don't retain ref while waiting
                    if (nanos < delay || leader != null)
                        nanos = available.awaitNanos(nanos);
                    else {
                        Thread thisThread = Thread.currentThread();
                        leader = thisThread;
                        try {
                            long timeLeft = available.awaitNanos(delay);
                            nanos -= delay - timeLeft;
                        } finally {
                            if (leader == thisThread)
                                leader = null;
                        }
                    }
                }
            }
        } finally {
            if (leader == null && queue[0] != null)
                available.signal();
            lock.unlock();
        }
    }
```

```
private RunnableScheduledFuture<?> finishPoll(RunnableScheduledFuture<?> f) {
    int s = --size;
    //取出队列的末尾元素，将其放入到heap的头部后再下虑
    RunnableScheduledFuture<?> x = queue[s];
    queue[s] = null;
    if (s != 0)
        siftDown(0, x);
    setIndex(f, -1);
    return f;
}
```

**remove()**

```
public boolean remove(Object x) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        //找到x元素的索引
        int i = indexOf(x);
        if (i < 0)
            return false;

        setIndex(queue[i], -1);
        //获取队列中的最后一个元素
        int s = --size;
        RunnableScheduledFuture<?> replacement = queue[s];
        queue[s] = null;
        if (s != i) {
            //通过siftDown寻找合适的元素放置到要替换掉的索引位置
            siftDown(i, replacement);
            if (queue[i] == replacement)
                siftUp(i, replacement);
        }
        return true;
    } finally {
        lock.unlock();
    }
}
```



参考文献：

1. http://ju.outofmemory.cn/entry/99456