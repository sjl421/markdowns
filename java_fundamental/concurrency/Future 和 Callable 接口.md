# Future 和 Callable 接口

java中通过Runnable接口来创建线程的缺点是无法返回结果和抛出受检查的异常, 对与需要返回计算结果和抛出受检查异常的线程来说,可以使用Future和Callable接口来实现:

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
  V get(long timeout, TimeUnit unit) throws InterruptException, ExecutionException,CalcellationException, TimeoutExeception;	//设置超时的get()方法;
}
```

然后通过Future<T> ExecutorService.submit(Callable<T> task) 来提交线程;

需要注意的是: 

* get() 方法是可阻塞的, 如果在get()的过程中,线程还没有执行结束,get()方法将会阻塞;
* 同时Future提供了可以设置超时的get()方法,如果指定时间内没有get到结果,将会抛出一个TimeoutException异常后退出;
* 如果线程在执行的过程中跑出了异常,会将该异常包装为ExecutionException后重新抛出,此时可以通过getCause() 来获得被封装的初始异常;
* get()方法的异常处理代码需要处理两个问题: 1. 任务执行的过程中抛出了一个异常; 2. 调用get的线程在获得结果之前被中断;