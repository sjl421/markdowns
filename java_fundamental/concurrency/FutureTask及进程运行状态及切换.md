# FutureTask及进程运行状态及切换

FutureTask的继承关系图:

![FutureTask](G:\snap\FutureTask.png)

由继承关系来看,FutureTask接口实现了RunnableFuture接口,而RunnableFuture接口同时继承了Runnalbe接口和Future接口,这最终就造成了FutureTask即拥有了Runnable接口和Callabel接口的可执行性,又拥有了Future接口的作为执行结果的返回性,所以可以: 

实例化一个FutureTask类，然后通过ExecutorService 的submit（）方法来执行该类， 然后就就可以通过该实例调用Future的相关接口来检测该线程的执行状态; 否则如果使用Future接口话在submit()的时候还要显式的接受submit的返回结果;

FutureTask的构造函数:

```
public FutureTask(Callable<V> callable)  
public FutureTask(Runnable runnable, V result)
```

### 线程运行状态

FutureTask中定义了

```
private volatile int state;
private static final int NEW          = 0;
private static final int COMPLETING   = 1;
private static final int NORMAL       = 2;
private static final int EXCEPTIONAL  = 3;
private static final int CANCELLED    = 4;
private static final int INTERRUPTING = 5;
private static final int INTERRUPTED  = 6;
```

状态之间的跳转

```
* Possible state transitions:
* NEW -> COMPLETING -> NORMAL
* NEW -> COMPLETING -> EXCEPTIONAL
* NEW -> CANCELLED
* NEW -> INTERRUPTING -> INTERRUPTED
```

```
public boolean cancel(boolean mayInterruptIfRunning) {
	if (!(state == NEW &&
		  UNSAFE.compareAndSwapInt(this, stateOffset, NEW,
			  mayInterruptIfRunning ? INTERRUPTING : CANCELLED)))
		return false;
	try {    // in case call to interrupt throws exception
		if (mayInterruptIfRunning) {
			try {
				Thread t = runner;
				if (t != null)
					t.interrupt();
			} finally { // final state
				UNSAFE.putOrderedInt(this, stateOffset, INTERRUPTED);
			}
		}
	} finally {
		finishCompletion();
	}
	return true;
}
```

未完待续...