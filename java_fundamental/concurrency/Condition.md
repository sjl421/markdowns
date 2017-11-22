# Condition

在没有Lock之前，我们使用synchronized来控制同步，配合Object的wait()、notify()系列方法可以实现等待/通知模式。在Java SE5后，Java提供了Lock接口，相对于Synchronized而言，Lock提供了条件Condition，对线程的等待、唤醒操作更加详细和灵活。下图是Condition与Object的监视器方法的对比（摘自《Java并发编程的艺术》）：

![201702060001](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/04/201702060001_thumb.jpg)



### Condition接口

1. **await()** ：造成当前线程在接到信号或被中断之前一直处于等待状态。
2. **await(long time, TimeUnit unit) **：造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。
3. **awaitNanos(long nanosTimeout) **：造成当前线程在接到信号、被中断或到达指定等待时间之前一直处于等待状态。返回值表示剩余时间，如果在nanosTimesout之前唤醒，那么返回值 = nanosTimeout - 消耗时间，如果返回值 <= 0 ,则可以认定它已经超时了。
4. **awaitUninterruptibly() **：造成当前线程在接到信号之前一直处于等待状态。【注意：该方法对中断不敏感】。
5. **awaitUntil(Date deadline) **：造成当前线程在接到信号、被中断或到达指定最后期限之前一直处于等待状态。如果没有到指定时间就被通知，则返回true，否则表示到了指定时间，返回返回false。
6. **signal()**：唤醒一个等待线程。该线程从等待方法返回前必须获得与Condition相关的锁。
7. **signalAll()**：唤醒所有等待线程。能够从等待方法返回的线程必须获得与Condition相关的锁。

Condition是一种广义上的条件队列。他为线程提供了一种更为灵活的等待/通知模式，线程在调用await方法后执行挂起操作，直到线程等待的某个条件为真时才会被唤醒。Condition必须要配合锁一起使用，因为对共享状态变量的访问发生在多线程环境下。一个Condition的实例必须与一个Lock绑定，因此Condition一般都是作为Lock的内部实现。

### Condition示例代码

```
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class BoundedQueue<T> {
    private Object[] items;
    private int addIndex, removeIndex, count;
    private Lock lock = new ReentrantLock();

    private Condition notEmpty = lock.newCondition();
    private Condition notFull = lock.newCondition();

    public BoundedQueue(int size) {
        items = new Object[size];
    }

    public void add(T t) throws InterruptedException {
    	//在调用condition的方法是先获取锁
        lock.lock();
        try {
            //注意，这里用的是while而不是if
            while (count == items.length) {
                notFull.await();
            }

            items[addIndex] = t;
            if (++addIndex == items.length) {
                addIndex = 0;
            }
            ++count;
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }

    public T remove() throws InterruptedException {
        lock.lock();
        T rt;
        try {
            //注意，这里用的是while而不是if
            while (count == 0) {
                notEmpty.await();
            }
            rt = (T) items[removeIndex];
            if (removeIndex == items.length) {
                removeIndex = 0;
            }
            --count;
            notFull.signal();
            return rt;
        } finally {
            lock.unlock();
        }
    }

    static class Worker implements Runnable {
        private BoundedQueue<Integer> queue;
        private String name;
        //0:add, 1:remove
        private int type;

        public Worker(BoundedQueue<Integer> queue, String name, int type) {
            this.queue = queue;
            this.name = name;
            this.type = type;
        }

        public void run() {
            System.out.println(String.format("%s is running", name));
            try {
                for (int i = 0; i < 10; i++) {
                    if (type == 0) {
                        queue.add(i);
                        System.out.println(String.format("Thread add %d", i));
                        Thread.sleep(500);
                    } else {
                        int data = queue.remove();
                        System.out.println(String.format("Thread remove %d:%d", i, data));
                        Thread.sleep(1000);
                    }
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        BoundedQueue<Integer> queue = new BoundedQueue<Integer>(2);
        Worker addWorker = new Worker(queue, "addWorker", 0);
        Worker removeWorker = new Worker(queue, "removeWorker", 1);
        new Thread(addWorker).start();
        new Thread(removeWorker).start();
    }
}
```

### ConditionObject

ConditionObject 实现了 Condition接口，Condition接口中定义了和Condition操作相关的对象；

每个Condition对象都包含一个FIFO队列（ReentrantLock中也是用一个FIFO队列实现同步）， 该队列是Condition对象通知/等待功能的关键，在每一个节点都包含了对一个线程的引用，这些线程就是在该condition对象上等待的线。

一个lock可以new多个condition对象出来，每一个condition对象都会有一个等待队列；

```
public class ConditionObject implements Condition, java.io.Serializable {    
    //FIFO队列头节点
    private transient Node firstWaiter;
    //FIFO队列尾节点
    private transient Node lastWaiter;

    public ConditionObject() {
    }
    
    /** 省略方法 **/
}
```

ConditionObject拥有首节点（firstWaiter）和尾节点（lastWaiter）。 当线程调用await()方法时，将会以当前线程构成一个节点（Node),并将节点加入到该队列的尾部。结构如下：

![201702060002](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/04/201702060002_thumb.jpg)

ConditionObject的FIFO对列的节点Node与AQS的CLH同步队列的节点使用的都是同一个类（AbstractQueuedSynchronized.Node静态内部类).

### 等待-await()

```
public final void await() throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    Node node = addConditionWaiter();
    //该线程加入等待队列后释放该线程锁获取的锁
    int savedState = fullyRelease(node);
    int interruptMode = 0;
    /**
    * 检测此节点的线程是否在同步队上，如果不在，则说明该线程还不具备竞争锁的资格，则继续等待
    * 直到检测到此节点在同步队列
    */
    while (!isOnSyncQueue(node)) {
       	//线程挂起
        LockSupport.park(this);
        //在线程等待的过程中检查线程是否被中断了,如果中断，则退出等待
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    //竞争同步状态
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null) // clean up if cancelled
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}
```

###addConditionWaiter()

```
private Node addConditionWaiter() {
    Node t = lastWaiter;
    // If lastWaiter is cancelled, clean out.
    // 如果尾节点被cancelled了，将尾节点移除
    // 其实unlinkCancelledWaiters()方法做了更多的事情
    if (t != null && t.waitStatus != Node.CONDITION) {
        unlinkCancelledWaiters();
        t = lastWaiter;
    }
    //将waitStatus设置为CONDITION，表示当前节点（线程）是在等待Condition
    Node node = new Node(Thread.currentThread(), Node.CONDITION);
    if (t == null)
        firstWaiter = node;
    else
        t.nextWaiter = node;
    lastWaiter = node;
    return node;
}
```

### unlinkCancelledWaiters()

unlinkCancelledWaiters() 方法会遍历整个等待队列，将队列中在等待时间内被cancelled掉的节点移除掉；

1. unlinkCancelledWaiters() 只会在获取锁的情况下执行；
2. 该方法只有在wait condition的过程中发生了cancellation 并且是在往同步队列中插入新的等待节点时发现lastWaiter节点被cancelled了才会被调用；
3. 该方法是遍历整个队列；

```
/**
 * Unlinks cancelled waiter nodes from condition queue.
 * Called only while holding lock. This is called when
 * cancellation occurred during condition wait, and upon
 * insertion of a new waiter when lastWaiter is seen to have
 * been cancelled. This method is needed to avoid garbage
 * retention in the absence of signals. So even though it may
 * require a full traversal, it comes into play only when
 * timeouts or cancellations occur in the absence of
 * signals. It traverses all nodes rather than stopping at a
 * particular target to unlink all pointers to garbage nodes
 * without requiring many re-traversals during cancellation
 * storms.
 */
/**
 * 遍历队列，将队列中的Cancelled掉的节点从队列中移除掉
 * 其中，t 代表当前检查的节点，trail 记录的是 t 的前一个节点， next 是 t 的后继节点
 * if 判断语句会判断 当前节点t的waitStatus是否 == CONDITION:
 * 1. 如果等，则继续检查t的下一个节点
 * 2. 如果不等，则通过将trial.nextWaiter = next来将当前节点 t 从对了中移除
 * 同时还需要不停的检查 if(next == null)来判断队列是否结束，同时更新lastWaiter节点；
 */
private void unlinkCancelledWaiters() {
    Node t = firstWaiter;
    Node trail = null;
    while (t != null) {
        Node next = t.nextWaiter;
        if (t.waitStatus != Node.CONDITION) {
            t.nextWaiter = null;
            if (trail == null)
                firstWaiter = next;
            else
                trail.nextWaiter = next;
            if (next == null)
                lastWaiter = trail;
        }
        else
            trail = t;
        t = next;
    }
}
```

### 通知-Signal()

调用Condition的Singnal()方法，将会唤醒在等待队列中等待时间最长时间的节点（队列的首节点）， 在唤醒节点之前，会将节点移动到CLH同步队列中；

```
    public final void signal() {
        //该方法首先会判断当前线程是否已经获得了锁，这是前置条件。然后唤醒条件队列中的头节点。
        if (!isHeldExclusively())
            throw new IllegalMonitorStateException();
        Node first = firstWaiter;
        if (first != null)
            doSignal(first);
    }
```

1. 该方法首先会判断自己是否已经获得了锁（只有获得了锁的线程才可以调用signal方法）。 然后调用doSignal()方法唤醒队列中的头节点。

####doSignal(Node first)

doSignal(Node first)主要是做两件事：

1. 修改头节点.
2. 调用transferForSignal(Node first) 方法将节点移动到CLH同步队列中。

```
* Removes and transfers nodes until hit non-cancelled one or
* null. Split out from signal in part to encourage compilers
* to inline the case of no waiters.
private void doSignal(Node first) {
    do {
        if ( (firstWaiter = first.nextWaiter) == null)
            lastWaiter = null;
        first.nextWaiter = null;
    } while (!transferForSignal(first) &&
             (first = firstWaiter) != null);
}
```

transferForSignal(Node first)源码如下：

```
final boolean transferForSignal(Node node) {
    /*
     * If cannot change waitStatus, the node has been cancelled.
     */
    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
        return false;

    /*
     * Splice onto queue and try to set waitStatus of predecessor to
     * indicate that thread is (probably) waiting. If cancelled or
     * attempt to set waitStatus fails, wake up to resync (in which
     * case the waitStatus can be transiently and harmlessly wrong).
     */
    //将节点加入到syn队列中去，返回的是syn队列中node节点前面的一个节点
    Node p = enq(node);
    int ws = p.waitStatus;
    //如果它的前一个结点p的状态为cancel 或者修改waitStatus失败，则直接唤醒该这个节点的线程让他resync
    if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
        LockSupport.unpark(node.thread);
    return true;
}
```

整个通知的流程如下：

1. 判断当前线程是否已经获取了锁，如果没有获取则直接抛出异常，因为获取锁为通知的前置条件。
2. 如果线程已经获取了锁，则将唤醒条件队列的首节点
3. 唤醒首节点是先将条件队列中的头节点移出，然后调用AQS的enq(Node node)方法将其安全地移到CLH同步队列中
4. 最后判断如果该节点的同步状态是否为Cancel，或者修改状态为Signal失败时，则直接调用LockSupport唤醒该节点的线程。

####总结

一个线程获取锁后，通过调用Condition的await()方法，会将当前线程先加入到条件队列中，然后释放锁，最后通过isOnSyncQueue(Node node)方法不断自检看节点是否已经在CLH同步队列了，如果是则尝试获取锁，否则一直挂起。当线程调用signal()方法后，程序首先检查当前线程是否获取了锁，然后通过doSignal(Node first)方法唤醒CLH同步队列的首节点。被唤醒的线程，将从await()方法中的while循环中退出来，然后调用acquireQueued()方法竞争同步状态。

其他其中await()的方式都是在await()的基础上稍微加了一些变形得到的；



参考文献：

1. http://cmsblogs.com/?p=2222