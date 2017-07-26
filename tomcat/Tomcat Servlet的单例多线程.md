# Tomcat Servlet的单例多线程

测试代码:

```java
@Controller
@ResponseBody
@RequestMapping(value = "/concurrent")
public class ConcurrentController {

    @RequestMapping(value = "/sleep", method = RequestMethod.GET)
    public Object thread1(@RequestParam(value = "sleep", required = false) String sleep,
                          HttpServletRequest request) {
        CommonResponse cr = new CommonResponse();
        System.out.println(Thread.currentThread().getName() + " holding...");
        boolean beSleep = "true".equals(sleep);
        doSleep(beSleep);
        System.out.println("do not sleep");
        return cr;
    }

    private synchronized void doSleep(boolean sleep) {
        while (sleep) {
            try {
                System.out.println("sleeping...");
                TimeUnit.MILLISECONDS.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

首先,需要了解的是tomcat servlet是单例多线程的, 即一个部署在tomcat中的web服务默认在tomcat中只会有一个实例存在, 它通过tomcat的Dispatcher xxx 通过多线程来响应多个客户端的请求.

由于有上面一个前提的存在,所以在编写web服务的时候需要考虑在并发访问时可能会存在的并发问题;

一般来讲,由于局部变量是线程私有的,所以这种类型的代码在并发访问时不存在安全性的问题, 但是不得不说的是某些比较特殊的方法或者资源是可能同时被多个线程访问的, 对于这类的代码就需要自己的在代码中采取恰当的措施来解决的并发的问题了;

1. test case1: 多个请求请求同一个api, doSleep()方法不加 synchronized关键字;

request 1: sleep=true, 让这个请求陷入死循环,永不返回; 同时request 2 请求相同的api, sleep=false,看能不能马上返回;

结论:方法doSleep() 不加synchronized关键字, 由于多线程,所以即便是第一个请求的没有返回,但是这个并不印象第二个线程的执行,所以第二个请求会正确返回;



2. test case 2: doSleep()加上 synchronized关键字, 由于这个方法同时只允许一个线程访问, 所以如果第一个请求陷入死循环,那么第二个请求在执行到这个方法时将无法获得这个方法的锁, 会被阻塞,所以第二个请求不会正确返回;