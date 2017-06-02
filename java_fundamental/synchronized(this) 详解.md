# synchronized 详解

```java
package org.sun.test.cocurrent.synchronizedlock;
public class Thread2 {
    public void m4t1(){
        synchronized (this){
            int i = 5;
            while(i-- > 0){
                System.out.println(Thread.currentThread().getName() + ":" +i);
                try{
                    Thread.sleep(500);
                }catch(InterruptedException ie){
                }
            }
        }
    }

    public void m4t2(){
        int i = 5;
        while (i-- > 0){
            System.out.println(Thread.currentThread().getName() + ":" + i);
        }
        try{
            Thread.sleep(500);
        }catch (InterruptedException ie){

        }
    }

    public void m4t3(){
        synchronized (this){
            int i = 5;
            while (i-- > 0){
                System.out.println(Thread.currentThread().getName() + ":" + i);
            }
            try{
                Thread.sleep(500);
            }catch (InterruptedException ie){
            }
        }
    }

    public synchronized void m4t4(){
        int i = 5;
        while (i-- > 0){
            System.out.println(Thread.currentThread().getName() + ":" + i);
        }

        try{
            Thread.sleep(500);
        }catch (InterruptedException ie){

        }
    }

    public void m4t6(){
        Object syncObject = 0;
        synchronized (syncObject){
            int i = 5;
            while (i-- > 0){
                System.out.println(Thread.currentThread().getName() + ":" + i);
            }
            try{
                Thread.sleep(500);
            }catch (InterruptedException ie){
            }
        }
    }

    public static void m4t5(){
        synchronized (Thread2.class){
            int i = 5;
            while (i-- > 0){
                System.out.println(Thread.currentThread().getName() + ":" + i);
            }
            try{
                Thread.sleep(500);
            }catch (InterruptedException ie){
            }
        }
    }


    public static void main(String[] args) {
        final Thread2 myt2 = new Thread2();
        Thread t1 = new Thread(
                new Runnable(){
                    public void run(){
                        myt2.m4t1();
                    }
                },"t1"
        );

        Thread t2 = new Thread(
                new Runnable() {
                    public void run() {
//                    	myt2.m4t2();
//                    	myt2.m4t3();
//                    	myt2.m4t4();
//                        myt2.m4t5();
                        myt2.m4t6();
                    }
                },"t2"
        );
        t1.start();
        t2.start();
    }
}
```

//所有现象的解释都可以用最后的总结来解释：

---

# Chapter 2

```java
//case 1:
class Sync {  
  
    public synchronized void test() {  
        System.out.println("test开始..");  
        try {  
            Thread.sleep(1000);  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
        System.out.println("test结束..");  
    }  
}  
  
class MyThread extends Thread {  
  
    public void run() {  
        Sync sync = new Sync();  
        sync.test();  
    }  
}  
  
public class Main {  
  
    public static void main(String[] args) {  
        for (int i = 0; i < 3; i++) {  
            Thread thread = new MyThread();  
            thread.start();  
        }  
    }  
}  
/*
运行结果：
test开始..
test开始..
test开始..
test结束..
test结束..
test结束..
*/// 结论是没有实现想要的同步效果
```

```java
// case 2:
public void test() {  
    synchronized(this){  
        System.out.println("test开始..");  
        try {  
            Thread.sleep(1000);  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
        System.out.println("test结束..");  
    }  
}  

/*
运行结果：
test开始..
test开始..
test开始..
test结束..
test结束..
test结束..
*///结论是依然没有实现想要的同步效果
```

case 1: 并没有实现想要的同步效果，

Root Cause:	每个线程中都new了一个Sync类的对象，也就是产生了三个Sync对象，由于不是同一个对象，所以可以多线程同时运行synchronized方法或代码段。

改为以下的代码：

```java
//case 3:
class MyThread extends Thread {  
  
    private Sync sync;  
  
    public MyThread(Sync sync) {  
        this.sync = sync;  
    }  
  
    public void run() {  
        sync.test();  
    }  
}  
  
public class Main {  
  
    public static void main(String[] args) {  
        Sync sync = new Sync();  
        for (int i = 0; i < 3; i++) {  
            Thread thread = new MyThread(sync);  
            thread.start();  
        }  
    }  
} 
/*
运行结果：
test开始..
test结束..
test开始..
test结束..
test开始..
test结束..
*///结论是实现了想要的同步
```

更常见的实现方式：

```java
//case 4: 通过synchronized(Sync.class)实现了全局锁的效果。
class Sync {  
  
    public void test() {  
        synchronized (Sync.class) {  
            System.out.println("test开始..");  
            try {  
                Thread.sleep(1000);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
            System.out.println("test结束..");  
        }  
    }  
}  
  
class MyThread extends Thread {  
  
    public void run() {  
        Sync sync = new Sync();  
        sync.test();  
    }  
}  
  
public class Main {  
  
    public static void main(String[] args) {  
        for (int i = 0; i < 3; i++) {  
            Thread thread = new MyThread();  
            thread.start();  
        }  
    }  
}  
```

---

# Conclusion

> synchronized(this)以及非static的synchronized方法（至于static synchronized方法请往下看），只能防止多个线程同时执行**同一个对象**的同步代码段。
>
> synchronized锁住的是代码还是对象。答案是：synchronized锁住的是括号里的对象，而不是代码。对于非static的synchronized方法，锁的就是对象本身也就是this。
>
> static synchronized方法可以直接类名加方法名调用，方法中无法使用this，所以它锁的不是this，而是类的Class对象，所以，static synchronized方法也相当于**全局锁**，相当于锁住了代码段(所有类的实例如果想要执行这段代码都要先获取这个锁)。
>
> 对于非static的方法也可以使用synchronized(Sync.class)实现了全局锁的效果。



看另一个帖子的一段描述：http://www.cnblogs.com/wyhailjn/p/4382505.html

> Java语法规定，任何线程执行同步方法(如上例testSyncMethod是同步方法)、同步代码块(如上例testSyncBlock方法中即是同步代码块)之前，必须先获取对应的**监视器**。对于上例而言：
>
> testSyncBlock方法中同步代码块的监视器是this，即调用该方法的引用对象；
>
> testSyncMethod方法因为加了static关键字，故它的监视器不是this，而是该类本身。

### sychronized(this) 和 sychronized 方法的区别

```java
sychronized(this)是对对象自身的同步，就是你在访问自己的成员是需要进行同步访问。
实际上
public synchronized void Push(char c){
  ...
}
相当于
public void Push(char c){
  synchronized(this){
    ...
  }
}
```



