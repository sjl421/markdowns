# Java 线程的取消机制

[TOC]

## 1. 任务中设置中断标志位,并在任务中循环的检查这个标志位

使用volatile类型的域来保存取消状态,然后线程中的每一次while循环都先去检查这个取消状态是否已经被置位了;

```java
public class PrimeGenerator implements Runnable {
    private final List<BigInteger> primes = new ArrayList<BigInteger>();
    private volatile boolean cancelled;  //should be volatile type

    @Override
    public void run() {
        BigInteger p = BigInteger.ONE;
        while(!cancelled) {
            p  = p.nextProbablePrime();
            synchronized (this) {
                primes.add(p);
            }
        }
    }

    public void cancel() {
        cancelled = true;
    }
    public synchronized List<BigInteger> get() {
        return new ArrayList<BigInteger>(primes);
    }
  
  	List<BigInteger> aSecondOfPrimes() throws InterruptedException {
        PrimeGenerator generator = new PrimeGenerator();
        new Thread(generator).start();

        try {
            TimeUnit.MILLISECONDS.sleep(500);
        } finally {
            generator.cancel();
        }
        return generator.get();
    }
 }
```

## 2. Thread.currentThread.isInterrupted()

java Thread类中提供的中断方法, 通过线程自身的中断方法,我们可以控制线程自己的声明周期(这和通过使用ExecutorService是不同的)

```java
public class Thread{
  public void interrupt();
  public boolean isInterrupted();
  public static boolean interrupted();
}
```

```java
public class BrokenPrimeProducerByInterrupt extends Thread {
    private final BlockingQueue<BigInteger> queue;

    BrokenPrimeProducerByInterrupt(BlockingQueue<BigInteger> queue) {
        this.queue = queue;
    }

    @Override
    public void run() {
        try {
            BigInteger p = BigInteger.ONE;
          //代码中使用下面两种检查线程是否中断的方式都是可以的, 由于是在run()方法内,都可以表示检查当前线程是否中断;
            while (!Thread.currentThread().isInterrupted()) {
//            while(!isInterrupted()) {
                System.out.println("run in while " + this);
                queue.put(p = p.nextProbablePrime());
                System.out.println("put: " + p);
            }
        } catch (InterruptedException e) {
            System.out.println(e);
        }
    }

    public void cancel() {
        System.out.println(this);
        interrupt();		//直接调用interrupt()方法也是可以的,相当于调用的是this.interrupt();
    }

    public void cancel2() {
        Thread currentThread = Thread.currentThread();
      	System.out.println(this);  //这里输出的是 [Thread-0,5,main]  
      	System.out.println(currentThread);	//这里输出的是 Thread[main,5,main]
        interrupt(); //能够正常的停止, 这里调用interrupt就相当于调用 this.interrupt();
//        Thread.currentThread().interrupt();	//不能够正常的终止线程, 通过上面打印的信息可以看出,在main方法中调用cancel2() 时, Thread.currentThread 得到的是main函数的进程信息,所以如果这里通过Thread.currentThread().interrupt()并不能正确的将子线程正常中断;
    }

    public static void main(String[] args) throws InterruptedException {
        System.out.println("Main: " + Thread.currentThread());
        BlockingQueue<BigInteger> queue = new ArrayBlockingQueue<BigInteger>(10);
        BrokenPrimeProducerByInterrupt producer = new BrokenPrimeProducerByInterrupt(queue);
        producer.start();
//        new Thread(producer).start();
        TimeUnit.MILLISECONDS.sleep(1000);
//        producer.cancel();
        producer.cancel2();
    }
}
```

>`Java 并发编程实战`中提到过:  中断是线程取消最合理的方式;

## 3.通过使用ExecutorService来管理线程的生命周期;

java中提供了线程池的方式来为我们提供创建线程和对线程声明周期的管理能力,ExecutorService中的`shutdown()` 和`shundownnow()` 连个方法为我们提供提供了取消线程的方法,其具体的含义为:

```
我们可以通过调用线程池的shutdown或shutdownNow方法来关闭线程池，它们的原理是遍历线程池中的工作线程，然后逐个调用线程的interrupt方法来中断线程，所以无法响应中断的任务可能永远无法终止。但是它们存在一定的区别，shutdownNow首先将线程池的状态设置成STOP，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表，而shutdown只是将线程池的状态设置成SHUTDOWN状态，然后中断所有没有正在执行任务的线程, 已经开始执行的任务将会等到线程执行完毕正常退出;

只要调用了这两个关闭方法的其中一个，isShutdown方法就会返回true。当所有的任务都已关闭后,才表示线程池关闭成功，这时调用isTerminaed方法会返回true。至于我们应该调用哪一种方法来关闭线程池，应该由提交到线程池的任务特性决定，通常调用shutdown来关闭线程池，如果任务不一定要执行完，则可以调用shutdownNow。
```

## 4. Future, FutureTask

Future接口和FutureTask提供了取消线程的方法:

```java
public interface Callable<V> {
  V call() throws Exception;
}

public interface Future<V> {
  boolean cancel(boolean mayInterruptIfRunning);
  boolean isCancelled();
  boolean isDone();
  V get() throws InterruptedException, ExeccutionException, CancellationException;
  V get(long timeout, TimeUnit unit) throws InterruptException, ExecutionException,CalcellationException, TimeoutExeception;	//设置超时的get()方法;
}
```

Future中的连个get()方法在获取线程执行的结果的时候都可能会抛出异常,我们的任务代码可以在收到这些异常后根据不同的情况来决定是否需要调用cancel()方法来终端该线程;

---

### 1. 处理不可阻塞的中断;

在java库中,许多可阻塞的方法都是通过提前返回或者抛出InterruptedException来响应中断请求的, 从而是开发人员更容易构建出能响应取消操作操作的任务, 然而,并非所有的可阻塞方法或者阻塞机制都能够响应中断;比如说:一个线程执行同步的Socker I/O,或者等待获得内置所而阻塞,那么终端请求只能设置线程的中断状态,除此之外没有其他任何作用.对于那些由于执行不可中断操作而被阻塞的线程, 可以使用类似于中断的手段来停止这些线程,但这要求我们必须知道线程阻塞的原因.实现的具体的方式是:

重写线程的interrupt()方法,在该方法的内部处理掉引起阻塞的条件,然后在调用super.interrupt()来中断线程;

```java
public class ReaderThread extends Thread {
  private final Socker socker;
  private final InputStream in;
  
  public ReaderThread (Socker socket) thros IOException {
    this. socket = socket;
    this.in = socket.getInputStream();
  }
  
  public void interrupt() {
    try {
      socket.close();
    } catch (IOException ignored) {}
    finally {
      super.interrup();
    }
    }
  }
  
  public void run() {
    ...
  }
}
```

### 停止基于线程的服务

* 日志服务
* 关闭ExecutorService
* 毒丸对象
* 处理线程的非正常终止

