# Java 线程的取消机制

[TOC]

## 1. 线程本身检查中断标志位

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

## 3.通过使用ExecutorService来管理线程的生命周期;

## 4. Future, FutureTask

### 1. cancel()

## 5. BlockingQueue

3. 计时运行
4. 处理不可阻塞的中断;

### 停止基于线程的服务

1. 日志服务
2. 关闭ExecutorService
3. 毒丸对象
4. 处理线程的非正常终止


