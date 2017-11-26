#PriorityBlockingQueue

PriorityBlockingQueue 是一个带优先级的阻塞队列。默认情况下元素采用自然顺序升序排序(类需要实现Comparable 接口)，当然我们也可以通过构造函数来指定Comparator来对元素进行排序。对于优先级相同的两个元素，PriorityBlockingQueue无法保证它们的顺序。

理论上PriorityBlockingQueue是无界的，任何的添加操作都将会添加成功，直到耗尽资源耗尽为止。

PriorityBlockingQueue不接受null元素,队列中最小的元素存储在queue[0]位置;

### JVM实现

Jvm中使用的基于数组的(以下简称队列)二叉堆来存储PriorityBlockingQueue中的元素,同时对队列的同步操作都是通过同一个锁(main lock)来实现的.

理解二叉堆是理解PriorityBlockingQueue的关键.

在队列Resize时通过一个自旋锁(spinlock) 来保证同步,并且Resize操作是在没有获取队里的main lock时做的,这样做的原因是这样可以允许队列在Resize时依然依然可以并发的从队列中take元素.

### 成员变量 & 构造器

####成员变量:

```
// 初始的默认元素是11,
private static final int DEFAULT_INITIAL_CAPACITY = 11;

//最大的元素Integer.MAX_VALUE - 8;
//为什么要减8 :Some VMs reserve some header words in an array.
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

//实现二叉堆的数组;
private transient Object[] queue;

//队列中元素的数量
private transient int size;

//比较器, 如果队列使用的是元素的自然顺序,则该元素为null
private transient Comparator<? super E> comparator;

//main lock, 除了Resize操作,队列的所有操作得同步都是通过该锁实现的
private final ReentrantLock lock;

//当队列为空时,通过该Condition对象实现阻塞操作;
//同时由于队列是无界队列,所以任何类型的put操作都不会阻塞,所以就没有notFull的Condition对象;
private final Condition notEmpty;

//Resize时使用的SpinLock
private transient volatile int allocationSpinLock;
```

####构造器:

```
public PriorityBlockingQueue() {
	this(DEFAULT_INITIAL_CAPACITY, null);
}

public PriorityBlockingQueue(int initialCapacity) {
	this(initialCapacity, null);
}

public PriorityBlockingQueue(int initialCapacity,
						Comparator<? super E> comparator) {
      if (initialCapacity < 1)
      throw new IllegalArgumentException();
      this.lock = new ReentrantLock();
      this.notEmpty = lock.newCondition();
      this.comparator = comparator;
      this.queue = new Object[initialCapacity];
}
```



### 重要的方法

```
public void put(E e) {
	offer(e); // never need to block
}

public boolean offer(E e) {
	if (e == null)
		throw new NullPointerException();
	final ReentrantLock lock = this.lock;
	lock.lock();
	int n, cap;
	Object[] array;
	//如果队列满了,此时往队列中添加元素就会触发resize,由于当前线程的resize操作可能会失败,所以这里需要用while()
	while ((n = size) >= (cap = (array = queue).length))
		tryGrow(array, cap);
	try {
		Comparator<? super E> cmp = comparator;
		if (cmp == null)
			//此时的array引用的的已经是新的队列了
			siftUpComparable(n, e, array);
		else
			siftUpUsingComparator(n, e, array, cmp);
		size = n + 1;
		notEmpty.signal();
	} finally {
		lock.unlock();
	}
	return true;
}

/**
 * tryGrow的操作主要分为两步:1. 重新申请一个数组; 2. 将旧队列中的元素拷贝到新的队列中;
 */
private void tryGrow(Object[] array, int oldCap) {
	//调用该方法的方法首先要获取 main lock
	lock.unlock(); // must release and then re-acquire main lock
	Object[] newArray = null;
	// 1. 申请一个新的数组
	if (allocationSpinLock == 0 &&
		UNSAFE.compareAndSwapInt(this, allocationSpinLockOffset,
								 0, 1)) {
		try {
			//如果原始队列的与与三俗
			int newCap = oldCap + ((oldCap < 64) ?
								   (oldCap + 2) : // grow faster if small
								   (oldCap >> 1));
			//判断新的队列的元素数是否超过最大值
			if (newCap - MAX_ARRAY_SIZE > 0) {    // possible overflow
				int minCap = oldCap + 1;
				if (minCap < 0 || minCap > MAX_ARRAY_SIZE)
					throw new OutOfMemoryError();
				newCap = MAX_ARRAY_SIZE;
			}
			if (newCap > oldCap && queue == array)
				newArray = new Object[newCap];
		} finally {
			allocationSpinLock = 0;
		}
	}
	if (newArray == null) // back off if another thread is allocating
		Thread.yield();
	// 2. 锁住队列,将此时队列中剩余的元素拷贝达新的数组中,并将新的数组的引用富裕queue
	lock.lock();
	/*
	 * 申请newArray的操作可能会失败
	 * 只有是在当前线程成功申请了newArray并且此时的queue还是之前的队列,即没有被其他线程resize并重新复制queue时才会执行
	 */
	if (newArray != null && queue == array) {
		queue = newArray;
		System.arraycopy(array, 0, newArray, 0, oldCap);
	}
}
```

**二叉堆的上虑方法:**

```
private static <T> void siftUpComparable(int k, T x, Object[] array) {
	Comparable<? super T> key = (Comparable<? super T>) x;
	while (k > 0) {
		int parent = (k - 1) >>> 1;
		Object e = array[parent];
		if (key.compareTo((T) e) >= 0)
			break;
		array[k] = e;
		k = parent;
	}
	array[k] = key;
}
private static <T> void siftUpUsingComparator(int k, T x, Object[] array,
								   Comparator<? super T> cmp) {
	while (k > 0) {
		int parent = (k - 1) >>> 1;
		Object e = array[parent];
		if (cmp.compare(x, (T) e) >= 0)
			break;
		array[k] = e;
		k = parent;
	}
	array[k] = x;
}
```

**Poll()的实现**

```
// dequeue() 是poll()和 take()方法的底层实现,由于poll()和take()最终是要从队列中删除元素的,
// 所有这些操作涉及到队列元素充分布的问题;
// peek()方法只获取队列的首元素,不增删队列中的元素,所以peek()的实现没有依赖dequeue()方法;
private E dequeue() {
	int n = size - 1;
	if (n < 0)
		return null;
	else {
		Object[] array = queue;
		E result = (E) array[0];
		E x = (E) array[n];
		array[n] = null;
		Comparator<? super E> cmp = comparator;
		if (cmp == null)
			siftDownComparable(0, x, array, n);
		else
			siftDownUsingComparator(0, x, array, n, cmp);
		size = n;
		return result;
	}
}
```

```
public E take() throws InterruptedException {
	final ReentrantLock lock = this.lock;
	lock.lockInterruptibly();
	E result;
	try {
		while ( (result = dequeue()) == null)
			notEmpty.await();
	} finally {
		lock.unlock();
	}
	return result;
}

public E poll(long timeout, TimeUnit unit) throws InterruptedException {
	long nanos = unit.toNanos(timeout);
	final ReentrantLock lock = this.lock;
	lock.lockInterruptibly();
	E result;
	try {
		while ( (result = dequeue()) == null && nanos > 0)
			nanos = notEmpty.awaitNanos(nanos);
	} finally {
		lock.unlock();
	}
	return result;
}

public E peek() {
	final ReentrantLock lock = this.lock;
	lock.lock();
	try {
		return (size == 0) ? null : (E) queue[0];
	} finally {
		lock.unlock();
	}
}
```

**二叉堆的下虑方法:**

```
private static <T> void siftDownComparable(int k, T x, Object[] array,
										   int n) {
	if (n > 0) {
		Comparable<? super T> key = (Comparable<? super T>)x;
		int half = n >>> 1;           // loop while a non-leaf
		while (k < half) {
			int child = (k << 1) + 1; // assume left child is least
			Object c = array[child];
			int right = child + 1;
			if (right < n &&
				((Comparable<? super T>) c).compareTo((T) array[right]) > 0)
				c = array[child = right];
			if (key.compareTo((T) c) <= 0)
				break;
			array[k] = c;
			k = child;
		}
		array[k] = key;
	}
}

private static <T> void siftDownUsingComparator(int k, T x, Object[] array,
												int n,
												Comparator<? super T> cmp) {
	if (n > 0) {
		int half = n >>> 1;
		while (k < half) {
			int child = (k << 1) + 1;
			Object c = array[child];
			int right = child + 1;
			if (right < n && cmp.compare((T) c, (T) array[right]) > 0)
				c = array[child = right];
			if (cmp.compare(x, (T) c) <= 0)
				break;
			array[k] = c;
			k = child;
		}
		array[k] = x;
	}
}
```

**如果要添加的元素可能出现优先级相同的元素同时又需要保证其相应的顺序呢，应该怎么办？**

jvm源码中给出了建议：将要操作的类再进行一次封装，同时在类中添加一个全局的AtomicLong 变量，同时重写compareTo()方法,当基类的优先级相同时，使用全局的AtomicLong 变量来作为判断优先级高低的第二个判断条件;

```
 * class FIFOEntry<E extends Comparable<? super E>>
 *     implements Comparable<FIFOEntry<E>> {
 *   static final AtomicLong seq = new AtomicLong(0);
 *   final long seqNum;
 *   final E entry;
 *   public FIFOEntry(E entry) {
 *     seqNum = seq.getAndIncrement();
 *     this.entry = entry;
 *   }
 *   public E getEntry() { return entry; }
 *   public int compareTo(FIFOEntry<E> other) {
 *     int res = entry.compareTo(other.entry);
 *     if (res == 0 && other.entry != this.entry)
 *       res = (seqNum < other.seqNum ? -1 : 1);
 *     return res;
 *   }
 * }}
```



参考文献:

1. http://cmsblogs.com/?p=2407
2. 二叉堆的知识自行脑补;

