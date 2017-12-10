#ScheduledThreadPoolExecutor

* ScheduledThreadPoolExecutor是一个size固定的pool,corePoolSize是在创建的时候就固定了的;
* 分为 scheduleAtFixedRate 和 scheduleWithFixedDelay 两种方法;
* threadPoolExecutor 内部的任务队列是无界的;
* ScheduledFutureTask
* DelayedWorkQueue



```
public ScheduledFuture<?> schedule(Runnable command,
								   long delay,
								   TimeUnit unit) {
	if (command == null || unit == null)
		throw new NullPointerException();
	RunnableScheduledFuture<?> t = decorateTask(command,
		new ScheduledFutureTask<Void>(command, null,
									  triggerTime(delay, unit)));
	delayedExecute(t);
	return t;
}
```

