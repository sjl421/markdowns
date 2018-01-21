# Java ThreadLocal

## ThreadLoacal是什么？

ThreadLocal是啥？以前面试别人时就喜欢问这个，有些伙伴喜欢把它和线程同步机制混为一谈，事实上ThreadLocal与线程同步无关。ThreadLocal虽然提供了一种解决多线程环境下成员变量的问题，但是它并不是解决多线程共享变量的问题。那么ThreadLocal到底是什么呢？

API是这样介绍它的：
**This class provides thread-local variables. These variables differ from their normal counterparts in that each thread that accesses one (via its {@code get} or {@code set} method) has its own, independently initialized copy of the variable. {@code ThreadLocal} instances are typically private static fields in classes that wish to associate state with a thread (e.g.,a user ID or Transaction ID).**

> 该类提供了线程局部 (thread-local) 变量。这些变量不同于它们的普通对应物，因为访问某个变量（通过其`get` 或 `set` 方法）的每个线程都有自己的局部变量，它独立于变量的**初始化副本**。`ThreadLocal`实例通常是类中的 private static 字段，它们希望将状态与某一个线程（例如，用户 ID 或事务 ID）相关联。

所以ThreadLocal与线程同步机制不同，线程同步机制是多个线程共享同一个变量，而ThreadLocal是为每一个线程创建一个单独的变量副本，故而每个线程都可以独立地改变自己所拥有的变量副本，而不会影响其他线程所对应的副本。可以说ThreadLocal为多线程环境下变量问题提供了另外一种解决思路。

ThreadLocal定义了四个方法：

- get()：返回此线程局部变量的当前线程副本中的值。
- initialValue()：返回此线程局部变量的当前线程的“初始值”。
- remove()：移除此线程局部变量当前线程的值。
- set(T value)：将此线程局部变量的当前线程副本中的值设置为指定值。

除了这四个方法，ThreadLocal内部还有一个静态内部类ThreadLocalMap，该内部类才是实现线程隔离机制的关键，get()、set()、remove()都是基于该内部类操作。ThreadLocalMap提供了一种用键值对方式存储每一个线程的变量副本的方法，key为当前ThreadLocal对象，value则是对应线程的变量副本。

对于ThreadLocal需要注意的有两点：

1. ThreadLocal实例本身是不存储值，它只是提供了一个在当前线程中找到副本值得key。
2. 是ThreadLocal包含在Thread中，而不是Thread包含在ThreadLocal中，有些小伙伴会弄错他们的关系。

下图是Thread、ThreadLocal、ThreadLocalMap的关系（<http://blog.xiaohansong.com/2016/08/06/ThreadLocal-memory-leak/>）

![Thread、ThreadLocal、ThreadLocalMap的关系](http://img.blog.csdn.net/20171005154736367?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlbnNzeQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## ThreadLocal使用示例

示例如下：

```
public class SeqCount {

    private static ThreadLocal<Integer> seqCount = new ThreadLocal<Integer>(){
        // 实现initialValue()
        public Integer initialValue() {
            return 0;
        }
    };

    public int nextSeq(){
        seqCount.set(seqCount.get() + 1);

        return seqCount.get();
    }

    public static void main(String[] args){
        SeqCount seqCount = new SeqCount();

        SeqThread thread1 = new SeqThread(seqCount);
        SeqThread thread2 = new SeqThread(seqCount);
        SeqThread thread3 = new SeqThread(seqCount);
        SeqThread thread4 = new SeqThread(seqCount);

        thread1.start();
        thread2.start();
        thread3.start();
        thread4.start();
    }

    private static class SeqThread extends Thread{
        private SeqCount seqCount;

        SeqThread(SeqCount seqCount){
            this.seqCount = seqCount;
        }

        public void run() {
            for(int i = 0 ; i < 3 ; i++){
                System.out.println(Thread.currentThread().getName() + " seqCount :" + seqCount.nextSeq());
            }
        }
    }
}
```

运行结果：

![运行结果](http://img.blog.csdn.net/20171005154820871?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2hlbnNzeQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

从运行结果可以看出，ThreadLocal确实是可以达到线程隔离机制，确保变量的安全性。这里我们想一个问题，在上面的代码中ThreadLocal的initialValue()方法返回的是0，加入该方法返回得是一个对象呢，会产生什么后果呢？例如：

```
    A a = new A();
    private static ThreadLocal<A> seqCount = new ThreadLocal<A>(){
        // 实现initialValue()
        public A initialValue() {
            return a;
        }
    };

    class A{
        // ....
    }
```

具体过程请参考：[对ThreadLocal实现原理的一点思考](http://www.jianshu.com/p/ee8c9dccc953)

http://cmsblogs.com/?p=2442







学习一个东西首先要知道为什么要引入它，就是我们能用它来干什么。所以我们先来看看ThreadLocal对我们到底有什么用，然后再来看看它的实现原理。

ThreadLocal如果单纯从名字上来看像是“本地线程"这么个意思，只能说这个名字起的确实不太好，很容易让人产生误解，ThreadLocalVariable（线程本地变量）应该是个更好的名字。我们先看一下官方对ThreadLocal的描述：

> 该类提供了线程局部 (thread-local) 变量。这些变量不同于它们的普通对应物，因为访问某个变量（通过其 get 或 set 方法）的每个线程都有自己的局部变量，它独立于变量的初始化副本。ThreadLocal 实例通常是类中的 private static 字段，它们希望将状态与某一个线程（例如，用户 ID 或事务 ID）相关联。

我们从中摘出要点：

1、每个线程都有自己的局部变量

​    每个线程都有一个独立于其他线程的上下文来保存这个变量，一个线程的本地变量对其他线程是不可见的（有前提，后面解释）

2、独立于变量的初始化副本

​    ThreadLocal可以给一个初始值，而每个线程都会获得这个初始化值的一个副本，这样才能保证不同的线程都有一份拷贝。

3、状态与某一个线程相关联

​    ThreadLocal 不是用于解决共享变量的问题的，不是为了协调线程同步而存在，而是为了方便每个线程处理自己的状态而引入的一个机制，理解这点对正确使用ThreadLocal至关重要。

我们先看一个简单的例子：

```
public class ThreadLocalTest {
        
        //创建一个Integer型的线程本地变量
	public static final ThreadLocal<Integer> local = new ThreadLocal<Integer>() {
		@Override
		protected Integer initialValue() {
			return 0;
		}
	};
	public static void main(String[] args) throws InterruptedException {
		Thread[] threads = new Thread[5];
		for (int j = 0; j < 5; j++) {		
               threads[j] = new Thread(new Runnable() {
				@Override
				public void run() {
                                        //获取当前线程的本地变量，然后累加5次
					int num = local.get();
					for (int i = 0; i < 5; i++) {
						num++;
					}
                                        //重新设置累加后的本地变量
					local.set(num);
					System.out.println(Thread.currentThread().getName() + " : "+ local.get());

				}
			}, "Thread-" + j);
		}

		for (Thread thread : threads) {
			thread.start();
		}
	}
}
```

运行后结果：

Thread-0 : 5
Thread-4 : 5
Thread-2 : 5
Thread-1 : 5
Thread-3 : 5

我们看到，每个线程累加后的结果都是5，各个线程处理自己的本地变量值，线程之间互不影响。

我们再来看一个例子：

```
public class ThreadLocalTest {
	private static Index num = new Index();
        //创建一个Index类型的本地变量 
	private static ThreadLocal<Index> local = new ThreadLocal<Index>() {
		@Override
		protected Index initialValue() {
			return num;
		}
	};

	public static void main(String[] args) throws InterruptedException {
		Thread[] threads = new Thread[5];
		for (int j = 0; j < 5; j++) {
			threads[j] = new Thread(new Runnable() {
				@Override
				public void run() {
                                        //取出当前线程的本地变量，并累加1000次
					Index index = local.get();
					for (int i = 0; i < 1000; i++) {                                          
						index.increase();
					}
					System.out.println(Thread.currentThread().getName() + " : "+ index.num);

				}
			}, "Thread-" + j);
		}
		for (Thread thread : threads) {
			thread.start();
		}
	}

	static class Index {
		int num;

		public void increase() {
			num++;
		}
	}
}
```

执行后我们发现结果如下（每次运行还都不一样）：

Thread-0 : 1390
Thread-2 : 2390
Thread-4 : 4390
Thread-3 : 3491
Thread-1 : 1390

这次为什么线程本地变量又失效了呢？大家可以仔细观察上面代码自己先找一下原因。

-----------------------------------------------低调的分割线-------------------------------------------

让我们再来回味一下 “ThreadLocal可以给一个初始值，而每个线程都会获得这个初始化值的一个副本” 这句话。“初始值的副本。。。”，貌似想起点什么。我们再来看一下上面代码中定义ThreadLocal的地方

```
private static Index num = new Index();
	private static ThreadLocal<Index> local = new ThreadLocal<Index>() {
		@Override
		protected Index initialValue() {
			return num;       // 注意这里，返回的是已经定义好的对象num，而不是new Index()
		}
	};
```

 上面代码中，我们通过覆盖initialValue函数来给我们的ThreadLocal提供初始值，每个线程都会获取这个初始值的一个副本。而现在我们的初始值是一个定义好的一个对象，num是这个对象的引用.换句话说我们的初始值是一个引用。引用的副本和引用指向的不就是同一个对象吗？

![img](http://static.oschina.net/uploads/space/2013/0801/233058_dbbJ_578710.png)

如果我们想给每一个线程都保存一个Index对象应该怎么办呢？那就是创建对象的副本而不是对象引用的副本：

```
private static ThreadLocal<Index> local = new ThreadLocal<Index>() {
		@Override
		protected Index initialValue() {
			return new Index(); //注意这里
		}
	};
```

对象的拷贝图示：

  ![img](http://static.oschina.net/uploads/space/2013/0801/233744_Hpao_578710.png)

现在我们应该能明白ThreadLocal本地变量的含义了吧。接下来我们就来看看ThreadLocal的源码，从内部来揭示它的神秘面纱。

ThreadLocal有一个内部类ThreadLocalMap,这个类的实现占了整个ThreadLocal类源码的一多半。这个ThreadLocalMap的作用非常关键，它就是线程真正保存线程自己本地变量的容器。每一个线程都有自己的单独的一个ThreadLocalMap实例，其所有的本地变量都会保存到这一个map中。现在就让我们从ThreadLocal的get和set这两个最常用的方法开始分析：

```
public T get() {
        //获取当前执行线程
        Thread t = Thread.currentThread();
        //取得当前线程的ThreadLocalMap实例
        ThreadLocalMap map = getMap(t);
        //如果map不为空，说明该线程已经有了一个ThreadLocalMap实例
        if (map != null) {
            //map中保存线程的所有的线程本地变量，我们要去查找当前线程本地变量
            ThreadLocalMap.Entry e = map.getEntry(this);
            //如果当前线程本地变量存在这个map中，则返回其对应的值
            if (e != null)
                return (T)e.value;
        }
        //如果map不存在或者map中不存在当前线程本地变量，返回初始值
        return setInitialValue();
    }
```

强调一下：Thread对象都有一个ThreadLocalMap类型的属性threadLocals，这个属性是专门用于保存自己所有的线程本地变量的。这个属性在线程对象初始化的时候为null。所以对一个线程对象第一次使用线程本地变量的时候，需要对这个threadLocals属性进行初始化操作。注意要区别 “线程第一次使用本地线程变量”和“第一次使用某一个线程本地线程变量”。

getMap方法:

```
//直接返回线程对象的threadLocals属性
ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
```

setInitialValue方法：（看完后再回顾一下之前的那个例子）

```
private T setInitialValue() {
        //获取初始化值，initialValue 就是我们之前覆盖的方法
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        //如果map不为空，将初始化值放入到当前线程的ThreadLocalMap对象中
        if (map != null)
            map.set(this, value);
        else
            //当前线程第一次使用本地线程变量，需要对map进行初始化工作
            createMap(t, value);
        //返回初始化值
        return value;
    }
```

我们再来看一下set方法

```
public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);

        if (map != null)
            map.set(this, value);
        //说明线程第一次使用线程本地变量(注意这里的第一次含义)
        else
            createMap(t, value);
    }
```

ThradLocal还有一个remove方法，让我们来分析一下:

```
public void remove() {
         //获取当前线程的ThreadLocalMap对象
         ThreadLocalMap m = getMap(Thread.currentThread());
         //如果map不为空，则删除该本地变量的值
         if (m != null)
             m.remove(this);
     }
```

到这里大家应该对ThreadLocal变量比较清晰了，至于ThradLocalMap的实现细节这里就不在说了。大家有兴趣可以自己去看ThreadLocal的源码。