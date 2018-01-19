# ReentrantLock

### 构造器

ReentrantLock提供了两个版本的构造器，差别在于实现公平的ReentrantLock还是非公平的ReentrantLock，默认的构造器是非公平方式的；

> 一个可重入的互斥锁定 Lock，它具有与使用 synchronized 方法和语句所访问的隐式监视器锁定相同的一些基本行为和语义，但功能更强大。ReentrantLock 将由最近成功获得锁定，并且还没有释放该锁定的线程所拥有。当锁定没有被另一个线程所拥有时，调用 lock 的线程将成功获取该锁定并返回。如果当前线程已经拥有该锁定，此方法将立即返回。可以使用 isHeldByCurrentThread() 和 getHoldCount() 方法来检查此情况是否发生。

* 其中非公平方式的构造器的吞吐量比较高；
* Fair ReentrantLock 并不是说线程一定可以获取锁，只是说给了他机会获取所（但不一定成功）；
* Sync为ReentrantLock里面的一个内部类，它继承AQS（AbstractQueuedSynchronizer），它有两个子类：公平锁FairSync和非公平锁NonfairSync；ReentrantLock里面大部分的功能都是委托给Sync来实现的，

```
    public ReentrantLock() {
        sync = new NonfairSync();
    }
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```

### Sync

```java
abstract static class Sync extends AbstractQueuedSynchronizer {
        abstract void lock();
        //abatract类中实现的nonfairTryAcquire(int)方法；
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            //只要该锁是没有被锁的状态，就去尝试获取该锁而不判断是否有其他线程已经在等待获取该锁
            //实现的非公平的获取锁是通过直接调用compareAndSetState(0, acquires)来实现的，并没有借用CHL队列
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }

        protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            //非锁的所有者调用此方法会抛出异常
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }

        protected final boolean isHeldExclusively() {
            return getExclusiveOwnerThread() == Thread.currentThread();
        }

        final Thread getOwner() {
            return getState() == 0 ? null : getExclusiveOwnerThread();
        }

        final int getHoldCount() {
            return isHeldExclusively() ? getState() : 0;
        }

        final boolean isLocked() {
            return getState() != 0;
        }
		//该所反序列化之后都是会处于unlocked的状态
        private void readObject(java.io.ObjectInputStream s)
            throws java.io.IOException, ClassNotFoundException {
            s.defaultReadObject();
            setState(0); // reset to unlocked state
        }
    }
```

### FairSync

FaiSync 继承 Sync， 重写了其中的tryAcquire（int acquires）方法，主要实现了以公平的方式来尝试获取锁；

```java
    static final class FairSync extends Sync {
        final void lock() {
            acquire(1);
        }

        /**
         * Fair version of tryAcquire.  Don't grant access unless
         * recursive call or no waiters or is first.
         */
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            //如果c==0，表示没有被锁
            if (c == 0) {	
            //判断是否有其他线程已经在等待获取该锁了
            //实现的公平获取锁是通过在调动compareAndSetState(0, acquires)之前来检查是否hasQueuedPredecessors()来实现的，底层是借用了AQS的CHL队列，在判断队列头部有没有其他线程在等待获取锁
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            //已经被锁，判断当前获得该锁的线程是不是当前线程？
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                //溢出了，锁获取的次数有上限的
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }

    public final boolean hasQueuedPredecessors() {
        // The correctness of this depends on head being initialized
        // before tail and on head.next being accurate if the current
        // thread is first in queue.
        Node t = tail; // Read fields in reverse initialization order
        Node h = head;
        Node s;
        return h != t &&
            ((s = h.next) == null || s.thread != Thread.currentThread());
    }
```

### NonfairSync

```java
    static final class NonfairSync extends Sync {
        /**
         * Performs lock.  Try immediate barge, backing up to normal
         * acquire on failure.
         */
        final void lock() {
          	//非公平锁在这里直接通过CAS尝试获取锁，如果失败就会进一步尝试通过acquire（）来获取锁
            //首先尝试以快速的方式直接修改state值来获取锁，如果失败，就用常规的方式来获取锁
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
          	/* AQS中的acquire方法，内部也是通过tryAcquire来快速的获取锁，如果没有获取到，则将
             * 当前线程加入到CHL队列中, tryAcquire方法由同步组件自己实现
             */
        }

        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }
```

### ReentrantLock的方法

```java
    public void lock() {
        sync.lock();
    }
    public boolean tryLock() {
        return sync.nonfairTryAcquire(1);
    }
    public void unlock() {
        sync.release(1);
    }
    public boolean isLocked() {
        return sync.isLocked();
    }
    protected Thread getOwner() {
        return sync.getOwner();
    }
```







## ReentrantLock与synchronized的区别

前面提到ReentrantLock提供了比synchronized更加灵活和强大的锁机制，那么它的灵活和强大之处在哪里呢？他们之间又有什么相异之处呢？

首先他们肯定具有相同的功能和内存语义。

1. 与synchronized相比，ReentrantLock提供了更多，更加全面的功能，具备更强的扩展性。例如：时间锁等候，可中断锁等候，锁投票。
2. ReentrantLock还提供了条件Condition，对线程的等待、唤醒操作更加详细和灵活，所以在多个条件变量和高度竞争锁的地方，ReentrantLock更加适合（以后会阐述Condition）。
3. ReentrantLock提供了可轮询的锁请求。它会尝试着去获取锁，如果成功则继续，否则可以等到下次运行时处理，而synchronized则一旦进入锁请求要么成功要么阻塞，所以相比synchronized而言，ReentrantLock会不容易产生死锁些。
4. ReentrantLock支持更加灵活的同步代码块，但是使用synchronized时，只能在同一个synchronized块结构中获取和释放。注：ReentrantLock的锁释放一定要在finally中处理，否则可能会产生严重的后果。
5. ReentrantLock支持中断处理，且性能较synchronized会好些。

##补充

### ReentrantLock推荐的使用方式

```
 *  <pre> {@code
 * class X {
 *   private final ReentrantLock lock = new ReentrantLock();
 *   // ...
 *
 *   public void m() {
 *     lock.lock();  // block until condition holds
 *     try {
 *       // ... method body
 *     } finally {
 *       lock.unlock()
 *     }
 *   }
 * }}</pre>
```

