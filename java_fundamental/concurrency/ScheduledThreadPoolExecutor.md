#ScheduledThreadPoolExecutor

* ScheduledThreadPoolExecutor是一个size固定的pool,corePoolSize是在创建的时候就固定了的;
* 分为 scheduleAtFixedRate 和 scheduleWithFixedDelay 两种方法;
* threadPoolExecutor 内部的任务队列是无界的;
* ScheduledFutureTask
* DelayedWorkQueue






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

