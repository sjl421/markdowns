#Flume 的启动和停止

###flume-ng

 整个flume的启动入口为apache-flume/bing/flume-ng，这是一个shell脚本，该脚本中读取了通过文件配置的jvm的一些启动参数，然后也提示除了启动的入口位置是flume的Application类，想搞清楚flume是如何启动的，以及flume时的一些jvm参数配置，应该从这个脚本入手。

###Application.java

主要来控制所有components的生命启动和停止，配置文件的读取，解析启动参数；

Application的主要的启动流程是：

`public static void main(String[] args) ` 是整个Application的main方法，也是真个flume的入口方法，主要实现的功能有：

1. 解析命令行传入的启动参数；
2. （省略不重要的），从制定的配置文件中读取配置信息，生成flume的配置类;
3. 启动flume各个组件；
4. 向jvm注册用于flume推出的钩子函数;

****

读取配置文件，根据配置文件生成相应的配置信息，最后启动flume的各个组件；

```
  public synchronized void start() {
    for (LifecycleAware component : components) {
      //启动所有组件的入口
      supervisor.supervise(component,
          new SupervisorPolicy.AlwaysRestartPolicy(), LifecycleState.START);
    }
  }
```

Flume 是如何启动所有的组件的:

Flume的所有组件都通过实现LifecycleAware接口来实现组件的生命周期管理，在Applicaton.start()方法中，通过调用LifecycleSupervisor.supervise()方法来启动所有的组件;关于LifecycleSupervisor关键：

1. LifecycleSupervisor 实例化了ScheduledThreadPoolExecutor和内部类MonitorRunnable。其中，MonitorRunnable是一个Runnable接口，主要是根据各个组件的desiredState来调用组件相应的生命周期方法，这其中就有组件的start()方法;
2. 通过ScheduledThreadPoolExecutor 来定期的调用MonitorRunnable方法来维护及更新flume各个组件的生命周期。

```
public class LifecycleSupervisor implements LifecycleAware {

  private Map<LifecycleAware, Supervisoree> supervisedProcesses;
  private Map<LifecycleAware, ScheduledFuture<?>> monitorFutures;

  /**
   * ScheduledThreadPoolExecutor 线程池，corePoolSize=10，MaximumPoolSize=20
   * 定期的来检测flume 各个 component的生命周期
   * 是flume 各个组件启停的关键
   */
  private ScheduledThreadPoolExecutor monitorService;

  private LifecycleState lifecycleState;
  
  public LifecycleSupervisor() {
    lifecycleState = LifecycleState.IDLE;
    supervisedProcesses = new HashMap<LifecycleAware, Supervisoree>();
    monitorFutures = new HashMap<LifecycleAware, ScheduledFuture<?>>();
    monitorService = new ScheduledThreadPoolExecutor(10,
        new ThreadFactoryBuilder().setNameFormat(
            "lifecycleSupervisor-" + Thread.currentThread().getId() + "-%d")
            .build());
    //线程池最大的数量是20
    monitorService.setMaximumPoolSize(20);
    monitorService.setKeepAliveTime(30, TimeUnit.SECONDS);
    purger = new Purger();
    needToPurge = false;
  }
  //重要的方法
  public synchronized void supervise(LifecycleAware lifecycleAware,
      SupervisorPolicy policy, LifecycleState desiredState) {
    if (this.monitorService.isShutdown()
        || this.monitorService.isTerminated()
        || this.monitorService.isTerminating()) {
      throw new FlumeException("Supervise called on " + lifecycleAware + " " +
          "after shutdown has been initiated. " + lifecycleAware + " will not" +
          " be started");
    }

    Supervisoree process = new Supervisoree();
    process.status = new Status();

    process.policy = policy;
    process.status.desiredState = desiredState;
    process.status.error = false;

    MonitorRunnable monitorRunnable = new MonitorRunnable();
    monitorRunnable.lifecycleAware = lifecycleAware;
    monitorRunnable.supervisoree = process;
    monitorRunnable.monitorService = monitorService;

    supervisedProcesses.put(lifecycleAware, process);

    //添加到全局monitorService(ScheduledThreadPoolExecutor)中
    ScheduledFuture<?> future = monitorService.scheduleWithFixedDelay(
        monitorRunnable, 0, 3, TimeUnit.SECONDS);
    monitorFutures.put(lifecycleAware, future);
  }
```

```
  public static class MonitorRunnable implements Runnable {

    /**
     * MonitorRunnable保留了对外层ScheduledExecutorService的引用，
     * 但是感觉在MonitorRunnable内部并没有什么用
     */
    public ScheduledExecutorService monitorService;
    /**
     * lifecycleAware 就是flume的各种组件，channel, source, sink 等，
     * 这些组件实现了lifecycleAware的接口，实现了生命周期的管理，也就意味着
     * 这些组件是通过lifecycleAware来实现启动和停止的
     */
    public LifecycleAware lifecycleAware;
    //用来记录component运行状态的一个辅助类
    public Supervisoree supervisoree;

    @Override
    public void run() {
      logger.debug("checking process:{} supervisoree:{}", lifecycleAware,
          supervisoree);

      long now = System.currentTimeMillis();

      try {
        if (supervisoree.status.firstSeen == null) {
          logger.debug("first time seeing {}", lifecycleAware);

          supervisoree.status.firstSeen = now;
        }

        supervisoree.status.lastSeen = now;
        synchronized (lifecycleAware) {
          if (supervisoree.status.discard) {
            // Unsupervise has already been called on this.
            logger.info("Component has already been stopped {}", lifecycleAware);
            return;
          } else if (supervisoree.status.error) {
            logger.info("Component {} is in error state, and Flume will not"
                + "attempt to change its state", lifecycleAware);
            return;
          }

          supervisoree.status.lastSeenState = lifecycleAware.getLifecycleState();

          if (!lifecycleAware.getLifecycleState().equals(
              supervisoree.status.desiredState)) {

            logger.debug("Want to transition {} from {} to {} (failures:{})",
                new Object[] { lifecycleAware, supervisoree.status.lastSeenState,
                    supervisoree.status.desiredState,
                    supervisoree.status.failures });

            //根据component 的 desiredState来做调用相应的lifecycleAware的方法
            switch (supervisoree.status.desiredState) {
              case START:
                try {
                  lifecycleAware.start();
                } catch (Throwable e) {
                  logger.error("Unable to start " + lifecycleAware
                      + " - Exception follows.", e);
                  if (e instanceof Error) {
                    // This component can never recover, shut it down.
                    supervisoree.status.desiredState = LifecycleState.STOP;
                    try {
                      lifecycleAware.stop();
                      logger.warn("Component {} stopped, since it could not be"
                          + "successfully started due to missing dependencies",
                          lifecycleAware);
                    } catch (Throwable e1) {
                      logger.error("Unsuccessful attempt to "
                          + "shutdown component: {} due to missing dependencies."
                          + " Please shutdown the agent"
                          + "or disable this component, or the agent will be"
                          + "in an undefined state.", e1);
                      supervisoree.status.error = true;
                      if (e1 instanceof Error) {
                        throw (Error) e1;
                      }
                      // Set the state to stop, so that the conf poller can
                      // proceed.
                    }
                  }
                  supervisoree.status.failures++;
                }
                break;
              case STOP:
                try {
                  lifecycleAware.stop();
                } catch (Throwable e) {
                  logger.error("Unable to stop " + lifecycleAware
                      + " - Exception follows.", e);
                  if (e instanceof Error) {
                    throw (Error) e;
                  }
                  supervisoree.status.failures++;
                }
                break;
              default:
                logger.warn("I refuse to acknowledge {} as a desired state",
                    supervisoree.status.desiredState);
            }

            if (!supervisoree.policy.isValid(lifecycleAware, supervisoree.status)) {
              logger.error(
                  "Policy {} of {} has been violated - supervisor should exit!",
                  supervisoree.policy, lifecycleAware);
            }
          }
        }
      } catch (Throwable t) {
        logger.error("Unexpected error", t);
      }
      logger.debug("Status check complete");
    }
  }
```

```
  private static class Supervisoree {

    public SupervisorPolicy policy;
    public Status status;
  }
  
  public static class Status {
    public Long firstSeen;
    public Long lastSeen;
    public LifecycleState lastSeenState;
    public LifecycleState desiredState;	//component 期望的状态
    public int failures;
    public boolean discard;
    public volatile boolean error;
  }
```

Flume组件的停止：

Flume并没有提供一个合适的停止的方式，而正常的停止方式是通过kill方式来杀死jvm进程来达到停止flume的目的， 而是在Application的底部的功过java的Runtime.getRuntime().addShutdownHook()来注册一个钩子函数，这样jvm会在退出的时候会调用注册的钩子函数，在Application中注册的钩子函数调用的正是Application.stop()方法。

```
      final Application appReference = application;
      /**
       * flume的退出机制，通过注册JVM的ShutdownHook()来再jvm退出时调用application.stop
       * 来停止flume的components,jvm在某些情况下退出时会调用ShutdownHook函数，但是
       * kill -9 是不会执行ShutdownHook函数的
       */
      Runtime.getRuntime().addShutdownHook(new Thread("agent-shutdown-hook") {
        @Override
        public void run() {
          appReference.stop();
        }
```

```
  public synchronized void stop() {
    supervisor.stop();
    if (monitorServer != null) {
      monitorServer.stop();
    }
  }
```

```
  #LifecycleSupervisor.java
  @Override
  public synchronized void stop() {

    logger.info("Stopping lifecycle supervisor {}", Thread.currentThread()
        .getId());

    if (monitorService != null) {
      //关闭ScheduledThreadPoll 线程池
      monitorService.shutdown();
      try {
        monitorService.awaitTermination(10, TimeUnit.SECONDS);
      } catch (InterruptedException e) {
        logger.error("Interrupted while waiting for monitor service to stop");
      }
      if (!monitorService.isTerminated()) {
        monitorService.shutdownNow();
        try {
          while (!monitorService.isTerminated()) {
            monitorService.awaitTermination(10, TimeUnit.SECONDS);
          }
        } catch (InterruptedException e) {
          logger.error("Interrupted while waiting for monitor service to stop");
        }
      }
    }

    for (final Entry<LifecycleAware, Supervisoree> entry : supervisedProcesses.entrySet()) {

      if (entry.getKey().getLifecycleState().equals(LifecycleState.START)) {
        //更新各个components的desiredState
        entry.getValue().status.desiredState = LifecycleState.STOP;
        //调用各个组件的stop()方法
        entry.getKey().stop();
      }
    }

    /* If we've failed, preserve the error state. */
    if (lifecycleState.equals(LifecycleState.START)) {
      lifecycleState = LifecycleState.STOP;
    }
    supervisedProcesses.clear();
    monitorFutures.clear();
    logger.debug("Lifecycle supervisor stopped");
  }
```

总结： 

通过是所有的组件都实现LifecycleAware来达到统一管理组件生命周期的做法感觉是很合适的做法;