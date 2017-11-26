#While-Wait-Or-If-Wait

 先看一段简单的代码：这段代码在判断wait条件的时候使用的while（），而另一种写法是在判断等待条件时使用的是if判断；

```
 public void add(T t) throws InterruptedException {
    	//在调用condition的方法是先获取锁
        lock.lock();
        try {
            //注意，这里用的是while而不是if
            while (count == items.length) {
                notFull.await();
            }

            items[addIndex] = t;
            if (++addIndex == items.length) {
                addIndex = 0;
            }
            ++count;
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
 }
```

对于应该使用while判断还是if判断，我们的结论是：永远不要在循环之外调用wait()方法<Effective Java> 第69条。 解释：

>循环会在等待之前和等待之后分别测试等待条件；
>
>1. 在等待之前测试条件，当条件已经成立时就跳过等待， 这对于确保活性（liveness）是必须的。如果条件已经成立，并且在线程等待之前，notify（或者notifyAll）方法已经被调用，则无法保证该线程从等待中苏醒过来（如果在该线程wait之前调用了notify，那么该线程由于之前已经判断等待条件已经成立了，会继续执行wait方法，但是此时notify方法已经被调用了，其他线程已经不会再去重新调用notify方法了，这就造成了等待线程一直等待下， but， 这句话到底想说什么？）；
>2. 在等待之后测试条件，如果条件不成立的话继续等待，这对于确保安全性（safety）是必要的。当条件不成立的时候，如果线程继续执行，则可能会破坏被锁保护的约束条件。当条件不成立时，有下面一些理由可使一个线程苏醒过来；
>   * 另一个线程可能已经得到了锁，并且从一个线程调用notify那一刻起，到等待线程苏醒过来的这段时间内，得到锁的线程已经改变了受保护的状态；
>   * 条件并不成立， 但是另一个线程可能意外地或者恶意地调用了notify。在公有可访问的对象上等待，这些实际上把自己暴露在了这种危险的境地中。公有可访问的同步方法中包含了wait都会出现这样的问题。
>   * 通知线程在唤醒等待线程时可能会过度“大方”。例如：即使只有某一些等待线程的条件已经被满足，但是通知线程可能仍然调用notifyAll。
>   * 在没有通知的情况下，等待线程也可能会（但很少）会苏醒过来。这被称为“伪唤醒（spurious wakeup）；



