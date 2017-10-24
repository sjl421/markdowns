#CyclicBarrier

它允许一组线程互相等待，直到到达某个公共屏障点 (common barrier point)。在涉及一组固定大小的线程的程序中，这些线程必须不时地互相等待，此时 CyclicBarrier 很有用。因为该 barrier 在释放等待线程后可以重用，所以称它为循环 的 barrier。

通俗点讲就是：让一组线程到达一个屏障时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。

#### 实现分析

CyclicBarrier的结构如下：

![201702100001](http://cmsblogs.qiniudn.com/wp-content/uploads/2017/04/201702100001_thumb.jpg)

CyclicBarrier内部使用了重复锁ReentrantLock和Condition。

构造函数：

```
public CyclicBarrier(int parties) {
    this(parties, null);
}

//定义了barrierAction，当所有线程都到达屏障时执行barrierAction
public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}
```

成员变量：

```java
private static class Generation {
    boolean broken = false;
}

/** The lock for guarding barrier entry */
private final ReentrantLock lock = new ReentrantLock();
/** Condition to wait on until tripped */
private final Condition trip = lock.newCondition();
/** The number of parties */
private final int parties;
/* The command to run when tripped */
private final Runnable barrierCommand;
/** The current generation */
private Generation generation = new Generation();

/**
 * Number of parties still waiting. Counts down from parties to 0
 * on each generation.  It is reset to parties on each new
 * generation or when broken.
 */
private int count;
```

### 等待-await()

await()方法是线程用来调用进入等待的方法，只是简单的调用了dowait()方法；

关于线程从await()中退出等待有多种的policy，而实现这些policy均是在dowait()中实现的；

1. CyclicBarrier的等待的所有线程都达到(最后一个线程到达)
2. 其他线程中断了当前线程（感觉当前线程是创建CyclicBarrier和其他子线程的主线程）
3. 已经在CyclicBarrier中等待的线程被其它线程中断了；
4. 在CyclicBarrier等待的线程超时；
5. 有其他线程在CyclicBarrier上调用了reset方法；

```
public int await() throws InterruptedException, BrokenBarrierException {
    try {
        return dowait(false, 0L);
    } catch (TimeoutException toe) {
        throw new Error(toe); // cannot happen
    }
}
```

```
private int dowait(boolean timed, long nanos)
    throws InterruptedException, BrokenBarrierException,
           TimeoutException {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        final Generation g = generation;

        if (g.broken)
            throw new BrokenBarrierException();

        if (Thread.interrupted()) {
            breakBarrier();
            throw new InterruptedException();
        }

        int index = --count;
        //最后一个线程到来
        if (index == 0) {  // tripped
            boolean ranAction = false;
            try {
                /** 如果CyclicBarrier设置了barrierCommand, 则有最后一个到达barrier的线程来
                 * 执行barrierCommand方法；
                 * 这里执行barrierCommand方式是调用的command.run()方法，并没有单独起一个线程
                 * 来执行
                 */
                final Runnable command = barrierCommand;
                if (command != null)
                    command.run();
                ranAction = true;
                //唤醒所有线程并更新Generation
                nextGeneration();
                return 0;
            } finally {
                //执行barrierCommand出错,标记generation.broken位，并唤醒所有线程
                if (!ranAction)
                    breakBarrier();
            }
        }

        // loop until tripped, broken, interrupted, or timed out
        /**
         * 实现了等待，并检测是否broke，interrupted，超时发生;
         * 如果有的话就做相应的处理
         */
        for (;;) {
            try {
                if (!timed)
                    trip.await();
                else if (nanos > 0L)
                    nanos = trip.awaitNanos(nanos);
            } catch (InterruptedException ie) {
                if (g == generation && ! g.broken) {
                    breakBarrier();
                    throw ie;
                } else {
                    // We're about to finish waiting even if we had not
                    // been interrupted, so this interrupt is deemed to
                    // "belong" to subsequent execution.
                    Thread.currentThread().interrupt();
                }
            }

            if (g.broken)
                throw new BrokenBarrierException();
                
            //generation已经更新，返回index
            if (g != generation)
                return index;
                
            //“超时等待”，并且时间已到,终止CyclicBarrier，并抛出异常
            if (timed && nanos <= 0L) {
                breakBarrier();
                throw new TimeoutException();
            }
        }
    } finally {
        lock.unlock();
    }
}
```

在上面的源代码中，我们可能需要注意Generation 对象，在上述代码中我们总是可以看到抛出BrokenBarrierException异常，那么什么时候抛出异常呢？如果一个线程处于等待状态时，如果其他线程调用reset()，或者调用的barrier原本就是被损坏的，则抛出BrokenBarrierException异常。同时，任何线程在等待时被中断了，则其他所有线程都将抛出BrokenBarrierException异常，并将barrier置于损坏状态。

​	同时，Generation描述着CyclicBarrier的更显换代。在CyclicBarrier中，同一批线程属于同一代。当有parties个线程到达barrier，generation就会被更新换代。其中broken标识该当前CyclicBarrier是否已经处于中断状态。

```
    private static class Generation {
        boolean broken = false;
    }
```

默认barrier是没有损坏的。

当barrier损坏了或者有一个线程中断了，则通过breakBarrier()来终止所有的线程：

```
    private void breakBarrier() {
        generation.broken = true;
        count = parties;
        trip.signalAll();
    }
```

在breakBarrier()中除了将broken设置为true，还会调用signalAll将在CyclicBarrier处于等待状态的线程全部唤醒。

当所有线程都已经到达barrier处（index == 0），则会通过nextGeneration()进行更新换地操作，在这个步骤中，做了三件事：唤醒所有线程，重置count，generation。

```
    private void nextGeneration() {
        trip.signalAll();
        count = parties;
        generation = new Generation();
    }
```

CyclicBarrier同时也提供了await(long timeout, TimeUnit unit) 方法来做超时控制，内部还是通过调用doawait()实现的。

###应用示例：

```java
import java.util.Random;
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

class Worker implements Runnable {
    private CyclicBarrier cb;
    private int id;

    public Worker(int id, CyclicBarrier cb) {
        this.id = id;
        this.cb = cb;
    }

    public void run() {
        Random random = new Random(System.currentTimeMillis());
        while (true) {
            try {
                Thread.sleep(random.nextInt(1000));
                System.out.println(String.format("%d is comming...", id));
                cb.await();
                System.out.println(String.format("%d is exiting...", id));
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }
}

public class TestCyclicBarrier {
    public static void main(String[] args) {
        int num = 5;
        CyclicBarrier cb = new CyclicBarrier(num, new Runnable() {
            public void run() {
                System.out.println("All thread has arrived...");
            }
        });
        for (int i = 0; i < 5; i++) {
            new Thread(new Worker(i, cb)).start();
        }
    }
}
```



CyclicBarrier试用与多线程结果合并的操作，用于多线程计算数据，最后合并计算结果的应用场景。比如我们需要统计多个Excel中的数据，然后等到一个总结果。我们可以通过多线程处理每一个Excel，执行完成后得到相应的结果，最后通过barrierAction来计算这些线程的计算结果，得到所有Excel的总和。