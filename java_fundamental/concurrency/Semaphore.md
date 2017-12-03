#Semaphore

信号量用来控制具有一定数量的资源的并发访问，信号量的有无/多少表征了要访问资源的有无/多少；

Semaphore，在API是这么介绍的：

​	一个计数信号量。从概念上讲，信号量维护了一个许可集。如有必要，在许可可用前会阻塞每一个 acquire()，然后再获取该许可。每个 release() 添加一个许可，从而可能释放一个正在阻塞的获取者。但是，不使用实际的许可对象，Semaphore 只对可用许可的号码进行计数，并采取相应的行动。

​	号量Semaphore是一个非负整数（>=1）。当一个线程想要访问某个共享资源时，它必须要先获取Semaphore，当Semaphore >0时，获取该资源并使Semaphore – 1。如果Semaphore值 = 0，则表示全部的共享资源已经被其他线程全部占用，线程必须要等待其他线程释放资源。当线程释放资源时，Semaphore则+1；

### Semaphore 实现

Semaphore的实现主要依靠内部的Sync类，Sync类有Fair和NonFair两种版本，Sync继承了AQS，这样是同步功能实现的核心，其中非公平为默认的实现方式；

![201702170001](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/05/201702170001_thumb.jpg)

```
abstract static class Sync extends AbstractQueuedSynchronizer {}

static final class NonfairSync extends Sync {
    private static final long serialVersionUID = -2694183684443567898L;

    NonfairSync(int permits) {
        super(permits);
    }

    protected int tryAcquireShared(int acquires) {
        return nonfairTryAcquireShared(acquires);
    }
}

static final class FairSync extends Sync {
    private static final long serialVersionUID = 2014338818796000944L;

    FairSync(int permits) {
        super(permits);
    }

    protected int tryAcquireShared(int acquires) {
        for (;;) {
            if (hasQueuedPredecessors())
                return -1;
            int available = getState();
            int remaining = available - acquires;
            if (remaining < 0 ||
                compareAndSetState(available, remaining))
                return remaining;
        }
    }
}
```

​	当信号量Semaphore = 1 时，它可以当作互斥锁使用。其中0、1就相当于它的状态，当=1时表示其他线程可以获取，当=0时，排他，即其他线程必须要等待。

### 构造器

```
public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}

public Semaphore(int permits, boolean fair) {
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}
```

### Public方法

信号量的获取：

```
public void acquire() throws InterruptedException {
	//获取一个信号量，调用AQS的acquireSharedInterruptibly()方法，参数为1；
	sync.acquireSharedInterruptibly(1);
}

public final void acquireSharedInterruptibly(int arg)
        throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    if (tryAcquireShared(arg) < 0)
        doAcquireSharedInterruptibly(arg);
}


```

说明：tryAcquireShared（arg）方法是在Sync的子类中实现的，FairSync和NonFairSync中都实现了该方法；

其中公平模式的tryAcquireShared：

```
protected int tryAcquireShared(int acquires) {
    for (;;) {
    	//判断当前线程是否在队列头部；
        if (hasQueuedPredecessors())
            return -1;
        int available = getState();
        int remaining = available - acquires;
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;
    }
}
```

非公平模式：

```
protected int tryAcquireShared(int acquires) {
    return nonfairTryAcquireShared(acquires);
}

final int nonfairTryAcquireShared(int acquires) {
    for (;;) {
        int available = getState();
        int remaining = available - acquires;
        if (remaining < 0 ||
            compareAndSetState(available, remaining))
            return remaining;
    }
}
```

信号量的批量获取：

```
public void acquire(int permits) throws InterruptedException {
    if (permits < 0) throw new IllegalArgumentException();
    sync.acquireSharedInterruptibly(permits);
}
```

```
public boolean tryAcquire(int permits) {
    if (permits < 0) throw new IllegalArgumentException();
    return sync.nonfairTryAcquireShared(permits) >= 0;
}

public boolean tryAcquire(int permits, long timeout, TimeUnit unit)
    throws InterruptedException {
    if (permits < 0) throw new IllegalArgumentException();
    return sync.tryAcquireSharedNanos(permits, unit.toNanos(timeout));
}
```

信号量的释放：

```
public void release() {
	sync.releaseShared(1);
}
```

```
public void release(int permits) {
	if (permits < 0) throw new IllegalArgumentException();
	sync.releaseShared(permits);
}

public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```



参考：

1. http://cmsblogs.com/?p=2263