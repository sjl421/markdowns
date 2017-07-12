# FutureTask

### 为什么会有Future和FutureTask类

场景： 在某种场景下主进程需要长时间的计算，此时可以使用FutureTask类来执行这个复杂的计算，然后主进程继续去执行一些其他的操作，然后等主进程处理完自己的操作之后再去通过FutureTask去取计算的结果，如果此时FutureTask类还没有计算完成，则FutureTask的get()会阻塞直到线程执行结束，得到结果；

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

3. 通过ExectorSrevice.submit()方法来提交任务；
4. 通过task.get()来获取执行结果，返回的类型为ResultType；

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

### 使用Future/FutureTask额外的好处

由于使用Future和FutureTask是可以通过cancell()来中断的，这也是他的优点之一，如果某个线程的执行超出了你允许的时间，则可以主动通过代码来将这个线程终端；

---

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

