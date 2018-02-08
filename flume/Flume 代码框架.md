# Flume 代码框架

需要梳理清楚几条线：

1. flume启动时是如何如何读取配置信息并生效的？
2. channel框架
3. sink框架
4. source框架
5. channel，sink，source组合起立的框架





Application.java

主要来控制所有components的生命启动和停止，配置文件的读取，解析启动参数；

Application的主要的启动流程是：

`public static void main(String[] args) ` 是整个Application的main方法，也是真个flume的入口方法，主要实现的功能有：

1. 解析命令行传入的启动参数；
2. （省略不重要的），从制定的配置文件中读取配置信息，这里只关注了



#Flume的启动

Application.java

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

