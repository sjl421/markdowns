### Daemon 线程

Daemon是java中的守护线程；

所谓守护线程，是指在程序运行的时候再后台提供一些通用服务的线程，并且这种线程并不属于程序不可或缺的一部分，因此当所有的非后台线程结束时，程序也就终止了，同时会杀死进程中的所有后台进程。

daemon线程的设置方法：

在`线程开始运行之前`调用线程的 setDaemon(true)方法；

```java
Thread thread = new Thread(new DaeMonTask(i));
thread.setDaemon(true);
thread.start();
```



```
public class DaeMonTask implements Runnable{
    private int id;

    public DaeMonTask(int id) {
        this.id = id;
    }

    @Override
    public void run() {
        try {
            while (true) {
                TimeUnit.MILLISECONDS.sleep(1000);
                System.out.println(Thread.currentThread() + " " + this);
            }
        } catch(InterruptedException e){
            e.printStackTrace();
        } finally {
            System.out.printf(Thread.currentThread() + " " + this + "Finally");
        }
    }

    public static void main(String[] args) throws InterruptedException {
        ExecutorService exec = Executors.newCachedThreadPool();
        for (int i = 0; i <5; i++) {
            Thread thread = new Thread(new DaeMonTask(i));
            thread.setDaemon(true);
            thread.start();
        }
        System.out.println("All daemon is started");
        TimeUnit.MILLISECONDS.sleep(5000);
    }
}
```

这里需要注意的地方是：

如果你线程中的run ()中右finally语句，那么这些语句是不会被执行的，这是由于daemon线程在最后一个非后台线程终止时，后台线程会“突然”终止，因此，一旦main（）退出，JVM会立即关闭所有的后台进程， 而不会有任何你希望出现的确认形式。