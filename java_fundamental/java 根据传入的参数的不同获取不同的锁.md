# java 根据传入的参数的不同获取不同的锁

最开始的时候碰到一个问题是对某些资源的访问, 资源有很多个具体的实例,希望java代码在针对某个是具体实例并发操作的时候对这个实例加上锁,保证对该资源的并发访问的安全性,但是此时又希望不会阻塞到对其他资源实例的操作, 此时针对折中情况就需要根据不同的资源生成不同的锁, 具体到java代码中可能的一种情况就是针对方法传入的参数来来生成具体的锁;

自己写了一段java代码如下:

```java
class SynchronizedKey implements Runnable {
    private static Map<String, byte[]> syncMap = new ConcurrentHashMap<String, byte[]>();
    private String key;
    private static Random rand = new Random(100);

    static {
        syncMap.put("0", new byte[0]);
        syncMap.put("1", new byte[0]);
        syncMap.put("2", new byte[0]);
        syncMap.put("3", new byte[0]);
        syncMap.put("4", new byte[0]);
    }

    public SynchronizedKey(String key) {
        this.key = key;
    }

    @Override
    public void run() {
        thread();
    }

    public void thread() {
        synchronized (syncMap.get(key)) {
            try {
                int i = rand.nextInt(10);
                System.out.println(key + " rand:" + i + " doing something");
                TimeUnit.MILLISECONDS.sleep(500 + i * 500);
                System.out.println(key + " exit");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public void thread2() {
        byte[] syncObj = syncMap.get(key);
        synchronized (SynchronizedKey.class) {
            if (syncObj == null) {
                syncMap.put(key, new byte[0]);
            }
        }
        synchronized (syncMap.get(key)) {
            try {
                int i = rand.nextInt(10);
                System.out.println(key + " rand:" + i + " doing something");
                TimeUnit.MILLISECONDS.sleep(500 + i * 500);
                System.out.println(key + " exit");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

public class TestSynchronizedKey {
    public static void main(String[] args) throws InterruptedException {
        TimeUnit.MILLISECONDS.sleep(1000);
        for (int i = 0; i < 5; i++) {
            new Thread(new SynchronizedKey(String.valueOf(i))).start();
        }
    }
}
```

SynchronizedKey 类有一个static的Map来针对所有的资源生成一个唯一的锁,这里用的对象是byte[0]\(这里必须是对象才可以,并且据说java代码在new byte[0] 的指令是最少的)

这样然在thread()方法中必须要获取根据传入的参数获取对应的byte[0]锁, `  synchronized (syncMap.get(key))  `

这段代码的运行是ok的, main方法中每个线程传入的参数的是不同的,所以并没有响应的线程会去争夺锁,所以线程可以无阻塞的并发的执行, 但是如果在传入的参数中会倒是线程需要去争夺同一个锁,那么线程就会发生相应的阻塞了;



### 问题2

上面的处理是在一直所有资源的前提下已经为所有的资源都生成了对应的锁,所以后面的代码在执行的过程中只需要获取响应的锁就可以了, 我有想到了一种其他的情况, 就是本来并不知道有那些资源的存在, 只知道要根据方法传入的参数去操作这个参数锁表示的具体的资源而已,此时为了防止对同时对同一个资源的访问, 还是需要通过加锁的方式来实现,只不过此时对应的资源的锁是并不存在的.

如果依然采用上面的通过map的方式来管理所,需要第一个获得资源访问权的线程往map中放入相应的锁,然后:

1. 自己再占掉这个锁,做响应的操作,此时其他线程需要通过 `synchronized (syncMap.get(key)) ` 来获得这个锁;
2. 线程放入锁后并不占掉这个锁, 需要再重新通过` synchronized (syncMap.get(key)) ` 来获得这个锁;

这里同时有一个很严重的问题就是我们这里实现了向Map中放入锁的动作,却没有从Map remove锁的动作,如果资源量很大,需要放的锁很多,可能还是需要某些处理逻辑将在操作完成之后将对应的锁 remove掉;

* 同时还需要认真的考虑的一个问题就是如何保证安全的add 锁 和remove 锁？