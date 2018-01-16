# J.U.C之AQS：阻塞和唤醒线程

> 此篇博客所有源码均来自JDK 1.8

在线程获取同步状态时如果获取失败，则加入CLH同步队列，通过通过自旋的方式不断获取同步状态，但是在自旋的过程中则需要判断当前线程是否需要阻塞，其主要方法在acquireQueued()：

```
if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
```

通过这段代码我们可以看到，在获取同步状态失败后，线程并不是立马进行阻塞，需要检查该线程的状态，检查状态的方法为 shouldParkAfterFailedAcquire(Node pred, Node node) 方法，该方法主要靠前驱节点判断当前线程是否应该被阻塞，代码如下：

```
    private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        //前驱节点
        int ws = pred.waitStatus;
        /* 前置节点状态是signal，那当前节点可以安全阻塞，因为前置节点承诺执行完之后会通知唤醒当前
* 节点
*/
        if (ws == Node.SIGNAL)
            return true;
            /*
            * Predecessor was cancelled. Skip over predecessors and
            * indicate retry.
            */            
        // 前置节点如果已经被取消了，则一直往前遍历直到前置节点不是取消状态，与此同时会修改链表关系
        if (ws > 0) {
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } 
        //前驱节点状态为Condition、propagate
        else {
            /*
             * waitStatus must be 0 or PROPAGATE.  Indicate that we
             * need a signal, but don't park yet.  Caller will need to
             * retry to make sure it cannot acquire before parking.
             */
// 前置节点是0或者propagate状态，这里通过CAS把前置节点状态改成signal
// 这里不返回true让当前节点阻塞，而是返回false，目的是让调用者再check一下当前线程是否能
// 成功获取锁，失败的话再阻塞，这里说实话我也不是特别理解这么做的原因        
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }
```

这段代码主要检查当前线程是否需要被阻塞，具体规则如下：

1. 如果当前线程的前驱节点状态为SINNAL，则表明当前线程需要被阻塞，调用unpark()方法唤醒，直接返回true，当前线程阻塞
2. 如果当前线程的前驱节点状态为CANCELLED（ws > 0），则表明该线程的前驱节点已经等待超时或者被中断了，则需要从CLH队列中将该前驱节点删除掉，直到回溯到前驱节点状态 <= 0 ，返回false
3. 如果前驱节点非SINNAL，非CANCELLED，则通过CAS的方式将其前驱节点设置为SINNAL，返回false

如果 shouldParkAfterFailedAcquire(Node pred, Node node) 方法返回true，则调用parkAndCheckInterrupt()方法阻塞当前线程：

```
    private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);
        return Thread.interrupted();
    }
```

parkAndCheckInterrupt() 方法主要是把当前线程挂起，从而阻塞住线程的调用栈，同时返回当前线程的中断状态。其内部则是调用LockSupport工具类的park()方法来阻塞该方法。

当线程释放同步状态后，则需要唤醒该线程的后继节点：

```
    public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
				//唤醒后继节点
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
```

我们先看tryRelease方法,tryRelease()方法也是有具体的同步组件来实现：

```
protected final boolean tryRelease(int releases) {
// 释放后c的状态值
            int c = getState() - releases;
// 如果持有锁的线程不是当前线程，直接抛出异常
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
// 如果c==0，说明所有持有锁都释放完了，其他线程可以请求获取锁
                free = true;
                setExclusiveOwnerThread(null);
            }
// 这里只会有一个线程执行到这，不存在竞争，因此不需要CAS
            setState(c);
            return free;
        }
```

调用unparkSuccessor(Node node)唤醒后继节点：

```
private void unparkSuccessor(Node node) {
        /*
         * If status is negative (i.e., possibly needing signal) try
         * to clear in anticipation of signalling.  It is OK if this
         * fails or if status is changed by waiting thread.
         */
        int ws = node.waitStatus;
        if (ws < 0)
/*
如果状态小于0，把状态改成0，0是空的状态，因为node这个节点的线程释放了锁后续不需要做任何
操作，不需要这个标志位，即便CAS修改失败了也没关系，其实这里如果只是对于锁来说根本不需要CAS，因为这个方法只会被释放锁的线程访问，只不过unparkSuccessor这个方法是AQS里的方法就必须考虑到多个线程同时访问的情况（可能共享锁或者信号量这种）
*/
            compareAndSetWaitStatus(node, ws, 0);

        /*
         * Thread to unpark is held in successor, which is normally
         * just the next node.  But if cancelled or apparently null,
         * traverse backwards from tail to find the actual
         * non-cancelled successor.
         */
        Node s = node.next;
// 这段代码的作用是如果下一个节点为空或者下一个节点的状态>0（目前大于0就是取消状态）
// 则从tail节点开始遍历找到离当前节点最近的且waitStatus<=0（即非取消状态）的节点并唤醒
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            LockSupport.unpark(s.thread);
    }    
```

可能会存在当前线程的后继节点为null，超时、被中断的情况，如果遇到这种情况了，则需要跳过该节点，但是为何是从tail尾节点开始，而不是从node.next开始呢？原因在于node.next仍然可能会存在null或者取消了，所以采用tail回溯办法找第一个可用的线程。最后调用LockSupport的unpark(Thread thread)方法唤醒该线程。

## LockSupport

从上面我可以看到，当需要阻塞或者唤醒一个线程的时候，AQS都是使用LockSupport这个工具类来完成的。

> LockSupport是用来创建锁和其他同步类的基本线程阻塞原语

每个使用LockSupport的线程都会与一个许可关联，如果该许可可用，并且可在进程中使用，则调用park()将会立即返回，否则可能阻塞。如果许可尚不可用，则可以调用 unpark 使其可用。但是注意许可不可重入，也就是说只能调用一次park()方法，否则会一直阻塞。

LockSupport定义了一系列以park开头的方法来阻塞当前线程，unpark(Thread thread)方法来唤醒一个被阻塞的线程。如下：

[![201701310001](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/03/201701310001_thumb.jpg)](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/03/201701310001.jpg)

park(Object blocker)方法的blocker参数，主要是用来标识当前线程在等待的对象，该对象主要用于问题排查和系统监控。

park方法和unpark(Thread thread)都是成对出现的，同时unpark必须要在park执行之后执行，当然并不是说没有不调用unpark线程就会一直阻塞，park有一个方法，它带了时间戳（parkNanos(long nanos)：为了线程调度禁用当前线程，最多等待指定的等待时间，除非许可可用）。

park()方法的源码如下：

```
    public static void park() {
        UNSAFE.park(false, 0L);
    }
```

unpark(Thread thread)方法源码如下：

```
    public static void unpark(Thread thread) {
        if (thread != null)
            UNSAFE.unpark(thread);
    }
```

从上面可以看出，其内部的实现都是通过UNSAFE（sun.misc.Unsafe UNSAFE）来实现的，其定义如下：

```
public native void park(boolean var1, long var2);
public native void unpark(Object var1);
```

两个都是native本地方法。Unsafe 是一个比较危险的类，主要是用于执行低级别、不安全的方法集合。尽管这个类和所有的方法都是公开的（public），但是这个类的使用仍然受限，你无法在自己的java程序中直接使用该类，因为只有授信的代码才能获得该类的实例。

## 参考资料

1. 方腾飞：《Java并发编程的艺术》
2. [LockSupport的park和unpark的基本使用,以及对线程中断的响应性](http://www.tuicool.com/articles/MveUNzF)

（原文地址：http://cmsblogs.com/?p=2205）

参考：

1. http://ifeve.com/juc-aqs-reentrantlock/