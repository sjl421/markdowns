# Java UncaughtExceptionHandler

由于java 线程本身的特点, 使得你不能捕获从线程中逃逸出的异常,一旦逃逸出run()的异常会向外传播到控制台;

java中提供了方法可以让我们来捕获这中异常, 通过使用java 体重的 UncaughtExceptionHandler接口,可以是我们为每一个Thread对象设定一个UncaughtExceptionHandler, 它会在线程因未捕获的异常而临近死亡时被调用;

java中的Thread对应的方法:

```java
Thread t = new Thread(new Runnable);
t.setUncaughtExceptionHandler(new MyUncaughtExceptionHander());
```

完整的测试代码:

```java
class ExceptionThread2 implements  Runnable{

    @Override
    public void run() {
        Thread t = Thread.currentThread();
        System.out.println("run() by " + t);
        System.out.println("eh = " + t.getUncaughtExceptionHandler());
        throw new RuntimeException();
    }
}

class MyUncaughtExceptionHander implements Thread.UncaughtExceptionHandler{

    @Override
    public void uncaughtException(Thread t, Throwable e) {
        System.out.println("caught:" + e);
    }
}

class HandlerThreadFactory implements ThreadFactory {

    @Override
    public Thread newThread(Runnable r) {
        System.out.println(this + " creating new thread");
        Thread t = new Thread(r);
        t.setUncaughtExceptionHandler(new MyUncaughtExceptionHander());
        System.out.println("en = " + t.getUncaughtExceptionHandler());
        return t;
    }
}

class ThreadWithExceptionHandler implements Runnable {
    @Override
    public void run() {
        System.out.println(Thread.currentThread() + " " + this);
        System.out.println("throw exception");
        throw new RuntimeException("hahahahaha");
    }
}

public class CpatureUncaughtException {
    public static void main(String[] args) {
        ExecutorService exec = Executors.newCachedThreadPool(new HandlerThreadFactory());
        exec.execute(new ExceptionThread2());
        exec.shutdown();

        System.out.println("====================================");
        Thread t = new Thread(new ThreadWithExceptionHandler());
        t.setUncaughtExceptionHandler(new MyUncaughtExceptionHander());
        t.start();
    }
}
```

上述的代码中有一部分是通过ThreadFactory, 通过实现此工厂方法,在使用线程池创建线程时,可以为每一个献策好难过都设定对应的UncaughtExceptionHandler;

上面的方法让你根据不同的情况设置不同的UncaughtExceptionHandler, 但是如果你知道你的设计中所有的线程都使用相同的ExceptionHandler, 那么你可以为Thread类设置一个默认的UncaughtExceptionHandler的, 这个处理器只有在线程没有设定专门店Handler时才会被调用;