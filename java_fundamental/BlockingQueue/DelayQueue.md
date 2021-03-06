# DelayQueue

##介绍

DelayQueue是一个支持延时获取元素的无界阻塞队列。里面的元素全部都是“可延期”的元素，列头的元素是最先“到期”的元素，如果队列里面没有元素到期，是不能从列头获取元素的，哪怕有元素也不行。也就是说只有在延迟期到时才能够从队列中取元素。如果当前已经有线程在等待队列中的元素了，那么其他的线程会等待，等到该线程从队列中取出元素之后才被唤醒开始尝试从队列中获取元素。

DelayQueue主要用于两个方面：
\- 缓存：清掉缓存中超时的缓存数据
\- 任务超时处理

###DelayQueue

DelayQueue实现的关键主要有如下几个：

1. 可重入锁ReentrantLock
2. 用于阻塞和通知的Condition对象
3. 根据Delay时间排序的优先级队列：PriorityQueue
4. 用于优化阻塞通知的线程元素leaders

###Delayed接口

DelayQueue中的元素都需要实现Delayed接口，该接口用来返回当前元素的过期时间还剩多少，如果getDelay()方法返回的值 <= 0，则标识该元素已经过期了，可以从队列中取出了；

同时，由于**DelayQueue中队列实现使用PriorityQueue实现的**，所以Delayed也继承了Comparable接口，这样队列中的元素按照最早过期的元素来排列，最早过期的元素会排在前面；

```
public interface Delayed extends Comparable<Delayed> {
    long getDelay(TimeUnit unit);
}
```

如何使用该接口呢？上面说的非常清楚了，实现该接口的getDelay()方法，同时定义compareTo()方法即可。

### 内部实现

**成员变量**

```
  public class DelayQueue<E extends Delayed> extends AbstractQueue<E>
          implements BlockingQueue<E> {
      /** 可重入锁 */
      private final transient ReentrantLock lock = new ReentrantLock();
      /** 支持优先级的BlockingQueue */
      private final PriorityQueue<E> q = new PriorityQueue<E>();
      /** 用于优化阻塞 */
      private Thread leader = null;
      /** Condition */
      private final Condition available = lock.newCondition();

      /**
       * 省略很多代码
       */
  }	
```

看了DelayQueue的内部结构就对上面几个关键点一目了然了，但是这里有一点需要注意，DelayQueue的元素都必须继承Delayed接口。同时也可以从这里初步理清楚DelayQueue内部实现的机制了：以支持优先级无界队列的PriorityQueue作为一个容器，容器里面的元素都应该实现Delayed接口，在每次往优先级队列中添加元素时以元素的过期时间作为排序条件，最先过期的元素放在优先级最高。

### offer()

```
    public boolean offer(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            // 向 PriorityQueue中插入元素
            q.offer(e);
            // 如果当前元素的对首元素（优先级最高），leader设置为空，唤醒所有等待线程
            if (q.peek() == e) {
                leader = null;
                available.signal();
            }
            // 无界队列，永远返回true
            return true;
        } finally {
            lock.unlock();
        }
    }
```

offer(E e)就是往PriorityQueue中添加元素。整个过程还是比较简单，但是在判断当前元素是否为对首元素，如果是的话则设置leader=null，这是非常关键的一个步骤，后面阐述。

### take()

```
    public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            for (;;) {
                // 对首元素
                E first = q.peek();
                // 对首为空，阻塞，等待off()操作唤醒
                if (first == null)
                    available.await();
                else {
                    // 获取对首元素的超时时间
                    long delay = first.getDelay(NANOSECONDS);
                    // <=0 表示已过期，出对，return
                    if (delay <= 0)
                        return q.poll();
                    first = null; // don't retain ref while waiting
                    // leader != null 证明有其他线程在操作，阻塞
                    if (leader != null)
                        available.await();
                    else {
                        // 否则将leader 设置为当前线程，独占
                        Thread thisThread = Thread.currentThread();
                        leader = thisThread;
                        try {
                            // 超时阻塞
                            available.awaitNanos(delay);
                        } finally {
                            // 释放leader
                            if (leader == thisThread)
                                leader = null;
                        }
                    }
                }
            }
        } finally {
            // 唤醒阻塞线程
            if (leader == null && q.peek() != null)
                available.signal();
            lock.unlock();
        }
    }

```

首先是获取对首元素，如果对首元素的延时时间 delay <= 0 ，则可以出对了，直接return即可。否则设置first = null，这里设置为null的主要目的是为了避免内存泄漏。如果 leader != null 则表示当前有线程占用，则阻塞，否则设置leader为当前线程，然后调用awaitNanos()方法超时等待。

**first = null**

这里为什么如果不设置first = null，则会引起内存泄漏呢？线程A到达，列首元素没有到期，设置leader = 线程A，这是线程B来了因为leader != null，则会阻塞，线程C一样。假如线程阻塞完毕了，获取列首元素成功，出列。这个时候列首元素应该会被回收掉，但是问题是它还被线程B、线程C持有着，所以不会回收，这里只有两个线程，如果有线程D、线程E...呢？这样会无限期的不能回收，就会造成内存泄漏。

这个入队、出对过程和其他的阻塞队列没有很大区别，无非是在出对的时候增加了一个到期时间的判断。同时通过leader来减少不必要阻塞。



（原文地址：http://cmsblogs.com/?p=2413）

推荐阅读: https://www.cnblogs.com/sunzhenchao/p/3515085.html 