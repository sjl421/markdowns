# Java中创建多线程的方法

java中创建多线程的方法大概有3中

1. 实现Runnable接口,然后通过new Thread(new Runnable).start() 来创建多线程;

```

```

> 这种做法也是比较常用的一种作法;

2. 实现线程类, 通过继承Thread 并重写其中的public void run() 方法来创建线程;

```java

```

> 这种做法的问题是由于java不支持多继承,所以在如果你的类继承了Thread()之后, 就无法再去继承其它的类了;

3. 实现Executor接口, 然后传入Runnable接口, public void execute() 方法中运行Runnable接口;

Executor接口:

```java
public interface Executor{
  void execute(Runnable command);
}
```

实例程序:

```
public class ThreadTaskExecutor implements Executor{
	public void executor(Runnable r) {
      new Thread(r).start();
	}
}
```

通过Executor框架实现多线程的好处是:

> 通过使用Executor, 将请求处理任务的提交和任务的实际执行解耦开来,并且只需采用另一个中不同的Executor配置就可以改变服务器的行为;

同时可以通过是想用Executor创建一个类似于单线程的行为(在这个线程内的行为类似于单线程的,这个线程执行很多的任务,这些是以单线程的形式执行的)基于同步的方式执行每个任务:

```java
public class WithInThreadExecutor implements Executor {
  public void executor(Runnalbe r) {
  	doSomenelseTask1();
    r.run();
    doSomenelseTask1();
  }
}
```

4. 线程池的使用

Executor 接口中之定义了一个方法 execute（Runnable command），该方法接收一个 Runable 实例，它用来执行一个任务，任务即一个实现了 Runnable 接口的类。ExecutorService 接口继承自 Executor 接口，它提供了更丰富的实现多线程的方法，比如，ExecutorService 提供了关闭自己的方法，以及可为跟踪一个或多个异步任务执行状况而生成 Future 的方法。 可以调用 ExecutorService 的 shutdown（）方法来平滑地关闭 ExecutorService，调用该方法后，将导致 ExecutorService 停止接受任何新的任务且等待已经提交的任务执行完成(已经提交的任务会分两类：一类是已经在执行的，另一类是还没有开始执行的)，当所有已经提交的任务执行完毕后将会关闭 ExecutorService。因此我们一般用该接口来实现和管理多线程。

ExecutorService 的生命周期包括三种状态：运行、关闭、终止。创建后便进入运行状态，当调用了 shutdown（）方法时，便进入关闭状态，此时意味着 ExecutorService 不再接受新的任务，但它还在执行已经提交了的任务，当素有已经提交了的任务执行完后，便到达终止状态。如果不调用 shutdown（）方法，ExecutorService 会一直处在运行状态，不断接收新的任务，执行新的任务，服务器端一般不需要关闭它，保持一直运行即可。

Executors 提供了一系列工厂方法用于创先线程池，返回的线程池都实现了 ExecutorService 接口。

public static ExecutorService newFixedThreadPool(int nThreads)
创建固定数目线程的线程池。

public static ExecutorService newCachedThreadPool()
创建一个可缓存的线程池，调用execute将重用以前构造的线程（如果线程可用）。如果现有线程没有可用的，则创建一个新线 程并添加到池中。终止并从缓存中移除那些已有 60 秒钟未被使用的线程。

public static ExecutorService newSingleThreadExecutor()
创建一个单线程化的Executor。

public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize)
创建一个支持定时及周期性的任务执行的线程池，多数情况下可用来替代Timer类。

这四种方法都是用的 Executors 中的 ThreadFactory 建立的线程，下面就以上四个方法做个比较：

详细介绍请看另一片md <并发新特性—Executor 框架与线程池>

```java
class Task implements Runnable {
    private int count;
    public Task (int count) {
        this.count =count;
    }

    @Override
    public void run() {
        System.out.println("Thread:" + count + " is running");
        try {
            TimeUnit.MILLISECONDS.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Thread:" + count + " is over");
    }
}

public class IThreadPool {

//    @Test
//    public void test0() {
    public static void main(String[] args) {
        ExecutorService threadPool = Executors.newFixedThreadPool(2);
        for (int i = 0; i < 5; i++) {
            threadPool.execute(new Task(i));
        }
        threadPool.shutdown();
    }
}
```

### java中的ExecutorService接口

java中ExecutorService接口继承了Executor接口,所以可以直接同通过ExecutorService.execute(Runnalbe 接口)来执行多线程.但是同时 ExecutorService又增加了一些对线程(池)声明周期管理的一些接口, 使得其可以直接管理线程(池)的声明周期;

同时ExecutorService又可以用来创建FutureTask线程;

```java
boolean	awaitTermination(long timeout, TimeUnit unit) 
  //Blocks until all tasks have completed execution after a shutdown request, or the timeout occurs, or the current thread is interrupted, whichever happens first.
<T> List<Future<T>>	invokeAll(Collection<? extends Callable<T>> tasks) 
  //Executes the given tasks, returning a list of Futures holding their status and results when all complete.
<T> List<Future<T>>	invokeAll(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit)
  //Executes the given tasks, returning a list of Futures holding their status and results when all complete or the timeout expires, whichever happens first.
<T> T	invokeAny(Collection<? extends Callable<T>> tasks)
  //Executes the given tasks, returning the result of one that has completed successfully (i.e., without throwing an exception), if any do.
<T> T	invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit)
  //Executes the given tasks, returning the result of one that has completed successfully (i.e., without throwing an exception), if any do before the given timeout elapses.
boolean	isShutdown()
  //Returns true if this executor has been shut down.
boolean	isTerminated()
  //Returns true if all tasks have completed following shut down.
void	shutdown() 
  //Initiates an orderly shutdown in which previously submitted tasks are executed, but no new tasks will be accepted.
List<Runnable>	shutdownNow() 
  //Attempts to stop all actively executing tasks, halts the processing of waiting tasks, and returns a list of the tasks that were awaiting execution.
<T> Future<T>	submit(Callable<T> task) 
  //Submits a value-returning task for execution and returns a Future representing the pending results of the task.
Future<?>	submit(Runnable task) 
  //Submits a Runnable task for execution and returns a Future representing that task.
<T> Future<T>	submit(Runnable task, T result) 
  //Submits a Runnable task for execution and returns a Future representing that task.
```

