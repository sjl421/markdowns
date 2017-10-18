#ReentrantReadWriteLock

重入锁ReentrantLock是排他锁，排他锁在同一时刻仅有一个线程可以进行访问，但是在大多数场景下，大部分时间都是提供读服务，而写服务占有的时间较少。然而读服务不存在数据竞争问题，如果一个线程在读时禁止其他线程读势必会导致性能降低。所以就提供了读写锁。

读写锁维护着一对锁，一个读锁和一个写锁。通过分离读锁和写锁，使得并发性比一般的排他锁有了较大的提升：在同一时间可以允许多个读线程同时访问，但是在写线程访问时，所有读线程和写线程都会被阻塞。

读写锁的主要特性：

1. 公平性：支持公平性和非公平性。
2. 重入性：支持重入。读写锁最多支持65535个递归写入锁和65535个递归读取锁。
3. 锁降级：遵循获取写锁、获取读锁在释放写锁的次序，写锁能够降级成为读锁

读写锁ReentrantReadWriteLock实现接口ReadWriteLock，该接口维护了一对相关的锁，一个用于只读操作，另一个用于写入操作。只要没有 writer，读取锁可以由多个 reader 线程同时保持。写入锁是独占的。

### 构造器

ReentrantReadWriteLock实现了ReadWriteLock接口，该接口中实现的了

```java
public interface ReadWriteLock {
    //Returns the lock used for reading.
    Lock readLock();
    
    //Returns the lock used for writing.
    Lock writeLock();
}
```

```
//默认构造器，ReentrantReadWriteLock默认是以非公平的方式实现；
public ReentrantReadWriteLock() {
	this(false);
}

public ReentrantReadWriteLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
    readerLock = new ReadLock(this);
    writerLock = new WriteLock(this);
}
```



```
public class ReentrantReadWriteLock
        implements ReadWriteLock{
    //内部类， 读锁
    private final ReentrantReadWriteLock.ReadLock readerLock;
    //内部类，写锁
    private final ReentrantReadWriteLock.WriteLock writerLock;
    //内部实现的同步组件，用来实现所有的同步行为
    final Sync sync;
```

### Sync

`abstract static class Sync extends AbstractQueuedSynchronizer `是ReentrantReadWriteLock 的同步实现；

*  它的两个子类`FairSync` 和`NoFairSync` 分别实现的公平锁和非公平锁的功能；
* WriteLock 和ReadLock同时有依赖同一个Sync对象；





































### 使用场景

ReentrantReadWriteLock在某些场景下可以提高读写的性能，比如说对并发的访问规模较大的集合时同时对结合的并发读的线程大于对集合并发写的线程数；

```
 * class RWDictionary {
 *   private final Map<String, Data> m = new TreeMap<String, Data>();
 *   private final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
 *   private final Lock r = rwl.readLock();
 *   private final Lock w = rwl.writeLock();
 *
 *   public Data get(String key) {
 *     r.lock();
 *     try { return m.get(key); }
 *     finally { r.unlock(); }
 *   }
 *   public String[] allKeys() {
 *     r.lock();
 *     try { return m.keySet().toArray(); }
 *     finally { r.unlock(); }
 *   }
 *   public Data put(String key, Data value) {
 *     w.lock();
 *     try { return m.put(key, value); }
 *     finally { w.unlock(); }
 *   }
 *   public void clear() {
 *     w.lock();
 *     try { m.clear(); }
 *     finally { w.unlock(); }
 *   }
 * }}
```

