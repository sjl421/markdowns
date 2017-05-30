# Spring 注解

1.  @PostConstruct  

   Spring的@PostConstruct注解在方法上，表示此方法是在Spring实例化该Bean之后马上执行此方法，之后才会去实例化其他Bean，并且一个Bean中@PostConstruct注解的方法可以有多个。可以用在实例化该Bean的时候做一些初始化的工作；

2. @PreDestory

   在销毁Bean之前做一些清理的工作；

