#单例模式的Double-checked Locking (DCL)问题

Double-checked Locking (DCL)用来在lazy initialisation 的单例模式中避免同步开销的一个方法。

下面是这么做的一个例子。

```java
public synchronized static MyFactory getFactory() {  
    if (instance == null)  
      instance = new MyFactory();  
    return instance;  
  }  
}  
```

上面的例子是完全正确的。但是考虑到所有的Read操作也需要同步，为了避免昂贵的同步开销，似乎有如下做法：

```java
public class MyBrokenFactory {  
  private static MyFactory instance;  
  private int field1, field2 ...  
  
  public static MyBrokenFactory getFactory() {  
    // This is incorrect: don't do it at home, kids!  
    if (instance == null) {  
      synchronized (MyBrokenFactory.class) {  
        if (instance == null)  
          instance = new MyFactory();  
      }  
    }  
    return instance;  
  }  
  
  private MyBrokenFactory() {  
    field1 = ...  
    field2 = ...  
  }  
}  
```

但是上面的做法是不正确的，考虑2个线程同时调用MyBrokenFactory.getFactory()，线程2在线程1完成对象的初始化之前就可能得到了对象的引用。

| Thread 1: 'gets in first' and starts creating instance. | Thread 2: gets in just as Thread 1 has written the object reference to memory, but before it has written all thefields. |
| ---------------------------------------- | ---------------------------------------- |
| 1. Is instance null? Yes.2. Synchronize on class. 3. Memory is allocated for instance. 4. Pointer to memory saved into instance. 7. Values for field1 and field2 are written to memory allocated for object. | 5. Is instance null? No. 6. instance is non-null, but field1 and field2 haven't yet been set! This thread sees invalid values for field1 and field2! |

如果解决上面的问题呢？

## **方法1：使用class loader**

```java
public class MyFactory {  
  private static final MyFactory instance = new MyFactory();  
  
  public static MyFactory getInstance() {  
    return instance;  
  }  
  
  private MyFactory() {}  
}  
```

如果需要处理异常情况，

```java
public class MyFactory {  
  private static final MyFactory instance;  
  
  static {  
    try {  
      instance = new MyFactory();  
    } catch (IOException e) {  
      throw new RuntimeException("Darn, an error's occurred!", e);  
    }  
  }  
  
  public static MyFactory getInstance() {  
    return instance;  
  }  
  
  private MyFactory() throws IOException {  
    // read configuration files...  
  }  
}  
```

但是这样就失去了lazy initialisation带来的好处，Java5以后还有一种办法，

## **方法2：使用DCL+volatile**

```java
 
public class MyFactory {  
  private static volatile MyFactory instance;  
  
  public static MyFactory getInstance(Connection conn)  
       throws IOException {  
    if (instance == null) {  
      synchronized (MyFactory.class) {  
        if (instance == null)  
          instance = new MyFactory(conn);  
      }  
    }  
    return instance;    
  }  
  
  private MyFactory(Connection conn) throws IOException {  
    // init factory using the database connection passed in  
  }  
}
```

JAVA5以后如果申明实例引用为volatile，那么DCL就是OK的。

JAVA5以后，访问一个volatile的变量具有synchronized 的语义。换句话说，JAVA5保证unsycnrhonized volatile read 会在写之后发生。（Accessing a volatile variable has the semantics of synchronization as of Java 5. In other words Java 5 ensures that the unsycnrhonized volatile readmust happen after the write has taken place。）

关于volatile的详细说明

http://blog.csdn.net/fw0124/article/details/6669984

## 方法3：Factory类的所有字段都是final字段

JAVA5之后，如果在constructor中对final字段赋值，JVM保证先把这些值提交到内存，然后才会更新内存中的对象引用。

换句话说，另外一个能看到这个对象的线程不能看到没有初始化过的final字段。

在Factory类的所有字段都是final字段这种情况下，我们实际上没有必要申明factory实例为volatile。

In Java 5, a change was made to the definition of final fields. Where the values of these fields are set in the constructor, the JVM ensures that these values are committed to main memorybefore the object reference itself. In other words, another thread that can "see" the objectcannot ever see uninitialised values of its final fields. In that case, we wouldn't actually need to declare the instance reference as volatile.

（原文http://javamex.com/tutorials/double_checked_locking_fixing.shtml）

## **方法4：使用静态内部类**

在类的初始化期间，JVM会去获取一个锁。这个锁可以同步多个线程对同一个类的初始化。基于这个特性，可以实现另一种线程安全的延迟初始化方案。

```java
public class SingleFactory {  
    private static class InstanceHolder {  
        public static final SingleFactory instance = new SingleFactory();  
    }  
      
  
    public static SingleFactory getInstance() {  
        return InstanceHolder.instance;  
    }  
  
    private SingleFactory() {  
    }  
}  
```

初始化一个类，包括执行这个类的静态初始化（static代码段）和初始化在这个类中声明的静态字段。
根据java语言规范，在首次发生下列任意一种情况时，一个类或接口类型T将被立即初始化：
\- T是一个类，而且一个T类型的实例被创建；
\- T是一个类，且T中声明的一个静态方法被调用；
\- T中声明的一个静态字段被赋值；
\- T中声明的一个静态字段被使用，而且这个字段不是一个常量字段；
\- T是一个顶级类（top level class，见java语言规范的§7.6），而且一个断言语句嵌套在T内部被执行。

以前转载了Java内存模型的系列文章，理解了这些文章，上面的内容就很好理解了。
[Java多线程 -- 深入理解JMM（Java内存模型） --（一）基础](http://blog.csdn.net/fw0124/article/details/6650483)
[Java多线程 -- 深入理解JMM（Java内存模型） --（二）重排序](http://blog.csdn.net/fw0124/article/details/6652305)
[Java多线程 -- 深入理解JMM（Java内存模型） --（三）顺序一致性](http://blog.csdn.net/fw0124/article/details/6653008)
[Java多线程 -- 深入理解JMM（Java内存模型） --（四）volatile](http://blog.csdn.net/fw0124/article/details/6655381)
[Java多线程 -- 正确使用Volatile变量](http://blog.csdn.net/fw0124/article/details/6669984)
[Java多线程 -- 深入理解JMM（Java内存模型） --（五）锁](http://blog.csdn.net/fw0124/article/details/6671447)
[Java多线程 -- 深入理解JMM（Java内存模型） --（六）final](http://blog.csdn.net/fw0124/article/details/6692022)
[Java多线程 -- 深入理解JMM（Java内存模型） --（七）总结](http://blog.csdn.net/fw0124/article/details/6738266)

> 同时可以通过http://cmsblogs.com/?p=2161来进一步的了解