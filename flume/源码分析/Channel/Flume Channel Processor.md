#Flume Channel Processor

> 搞懂channel processor, channel, 和 channel selector之间的关系;

* channel processor  暴露出了向channel中put Event的操作;
* channel Processor的主要功能是将event(单个的event)/events(list, 批量的操作)放入到对应的channel中,其中channel 可分为required channel 和 optional channel,
* 为了区分 required channel 和 optional channel, channel processor中还实例化了channel selector来确定那些channel是必选的,哪些channel是可选的;
* channel processor 实例化了interceptorChain, channelProcessor的操作中有对interceptorChain的基本操作操作;

### 配置InterceptorChain

```java
  /* 初始化 interceptor chain */
  public void initialize() {
    interceptorChain.initialize();
  }

  /*  关闭 interceptor chain */
  public void close() {
    interceptorChain.close();
  }

  /* channelProcessor的configure方法只configure了自己的InterceptorChain对象 */
  @Override
  public void configure(Context context) {
    configureInterceptors(context);
  }

  // WARNING: throws FlumeException (is that ok?)
  /* 从 context 中配置Interceptor, 入参为 Context 对象 */
  private void configureInterceptors(Context context) {
    List<Interceptor> interceptors = Lists.newLinkedList();
    String interceptorListStr = context.getString("interceptors", "");
    if (interceptorListStr.isEmpty()) {
      return;
    }
    String[] interceptorNames = interceptorListStr.split("\\s+");
    Context interceptorContexts =
        new Context(context.getSubProperties("interceptors."));
    // run through and instantiate all the interceptors specified in the Context
    InterceptorBuilderFactory factory = new InterceptorBuilderFactory();
    for (String interceptorName : interceptorNames) {
      Context interceptorContext = new Context(
          interceptorContexts.getSubProperties(interceptorName + "."));
      String type = interceptorContext.getString("type");
      if (type == null) {
        LOG.error("Type not specified for interceptor " + interceptorName);
        throw new FlumeException("Interceptor.Type not specified for " +
            interceptorName);
      }
      try {
        Interceptor.Builder builder = factory.newInstance(type);
        builder.configure(interceptorContext);
        interceptors.add(builder.build());
      } catch (ClassNotFoundException e) {
        LOG.error("Builder class not found. Exception follows.", e);
        throw new FlumeException("Interceptor.Builder not found.", e);
      } catch (InstantiationException e) {
        LOG.error("Could not instantiate Builder. Exception follows.", e);
        throw new FlumeException("Interceptor.Builder not constructable.", e);
      } catch (IllegalAccessException e) {
        LOG.error("Unable to access Builder. Exception follows.", e);
        throw new FlumeException("Unable to access Interceptor.Builder.", e);
      }
    }
    interceptorChain.setInterceptors(interceptors);
  }
```

### 将Event/Events put到channel中

```java
# 将 events(list) 批量的放入到channel中
public void processEventBatch(List<Event> events);
# 将单个 events 的放入到channel中
public void processEvent(Event event);
```

在将event放入到channel中时, channel的属性可分为required channel和 optional channel, 通过channel processor 中实例化的channel Selector来区分;

```java
  public void processEventBatch(List<Event> events) {
    Preconditions.checkNotNull(events, "Event list must not be null");

    events = interceptorChain.intercept(events);

    //存放的是要放大reqChannel中List<Event>
    Map<Channel, List<Event>> reqChannelQueue =
        new LinkedHashMap<Channel, List<Event>>();

    //存放的是要放大optChannel中List<Event>
    Map<Channel, List<Event>> optChannelQueue =
        new LinkedHashMap<Channel, List<Event>>();

    //将events放到对应channel的eventQueue中
    for (Event event : events) {
      List<Channel> reqChannels = selector.getRequiredChannels(event);

      //将events放到对应reqChannel的eventQueue中
      for (Channel ch : reqChannels) {
        List<Event> eventQueue = reqChannelQueue.get(ch);
        if (eventQueue == null) {
          eventQueue = new ArrayList<Event>();
          reqChannelQueue.put(ch, eventQueue);
        }
        eventQueue.add(event);
      }

      List<Channel> optChannels = selector.getOptionalChannels(event);

      //将events放到对应optChannel的eventQueue中
      for (Channel ch : optChannels) {
        List<Event> eventQueue = optChannelQueue.get(ch);
        if (eventQueue == null) {
          eventQueue = new ArrayList<Event>();
          optChannelQueue.put(ch, eventQueue);
        }

        eventQueue.add(event);
      }
    }

    // Process required channels
    for (Channel reqChannel : reqChannelQueue.keySet()) {
      Transaction tx = reqChannel.getTransaction();
      Preconditions.checkNotNull(tx, "Transaction object must not be null");
      try {
        tx.begin();

        List<Event> batch = reqChannelQueue.get(reqChannel);

        for (Event event : batch) {
          reqChannel.put(event);
        }

        tx.commit();
      } catch (Throwable t) {
        tx.rollback();
        if (t instanceof Error) {
          LOG.error("Error while writing to required channel: " + reqChannel, t);
          throw (Error) t;
        } else if (t instanceof ChannelException) {
          throw (ChannelException) t;
        } else {
          throw new ChannelException("Unable to put batch on required " +
              "channel: " + reqChannel, t);
        }
      } finally {
        if (tx != null) {
          tx.close();
        }
      }
    }

    // Process optional channels
    for (Channel optChannel : optChannelQueue.keySet()) {
      Transaction tx = optChannel.getTransaction();
      Preconditions.checkNotNull(tx, "Transaction object must not be null");
      try {
        tx.begin();

        List<Event> batch = optChannelQueue.get(optChannel);

        for (Event event : batch) {
          optChannel.put(event);
        }

        tx.commit();
      } catch (Throwable t) {
        tx.rollback();
        LOG.error("Unable to put batch on optional channel: " + optChannel, t);
        if (t instanceof Error) {
          throw (Error) t;
        }
      } finally {
        if (tx != null) {
          tx.close();
        }
      }
    }
  }
```

```java
  /**
   * Attempts to {@linkplain Channel#put(Event) put} the given event into each
   * configured channel. If any {@code required} channel throws a
   * {@link ChannelException}, that exception will be propagated.
   * <p>
   * <p>Note that if multiple channels are configured, some {@link Transaction}s
   * may have already been committed while others may be rolled back in the
   * case of an exception.
   *
   * @param event The event to put into the configured channels.
   * @throws ChannelException when a write to a required channel fails.
   */
  /*  将单个 events 放到对应channel中 */
  public void processEvent(Event event) {

    event = interceptorChain.intercept(event);
    if (event == null) {
      return;
    }

    // Process required channels
    List<Channel> requiredChannels = selector.getRequiredChannels(event);
    for (Channel reqChannel : requiredChannels) {
      Transaction tx = reqChannel.getTransaction();
      Preconditions.checkNotNull(tx, "Transaction object must not be null");
      try {
        tx.begin();

        reqChannel.put(event);

        tx.commit();
      } catch (Throwable t) {
        tx.rollback();
        if (t instanceof Error) {
          LOG.error("Error while writing to required channel: " + reqChannel, t);
          throw (Error) t;
        } else if (t instanceof ChannelException) {
          //在放入requiredChannel时发生异常，将重新thrown异常,此次函数将返回，后续对optional channel的处理也不会执行
          throw (ChannelException) t;
        } else {
          throw new ChannelException("Unable to put event on required " +
              "channel: " + reqChannel, t);
        }
      } finally {
        if (tx != null) {
          tx.close();
        }
      }
    }

    // Process optional channels
    /**
     * 依据该方法的代码逻辑，只有所有required channel都处理成功之后才会将event向optional channel中put
     * 并且如果optional channel 处理失败，不会跑出ChannelException(意味着上层也不会做重试动作)
     */
    List<Channel> optionalChannels = selector.getOptionalChannels(event);
    for (Channel optChannel : optionalChannels) {
      Transaction tx = null;
      try {
        tx = optChannel.getTransaction();
        tx.begin();

        optChannel.put(event);

        tx.commit();
      } catch (Throwable t) {
        tx.rollback();
        LOG.error("Unable to put event on optional channel: " + optChannel, t);
        if (t instanceof Error) {
          throw (Error) t;
        }
        //在放入optionalChannel时发生ChannelException异常，将忽略该异常
      } finally {
        if (tx != null) {
          tx.close();
        }
      }
    }
  }
```

