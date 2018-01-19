#ReentrantReadWriteLock

## 简介

​	 实现同步互斥的效果，它们必须用同一个Lock对象。

​	读写锁：分为读锁和写锁，多个读锁不互斥，读锁与写锁互斥，这是由jvm自己控制的，你只要上好相应的锁即可。如果你的代码只读数据，可以很多人同时读，但不能同时写，那就上读锁；如果你的代码修改数据，只能有一个人在写，且不能同时读取，那就上写锁。总之，读的时候上读锁，写的时候上写锁！

　　ReentrantReadWriteLock会使用两把锁来解决问题，一个读锁，一个写锁
线程进入读锁的前提条件：

* 没有其他线程的写锁，


* 没有写请求或者有写请求，但调用线程和持有锁的线程是同一个

线程进入写锁的前提条件：

* 没有其他线程的读锁
* 没有其他线程的写锁

到ReentrantReadWriteLock，首先要做的是与ReentrantLock划清界限。它和后者都是单独的实现，彼此之间没有继承或实现的关系。然后就是总结这个锁机制的特性了： 
​     (a).重入方面其内部的WriteLock可以获取ReadLock，但是反过来ReadLock想要获得WriteLock则永远都不要想。 
​     (b).WriteLock可以降级为ReadLock，顺序是：先获得WriteLock再获得ReadLock，然后释放WriteLock，这时候线程将保持Readlock的持有。反过来ReadLock想要升级为WriteLock则不可能，为什么？参看(a)，呵呵. 
​     (c).ReadLock可以被多个线程持有并且在作用时排斥任何的WriteLock，而WriteLock则是完全的互斥。这一特性最为重要，因为对于高读取频率而相对较低写入的数据结构，使用此类锁同步机制则可以提高并发量。 
​     (d).不管是ReadLock还是WriteLock都支持Interrupt，语义与ReentrantLock一致。 
​     (e).WriteLock支持Condition并且与ReentrantLock语义一致，而ReadLock则不能使用Condition，否则抛出UnsupportedOperationException异常。 

实例代码：

```
package com.thread;

import java.util.Random;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class ReadWriteLockTest {
    public static void main(String[] args) {
        final Queue3 q3 = new Queue3();
        for(int i=0;i<3;i++)
        {
            new Thread(){
                public void run(){
                    while(true){
                        q3.get();                        
                    }
                }
                
            }.start();
        }
        for(int i=0;i<3;i++)
        {        
            new Thread(){
                public void run(){
                    while(true){
                        q3.put(new Random().nextInt(10000));
                    }
                }            
                
            }.start();    
        }
    }
}

class Queue3{
    private Object data = null;//共享数据，只能有一个线程能写该数据，但可以有多个线程同时读该数据。
    private ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
    public void get(){
        rwl.readLock().lock();//上读锁，其他线程只能读不能写
        System.out.println(Thread.currentThread().getName() + " be ready to read data!");
        try {
            Thread.sleep((long)(Math.random()*1000));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "have read data :" + data);        
        rwl.readLock().unlock(); //释放读锁，最好放在finnaly里面
    }
    
    public void put(Object data){

        rwl.writeLock().lock();//上写锁，不允许其他线程读也不允许写
        System.out.println(Thread.currentThread().getName() + " be ready to write data!");                    
        try {
            Thread.sleep((long)(Math.random()*1000));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        this.data = data;        
        System.out.println(Thread.currentThread().getName() + " have write data: " + data);                    
        
        rwl.writeLock().unlock();//释放写锁    
    }
}
```

运行结果：

```
Thread-0 be ready to read data!
Thread-1 be ready to read data!
Thread-2 be ready to read data!
Thread-0have read data :null
Thread-2have read data :null
Thread-1have read data :null
Thread-5 be ready to write data!
Thread-5 have write data: 6934
Thread-5 be ready to write data!
Thread-5 have write data: 8987
Thread-5 be ready to write data!
Thread-5 have write data: 8496
```

// 以下是jvm 源码中提供的例子

```
 * <pre> {@code
 * class CachedData {
 *   Object data;
 *   volatile boolean cacheValid;
 *   final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
 *
 *   void processCachedData() {
 *     rwl.readLock().lock();
 *     if (!cacheValid) {
 *       // Must release read lock before acquiring write lock
 *       rwl.readLock().unlock();
 *       rwl.writeLock().lock();
 *       try {
 *         // Recheck state because another thread might have
 *         // acquired write lock and changed state before we did.
 *         if (!cacheValid) {
 *           data = ...
 *           cacheValid = true;
 *         }
 *         // Downgrade by acquiring read lock before releasing write lock
 *         rwl.readLock().lock();
 *       } finally {
 *         rwl.writeLock().unlock(); // Unlock write, still hold read
 *       }
 *     }
 *
 *     try {
 *       use(data);
 *     } finally {
 *       rwl.readLock().unlock();
 *     }
 *   }
 * }}</pre>
 *
 * ReentrantReadWriteLocks can be used to improve concurrency in some
 * uses of some kinds of Collections. This is typically worthwhile
 * only when the collections are expected to be large, accessed by
 * more reader threads than writer threads, and entail operations with
 * overhead that outweighs synchronization overhead. For example, here
 * is a class using a TreeMap that is expected to be large and
 * concurrently accessed.
 *
 *  <pre> {@code
 * class RWDictionary {
 *   private final Map<String, Data> m = new TreeMap<String, Data>();
 *   private final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
 *   private final Lock r = rwl.readLock();
 *   private final Lock w = rwl.writeLock();
 *
 *   public Data get(String key) {
 *     r.lock();
 *     try { return m.get(key); }
 *     finally { r.unlock(); }
 *   }
 *   public String[] allKeys() {
 *     r.lock();
 *     try { return m.keySet().toArray(); }
 *     finally { r.unlock(); }
 *   }
 *   public Data put(String key, Data value) {
 *     w.lock();
 *     try { return m.put(key, value); }
 *     finally { w.unlock(); }
 *   }
 *   public void clear() {
 *     w.lock();
 *     try { m.clear(); }
 *     finally { w.unlock(); }
 *   }
 * }}</pre>
```



## jvm 源码

重入锁ReentrantLock是排他锁，排他锁在同一时刻仅有一个线程可以进行访问，但是在大多数场景下，大部分时间都是提供读服务，而写服务占有的时间较少。然而读服务不存在数据竞争问题，如果一个线程在读时禁止其他线程读势必会导致性能降低。所以就提供了读写锁。

读写锁维护着一对锁，一个读锁和一个写锁。通过分离读锁和写锁，使得并发性比一般的排他锁有了较大的提升：在同一时间可以允许多个读线程同时访问，但是在写线程访问时，所有读线程和写线程都会被阻塞。

读写锁的主要特性：

1. 公平性：支持公平性和非公平性。
2. 重入性：支持重入。读写锁最多支持65535个递归写入锁和65535个递归读取锁。
3. 锁降级：遵循获取写锁、获取读锁在释放写锁的次序，写锁能够降级成为读锁

读写锁ReentrantReadWriteLock实现接口ReadWriteLock，该接口维护了一对相关的锁，一个用于只读操作，另一个用于写入操作。只要没有 writer，读取锁可以由多个 reader 线程同时保持。写入锁是独占的。

### 构造器

ReentrantReadWriteLock实现了ReadWriteLock接口，该接口中实现的了

```java
public interface ReadWriteLock {
    //Returns the lock used for reading.
    Lock readLock();
    
    //Returns the lock used for writing.
    Lock writeLock();
}
```

```
//默认构造器，ReentrantReadWriteLock默认是以非公平的方式实现；
public ReentrantReadWriteLock() {
	this(false);
}

public ReentrantReadWriteLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
    readerLock = new ReadLock(this);
    writerLock = new WriteLock(this);
}
```

```
public class ReentrantReadWriteLock
        implements ReadWriteLock{
    //内部类， 读锁
    private final ReentrantReadWriteLock.ReadLock readerLock;
    //内部类，写锁
    private final ReentrantReadWriteLock.WriteLock writerLock;
    //内部实现的同步组件，用来实现所有的同步行为
    final Sync sync;
abstract static class Sync extends AbstractQueuedSynchronizer {
        /**
         * 省略其余源代码
         */
}
public static class WriteLock implements Lock, java.io.Serializable{
	//...
}

public static class ReadLock implements Lock, java.io.Serializable {
	/..
}
```

ReentrantReadWriteLock与ReentrantLock一样，其锁主体依然是Sync，它的读锁、写锁都是依靠Sync来实现的。所以ReentrantReadWriteLock实际上只有一个锁，只是在获取读取锁和写入锁的方式上不一样而已，它的读写锁其实就是两个类：ReadLock、writeLock，这两个类都是lock实现。

### Sync

`abstract static class Sync extends AbstractQueuedSynchronizer `是ReentrantReadWriteLock 的同步实现；

*  它的两个子类`FairSync` 和`NoFairSync` 分别实现的公平锁和非公平锁的功能；
*  WriteLock 和ReadLock同时有依赖同一个Sync对象；


在ReentrantLock中使用一个int类型的state来表示同步状态，该值表示锁被一个线程重复获取的次数。但是读写锁ReentrantReadWriteLock内部维护着两个一对锁，需要用一个变量维护多种状态。所以读写锁采用“按位切割使用”的方式来维护这个变量，将其切分为两部分，高16为表示读，低16为表示写。分割之后，读写锁是如何迅速确定读锁和写锁的状态呢？通过为运算。假如当前同步状态为S，那么写状态等于 S & 0x0000FFFF（将高16位全部抹去），读状态等于S >>> 16(无符号补0右移16位)。代码如下：

```java
static final int SHARED_SHIFT   = 16;
static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;
//左移16位再减1，表示的无符号数是65536， 也就是2^16 -1,二进制表示是16个1
static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;

/** Returns the number of shared holds represented in count  */
// >>> 和 <<< 是java中的无符号移位，空位都补0
// 补充： >> 和 << 是的带符号移位， 对 >>来说，有符号的最高位补1， 无符号的最高位补0
// c >>> 16， 只保留了高位的16位，而这16位（unsigned int)正是shared reader hold count
static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
/** Returns the number of exclusive holds represented in count  */
static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }
```

# 写锁WriteLock

写锁就是一个排它锁

### 写锁的获取

写锁只有在此时没有进程拥有该所的写锁和读锁的时候才能成功获取写锁

如果锁被其他线程获取，则线程会被阻塞知道该锁被释放了

WriteLock的lock方法：

```
public void lock() {
	//调用的AQS的acquire()方法
	sync.acquire(1);
}
```

sync 的acquire()方法：

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

```java
protected final boolean tryAcquire(int acquires) {
    /*
     * Walkthrough:
     * 1. If read count nonzero or write count nonzero
     *    and owner is a different thread, fail.
     * 2. If count would saturate, fail. (This can only
     *    happen if count is already nonzero.)
     * 3. Otherwise, this thread is eligible for lock if
     *    it is either a reentrant acquire or
     *    queue policy allows it. If so, update state
     *    and set owner.
     */
    /**
     * 写锁的lock()方法最终会调用tryAcquire()方法;
     * 对于写锁而言，如果当前已经给获取了锁，需要此时没有其他线程的读锁和写锁
     */
    Thread current = Thread.currentThread();
    //当前锁的个数, 不为0表示当前才能子啊读锁或者写锁
    int c = getState();
    //读锁
    int w = exclusiveCount(c);
    //写锁或者读锁不为0
    if (c != 0) {
        // (Note: if c != 0 and w == 0 then shared count != 0)
        // 当前是线程不是写锁的持有者，返回false
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        //判断是否超所允许获取的锁数量的上限
        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        // Reentrant acquire
        setState(c + acquires);
        return true;
    }
    //writerShouldBlock():写锁是否需要等待（公平锁原则）
    if (writerShouldBlock() ||
        !compareAndSetState(c, c + acquires))
        return false;
    setExclusiveOwnerThread(current);
    return true;
}
```

该方法和ReentrantLock的tryAcquire(int arg)大致一样，在判断重入时增加了一项条件：读锁是否存在。因为要确保写锁的操作对读锁是可见的，如果在存在读锁的情况下允许获取写锁，那么那些已经获取读锁的其他线程可能就无法感知当前写线程的操作。因此只有等读锁完全释放后，写锁才能够被当前线程所获取，一旦写锁获取了，所有其他读、写线程均会被阻塞。

## readLock的获取

```
/**
* readLock的lock()方法调用的是sync的acquireshared()方法，不是sync的排他的acquires()方法
* 这就保证了读锁可以由多个线程同时获取
*/
public void lock() {
	sync.acquireShared(1);
}
```

Sync的acquireShared(int arg)定义在AQS中：

```
public final void acquireShared(int arg) {
	if (tryAcquireShared(arg) < 0)
		doAcquireShared(arg);
}
```

tryAcqurireShared(int arg)尝试获取读同步状态，该方法主要用于获取共享式同步状态，获取成功返回 >= 0的返回结果，否则返回 < 0 的返回结果。

```
    protected final int tryAcquireShared(int unused) {
        //当前线程
        Thread current = Thread.currentThread();
        int c = getState();
        //exclusiveCount(c)计算写锁
        //如果存在写锁，且锁的持有者不是当前线程，直接返回-1
        //存在锁降级问题，后续阐述
        if (exclusiveCount(c) != 0 &&
                getExclusiveOwnerThread() != current)
            return -1;
        //读锁
        int r = sharedCount(c);

        /*
         * readerShouldBlock():读锁是否需要等待（公平锁原则）
         * r < MAX_COUNT：持有线程小于最大数（65535）
         * compareAndSetState(c, c + SHARED_UNIT)：设置读取锁状态
         */
        if (!readerShouldBlock() &&
                r < MAX_COUNT &&
                compareAndSetState(c, c + SHARED_UNIT)) {
            /*
             * holdCount部分后面讲解
             */
            if (r == 0) {
                firstReader = current;
                firstReaderHoldCount = 1;
            } else if (firstReader == current) {
                firstReaderHoldCount++;
            } else {
                HoldCounter rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    cachedHoldCounter = rh = readHolds.get();
                else if (rh.count == 0)
                    readHolds.set(rh);
                rh.count++;
            }
            return 1;
        }
        return fullTryAcquireShared(current);
    }
```

读锁获取的过程相对于独占锁而言会稍微复杂下，整个过程如下：

1. 因为存在锁降级情况，如果存在写锁且锁的持有者不是当前线程则直接返回失败，否则继续
2. 依据公平性原则，判断读锁是否需要阻塞，读锁持有线程数小于最大值（65535），且设置锁状态成功，执行以下代码（对于HoldCounter下面再阐述），并返回1。如果不满足该条件，执行fullTryAcquireShared()。

```
final int fullTryAcquireShared(Thread current) {
    HoldCounter rh = null;
    for (;;) {
        int c = getState();
        if (exclusiveCount(c) != 0) {
            if (getExclusiveOwnerThread() != current)
                return -1;
            // else we hold the exclusive lock; blocking here
            // would cause deadlock.
        } else if (readerShouldBlock()) {
            // Make sure we're not acquiring read lock reentrantly
            if (firstReader == current) {
                // assert firstReaderHoldCount > 0;
            } else {
                if (rh == null) {
                    rh = cachedHoldCounter;
                    if (rh == null || rh.tid != getThreadId(current)) {
                        rh = readHolds.get();
                        if (rh.count == 0)
                            readHolds.remove();
                    }
                }
                if (rh.count == 0)
                    return -1;
            }
        }
        //锁已经达到了最大值
        if (sharedCount(c) == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        //尝试获取读锁
        if (compareAndSetState(c, c + SHARED_UNIT)) {
            if (sharedCount(c) == 0) {
                firstReader = current;
                firstReaderHoldCount = 1;
            } else if (firstReader == current) {
                firstReaderHoldCount++;
            } else {
                if (rh == null)
                    rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    rh = readHolds.get();
                else if (rh.count == 0)
                    readHolds.set(rh);
                rh.count++;
                cachedHoldCounter = rh; // cache for release
            }
            return 1;
        }
    }
}
```

fullTryAcquireShared(Thread current)会根据“是否需要阻塞等待”，“读取锁的共享计数是否超过限制”等等进行处理。如果不需要阻塞等待，并且锁的共享计数没有超过限制，则通过CAS尝试获取锁，并返回1

###readLock的释放

```java
public void unlock() {
	sync.releaseShared(1);
}
```

```java
public final boolean releaseShared(int arg) {
  	//依赖的是tryReleaseShared()
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```

```
/*
 * 释放读锁
 */
protected final boolean tryReleaseShared(int unused) {
    Thread current = Thread.currentThread();
    if (firstReader == current) {
        // assert firstReaderHoldCount > 0;
        if (firstReaderHoldCount == 1)
            firstReader = null;
        else
            firstReaderHoldCount--;
    } else {
        //获取rh对象，并更新“当前线程获取锁的信息”
        HoldCounter rh = cachedHoldCounter;
        if (rh == null || rh.tid != getThreadId(current))
            rh = readHolds.get();
        int count = rh.count;
        if (count <= 1) {
            readHolds.remove();
            if (count <= 0)
                throw unmatchedUnlockException();
        }
        --rh.count;
    }
    for (;;) {
        int c = getState();
        int nextc = c - SHARED_UNIT;
        if (compareAndSetState(c, nextc))
            // Releasing the read lock has no effect on readers,
            // but it may allow waiting writers to proceed if
            // both read and write locks are now free.
            return nextc == 0;
    }
}
```

## HoldCounter

在读锁获取锁和释放锁的过程中，我们一直都可以看到一个变量rh （HoldCounter ），该变量在读锁中扮演着非常重要的作用。

我们了解读锁的内在机制其实就是一个共享锁，为了更好理解HoldCounter ，我们暂且认为它不是一个锁的概率，而相当于一个计数器。一次共享锁的操作就相当于在该计数器的操作。获取共享锁，则该计数器 + 1，释放共享锁，该计数器 - 1。只有当线程获取共享锁后才能对共享锁进行释放、重入操作。所以HoldCounter的作用就是当前线程持有共享锁的数量，这个数量必须要与线程绑定在一起，否则操作其他线程锁就会抛出异常。我们先看HoldCounter的定义：

```
static final class HoldCounter {
	int count = 0;
	final long tid = getThreadId(Thread.currentThread());
}
```

HoldCounter 定义非常简单，就是一个计数器count 和线程 id tid 两个变量。按照这个意思我们看到HoldCounter 是需要和某给线程进行绑定了，我们知道如果要将一个对象和线程绑定仅仅有tid是不够的，而且从上面的代码我们可以看到HoldCounter 仅仅只是记录了tid，根本起不到绑定线程的作用。那么怎么实现呢？答案是ThreadLocal，定义如下：

```
static final class ThreadLocalHoldCounter extends ThreadLocal<HoldCounter> {
	public HoldCounter initialValue() {
		return new HoldCounter();
	}
}
```

通过上面代码HoldCounter就可以与线程进行绑定了。故而，HoldCounter应该就是绑定线程上的一个计数器，而ThradLocalHoldCounter则是线程绑定的ThreadLocal。从上面我们可以看到ThreadLocal将HoldCounter绑定到当前线程上，同时HoldCounter也持有线程Id，这样在释放锁的时候才能知道ReadWriteLock里面缓存的上一个读取线程（cachedHoldCounter）是否是当前线程。这样做的好处是可以减少ThreadLocal.get()的次数，因为这也是一个耗时操作。需要说明的是这样HoldCounter绑定线程id而不绑定线程对象的原因是避免HoldCounter和ThreadLocal互相绑定而GC难以释放它们（尽管GC能够智能的发现这种引用而回收它们，但是这需要一定的代价），所以其实这样做只是为了帮助GC快速回收对象而已。

看到这里我们明白了HoldCounter作用了，我们在看一个获取读锁的代码段：

```
else if (firstReader == current) {
    firstReaderHoldCount++;
} else {
    if (rh == null)
        rh = cachedHoldCounter;
    if (rh == null || rh.tid != getThreadId(current))
        rh = readHolds.get();
    else if (rh.count == 0)
        readHolds.set(rh);
    rh.count++;
    cachedHoldCounter = rh; // cache for release
}
```

这段代码涉及了几个变量：firstReader 、firstReaderHoldCount、cachedHoldCounter 。我们先理清楚这几个变量：

```
private transient Thread firstReader = null;
private transient int firstReaderHoldCount;
private transient HoldCounter cachedHoldCounter;
```

firstReader 看名字就明白了为第一个获取读锁的线程，firstReaderHoldCount为第一个获取读锁的重入数，cachedHoldCounter为HoldCounter的缓存。

理清楚上面所有的变量了，HoldCounter也明白了，我们就来给上面那段代码标明注释，如下：

```
    //如果获取读锁的线程为第一次获取读锁的线程，则firstReaderHoldCount重入数 + 1
    else if (firstReader == current) {
        firstReaderHoldCount++;
    } else {
        //非firstReader计数
        if (rh == null)
            rh = cachedHoldCounter;
        //rh == null 或者 rh.tid != current.getId()，需要获取rh
        if (rh == null || rh.tid != getThreadId(current))
            rh = readHolds.get();
            //加入到readHolds中
        else if (rh.count == 0)
            readHolds.set(rh);
        //计数+1
        rh.count++;
        cachedHoldCounter = rh; // cache for release
    }
```

这里解释下为何要引入firstRead、firstReaderHoldCount。这是为了一个效率问题，firstReader是不会放入到readHolds中的，如果读锁仅有一个的情况下就会避免查找readHolds。

## 锁降级

上开篇是LZ就阐述了读写锁有一个特性就是锁降级，锁降级就意味着写锁是可以降级为读锁的，但是需要遵循先获取写锁、获取读锁在释放写锁的次序。注意如果当前线程先获取写锁，然后释放写锁，再获取读锁这个过程不能称之为锁降级，锁降级一定要遵循那个次序。

在获取读锁的方法tryAcquireShared(int unused)中，有一段代码就是来判读锁降级的：

```java
int c = getState();
//exclusiveCount(c)计算写锁
//如果存在写锁，且锁的持有者不是当前线程，直接返回-1
//存在锁降级问题，后续阐述
if (exclusiveCount(c) != 0 &&
        getExclusiveOwnerThread() != current)
    return -1;
//读锁
int r = sharedCount(c);
```

锁降级中读锁的获取释放为必要？肯定是必要的。试想，假如当前线程A不获取读锁而是直接释放了写锁，这个时候另外一个线程B获取了写锁，那么这个线程B对数据的修改是不会对当前线程A可见的。如果获取了读锁，则线程B在获取写锁过程中判断如果有读锁还没有释放则会被阻塞，只有当前线程A释放读锁后，线程B才会获取写锁成功。

参考：

1.  http://cmsblogs.com/?p=2213
2.  http://www.cnblogs.com/liuling/archive/2013/08/21/2013-8-21-03.html

