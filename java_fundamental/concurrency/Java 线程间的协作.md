# Java 线程间的协作

[TOC]

# wait

wait() 和 nofify() 是java中Object对象中自带的方法;

在某种情况下,一个线程处理完自己的某些任务,但是继续往下执行则需要等待某些条件的发生,而这个线程也不能空循环浪费CPU,此时这个线程就可以调用wait()方法将自己挂起后,然后等待他所需要的条件的发生,也可以说等待其他线程调用notify()方法;

wait()方法有一个需要注意的地方就是wait()将会释放线程所获得的锁,而sleep()和yield()这两个方法并没有释放自己所获得的锁(理解这一点很重要), 由于wait()会释放自己所获得的锁,这就意味着其他的线程可以获得这个锁,因此在该对象上中的其他synchronized方法可以在wait()期间被调用.

wait()是java Object 对象自带的方法, 不属于Thread类;

wait() 的两种版本:

```
wait(); //不接受任何参数,这种wait()将会是线程无限期的等待下去, 直到收到notify() 或者 notifyAll();
wait(long timeout);	//设定wait的时间段,如果在此期间收到了nitify() 或者 notifyAll()或者是等待的时间到了,都将退出wait()的状态;
wait(long timeout, int nanos);
```

# notify()



# Condition

