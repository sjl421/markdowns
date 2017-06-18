# java_type_info

1. `xxx.getMethods();`

```java
Method[] methods = c.getMethods();	// 获取一个类中所有的方法
Constructor[] ctors = c.getConstructors();	//获取一个类所有的构造器
```

2. java 动态代理的实现

```java
package org.sun.java.testclass;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

class DynamicProxyHandler implements InvocationHandler {
    private Object proxied;
    public DynamicProxyHandler(Object proxied) {
        this.proxied = proxied;
    }
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("****** proxy" + proxy.getClass() +
        ": method: " + method + ", args: " + args);
        if(args != null) {
            for(Object arg : args) {
                System.out.println("  " + arg);
            }
        }
        return method.invoke(proxied,args);
    }
}

public class SimpleDynamicProxy {
    public static void consumer(Interface iface) {
        iface.doSomething();
        iface.somethingElse("bonobo");
    }

    public static void main(String[] args) {
        RealObject real = new RealObject();
        consumer(real);

        Interface proxy = (Interface) Proxy.newProxyInstance(
                Interface.class.getClassLoader(),
                new Class[] {Interface.class},
                new DynamicProxyHandler(real)
        );
        consumer(proxy);
    }
}
```

> 通过调用静态方法 Proxy.newProxyInstance()可以创建动态代理，传入的参数有：1、  一个类加载器;2、一个你希望代理实现的接口列表；3、以及InvacationHandler接口的一个实现。动态代理可以将所有调用重定向到调用处理器，因此通常会向调用处理器的构造器传递给一个“实际”对象的引用，从而使用调用处理器再执行起中介任务时，可以讲请求转发；

