# Java 获取当前类名和方法名

## 获取方法名

```java
Thread.currentThread().getStackTrace()[1].getMethodName();
```

//具体使用数组的那个元素和JVM的实现有关，我在SUN JDK6下面测试的是第二个元素，具体说明可以查看Thread.getStackTrace方法的javadoc  



## 获取当前类名

```java
this.getClass().getName();
```



