# Condition







### 实例代码

```
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class BoundedQueue<T> {
    private Object[] items;
    private int addIndex, removeIndex, count;
    private Lock lock = new ReentrantLock();

    private Condition notEmpty = lock.newCondition();
    private Condition notFull = lock.newCondition();

    public BoundedQueue(int size) {
        items = new Object[size];
    }

    public void add(T t) throws InterruptedException {
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

    public T remove() throws InterruptedException {
        lock.lock();
        T rt;
        try {
            //注意，这里用的是while而不是if
            while (count == 0) {
                notEmpty.await();
            }
            rt = (T) items[removeIndex];
            if (removeIndex == items.length) {
                removeIndex = 0;
            }
            --count;
            notFull.signal();
            return rt;
        } finally {
            lock.unlock();
        }
    }

    static class Worker implements Runnable {
        private BoundedQueue<Integer> queue;
        private String name;
        //0:add, 1:remove
        private int type;

        public Worker(BoundedQueue<Integer> queue, String name, int type) {
            this.queue = queue;
            this.name = name;
            this.type = type;
        }

        public void run() {
            System.out.println(String.format("%s is running", name));
            try {
                for (int i = 0; i < 10; i++) {
                    if (type == 0) {
                        queue.add(i);
                        System.out.println(String.format("Thread add %d", i));
                        Thread.sleep(500);
                    } else {
                        int data = queue.remove();
                        System.out.println(String.format("Thread remove %d:%d", i, data));
                        Thread.sleep(1000);
                    }
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        BoundedQueue<Integer> queue = new BoundedQueue<Integer>(2);
        Worker addWorker = new Worker(queue, "addWorker", 0);
        Worker removeWorker = new Worker(queue, "removeWorker", 1);
        new Thread(addWorker).start();
        new Thread(removeWorker).start();
    }
}
```

