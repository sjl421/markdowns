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
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
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

