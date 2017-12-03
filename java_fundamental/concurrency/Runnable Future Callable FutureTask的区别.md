#Runnable Future Callable FutureTask的区别

> 该文档针对jdk1.8

[TOC]

### Runnable

Runnable接口是java并发矿建中的最常用的接口，也应该是所有学习java并发框架最开始接触的接口；

Runnable接口中只有一个run()方法,在通过创建线程来执行Runnalbe接口时执行的就是run()方法;

Runnable接口的特点(缺点)是:

1. 没有返回值;
2. 不能跑出受检查的异常类型;

正是由于这两个缺点,才有了后面的Callable接口;

```
public interface Runnable {
    public abstract void run();
}
```

调用方式:

1. 通过Thread类来指定Runnalbe接口:

```
new Thread(new Runnable()).start() 
```

2. 通过ExecutorService的submit()方法:

```
Future<?> submit(Runnable task);
<T> Future<T> submit(Runnable task, T result);
```

### Callable\<V>

为了弥补Runnalbe接口的缺点才有了Callable\<V>接口,callble接口中只有一个方法call(),该方法配合Future接口,可以实现:

1. 有返回值, 返回的参数类型就是传入的类型V;	
2. 在运行时可以抛出接受检查的异常;

```
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     */
    V call() throws Exception;
}
```

调用方式:通过ExecutorService中的submit方法和invoke方法调用, 返回的Future<T>;

```
Future<T> ExecutorService.submit(Callable<T> task)
List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;
List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                  long timeout, TimeUnit unit)
        throws InterruptedException;
T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;
T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;        
```

### Future

Future接口搭配Callable接口实现了线程返回值,同时Future接口也提供了对线程生命周期的管理功能;

```
public interface Future<V> {
  //cancel线程,如果线程已经执行结束,活着已经被cancelled了,或者由于某些原因无法cancel,返回false; 如果线程还尚未开始执行,返回ture;如果函数已经开始执行了,并且如果mayInterruptIfRunning = ture,则尝试中断线程;
  boolean cancel(boolean mayInterruptIfRunning); 
  //如果线程正常执行结束, 或者被异常中断,或者被cancell并且停止了,返回true;
  boolean isCancelled();
  //如果线程正常执行结束了,返回true;
  boolean isDone();
  //阻塞获取线程执行结果, 返回类V;
  V get() throws InterruptedException, ExeccutionException, CancellationException;
  V get(long timeout, TimeUnit unit) throws InterruptException, ExecutionException,CalcellationException, TimeoutExeception;	//设置超时的get()方法;
}
```

###FutureTask

FutureTask接口实现了RunnableFuture接口,而RunnableFuture接口同时继承了Runnalbe接口和Future接口,这最终就造成了FutureTask即拥有了Runnable接口和Callabel接口的可执行性,又拥有了Future接口的作为执行结果的返回性,所以可以: 

实例化一个FutureTask类，然后通过ExecutorService 的submit（）方法来执行该类， 然后就就可以通过该实例调用Future的相关接口来检测该线程的执行状态; 否则如果使用Future接口话在submit()的时候还要显式的接受submit的返回结果;

![FutureTask](G:\snap\FutureTask.png)

FutureTask的构造函数:

```
public FutureTask(Callable<V> callable)  
public FutureTask(Runnable runnable, V result)
```

FutureTask Usage Example:

```
FutureTask<String> task = new FutureTask<String>(new Callable<String>() {
    @Override
    public String call() throws Exception {
        TimeUnit.MILLISECONDS.sleep(new Random(System.currentTimeMillis()).nextInt(10)*100);
        System.out.println("Task completed");
        return "done...";
    }
});

ExecutorService exec = Executors.newCachedThreadPool();
exec.submit(task);
task.get(500, TimeUnit.MILLISECONDS);
```

同时Future和FutureTask的两外一个很重要的功能:**进程生命周期的管理**

---

下面附上之前的另外一篇md,两篇的内容有点重复,放到一起吧, 懒得做最终的统一整理了:

### 为什么会有Future和FutureTask类

场景： 在某种场景下主进程需要长时间的计算，此时可以使用FutureTask类来执行这个复杂的计算，然后主进程继续去执行一些其他的操作，然后等主进程处理完自己的操作之后再去通过FutureTask去取计算的结果，如果此时FutureTask类还没有计算完成，则FutureTask的get()会阻塞直到线程执行结束，得到结果；

### Future 和 Callable 接口

java中通过Runnable接口来创建线程的缺点是**无法返回结果**和**抛出受检查的异常**, 对与需要返回计算结果和抛出受检查异常的线程来说,可以使用Future和Callable接口来实现:

Callable 和Future接口:

```java
public interface Callable<V> {
  V call() throws Exception;
}

public interface Future<V> {
  boolean cancel(boolean mayInterruptIfRunning);
  boolean isCancelled();
  boolean isDone();
  V get() throws InterruptedException, ExeccutionException, CancellationException;
  V get(long timeout, TimeUnit unit) throws InterruptException, 				ExecutionException,CalcellationException, TimeoutExeception;	//设置超时的get()方法;
}
```

然后通过Future<T> ExecutorService.submit(Callable<T> task) 来提交线程;

需要注意的是: 

- get() 方法是可阻塞的, 如果在get()的过程中,线程还没有执行结束,get()方法将会阻塞;
- 同时Future提供了可以设置超时的get()方法,如果指定时间内没有get到结果,将会抛出一个TimeoutException异常后退出;
- 如果线程在执行的过程中抛出了异常,会将该异常包装为ExecutionException后重新抛出,此时可以通过getCause() 来获得被封装的初始异常;
- get()方法的异常处理代码需要处理两个问题: 1. 任务执行的过程中抛出了一个异常; 2. 调用get的线程在获得结果之前被中断;

### Future 和FutureTask之间的关系

 FutureTask 是Future的一个实现，Future 可实现 Runnable，所以可通过 threadPool 来执行。例如，可用下列内容替换上面带有 submit 的构造(为简化，返回String类型)；

```java
public class FutureTask<V> implements RunnableFuture<V> 
public interface RunnableFuture<V> extends Runnable, Future<V>
```

### 如何使用FutureTask类

FutureTask的构造函数有两个，一个是接受Callable接口的，另一个是实现Runnable接口

```java
public FutureTask(Callable<V> callable)  
public FutureTask(Runnable runnable, V result)
```

其中，第一中是最常见的使用方法，第二种比较少用，使用步骤：

1. 创建task类实现Callable\<V\>接口；
2. 实例化FutureTask类

```
new FutureTask<ResultType> task = new FutureTask<ResultType>();
```

1. 通过ExectorSrevice.submit()方法来提交任务；
2. 通过task.get()来获取执行结果，返回的类型为ResultType；

```java
class computeCallable implements Callable<String> {
    private int id;
    public computeCallable(int id) {
        this.id = id;
    }
    
    @Override
    public String call() {
        String rtn = "id: " + id + " I am computing, computing, computingtingting....";
        return rtn;
    }
}

class computeRunnable implements Runnable {
    private int id;
    public computeRunnable(int id) {
        this.id = id;
    }

    @Override
    public void run() {
        String rtn = "id: " + id + " I am computing, computing, computingtingting....";
        System.out.println(rtn);
    }
}

public class myfuturetask {
    @Test
    public void test0(){
        ExecutorService exec = Executors.newCachedThreadPool();
        List<FutureTask<String>> task = new ArrayList<FutureTask<String>>();
        List<String> result = new ArrayList<String>();
        for(int i = 0; i < 5; i++) {
            task.add(new FutureTask<String>(new computeCallable(i)));
        }

        for(FutureTask<String> ft : task) {
            exec.submit(ft);
        }
      	//或者使用exec.invokeAll(List<xxx>), 这样就不用使用for循环了；
        System.out.println("doing something else");
        for(FutureTask<String> ft : task) {
            try {
                System.out.println(ft.get());
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
        }
    }

    @Test
    public void test1(){
        ExecutorService exec = Executors.newCachedThreadPool();
        List<FutureTask<String>> task = new ArrayList<FutureTask<String>>();
        List<String> result = new ArrayList<String>();
        for(int i = 0; i < 5; i++) {
            result.add(String.valueOf(i));
            task.add(new FutureTask(new computeRunnable(i),result.get(i)));
        }

        for(FutureTask<String> ft : task) {
            exec.submit(ft);
        }

        System.out.println("doing something else");
        for(FutureTask<String> ft : task) {
            int i = 0;
            while(!ft.isDone());
            try {
                System.out.println(ft.get());
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (ExecutionException e) {
                e.printStackTrace();
            }
        }
    }
}
```

创建FutureTask的两种方法：

注意： FutureTask 是一个类，不是接口， 不过FutureTask实现了Runnable接口和Future接口；

1. 实例化一个FutureTask类，然后通过ExecutorService 的submit（）方法来执行该类， 然后就就可以通过该实例调用Future的相关接口来检测该线程的执行状态；

```java
FutureTask<String> task = new FutureTask<String>(new Callable<String>() {
    @Override
    public String call() throws Exception {
        TimeUnit.MILLISECONDS.sleep(new Random(System.currentTimeMillis()).nextInt(10)*100);
        System.out.println("Task completed");
        return "done...";
    }
});

ExecutorService exec = Executors.newCachedThreadPool();
exec.submit(task);
task.get(500, TimeUnit.MILLISECONDS);
```

1. 你的task类实现callable接口（实现其中的call()，然后直接ExecutorService的submit方法运行该Task类，submit()方法返回Future<T> ，通过返回的Future 可以调用future相关的方法来获取线程执行的具体情况；

```java
ExecutorService exec = Executors.newCachedThreadPool();
Future<String> result = exec.submit(new Task(100, 1000));
result.get();
```

1. 类似于二者结合的一种方法：

```java
FutureTask<String> task = new FutureTask<String>(new Task(i,1000));
exec.submit(task);
```

### 使用Future/FutureTask额外的好处(对线程生命周期的管理)

由于使用Future和FutureTask是可以通过cancell()来中断的，这也是他的优点之一，如果某个线程的执行超出了你允许的时间，则可以主动通过代码来将这个线程终端；

------

额外需要注意的地方：

在使用FutureTask +Runnable 来执行线程的时候需要传入来两个参数，其中第二个参数值将作为线程最终调用的结果返回：即再线程执行结束后调用get()方法，返回的是传入FutureTask的第二个参数；

```java
public FutureTask(Runnable runnable, V result) {  
      this.callable = Executors.callable(runnable, result);  
      this.state = NEW;       // ensure visibility of callable  
}
```

```java
public static <T> Callable<T> callable(Runnable task, T result) {  
       if (task == null)  
           throw new NullPointerException();  
       return new RunnableAdapter<T>(task, result);  
}
```

```java
static final class RunnableAdapter<T> implements Callable<T> {  
      final Runnable task;  
      final T result;  
      RunnableAdapter(Runnable task, T result) {  
          this.task = task;  
          this.result = result;  
      }  
      public T call() {  
          task.run();  
          return result;  
      }  
}  
```

