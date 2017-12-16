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

###获取元素

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

