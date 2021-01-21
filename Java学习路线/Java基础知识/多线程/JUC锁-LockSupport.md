# JUC锁: LockSupport详解

[TOC]

## 带着面试题学知识

- 为什么LockSupport也是核心基础类? AQS框架借助于两个类：Unsafe(提供CAS操作)和LockSupport(提供park/unpark操作)

- 写出分别通过wait/notify和LockSupport的park/unpark实现同步?

- LockSupport.park()会释放锁资源吗? 那么Condition.await()呢?

- Thread.sleep()、Object.wait()、Condition.await()、LockSupport.park()的区别? 重点

- 如果在wait()之前执行了notify()会怎样?

- 如果在park()之前执行了unpark()会怎样?

## LockSupport简介

当调用LockSupport.park时，表示当前线程将会等待，直至获得许可，当调用LockSupport.unpark时，必须把等待获得许可的线程作为参数进行传递，好让此线程继续运行。

### LockSupport常用方法

```java
park()	//直到unpark唤醒，否则阻塞线程
park(Object blocker) //传入一个blocker，既线程的parkBlocker字段
parkNanos(Object blocker, long nanos) //传入一个相对时间
parkUntil(Object blocker, long deadline)  //传入一个绝对时间
unpark(Thread thread)  //传入一个线程
```

#### park()实现

未传入blocker字段，即不设置parkBolcker，无限阻塞，等到unpark唤醒

```java
public static void park() {
    // 获取许可，设置时间为无限长，直到可以获取许可
    UNSAFE.park(false, 0L);
}
```

#### park(Object blocker)实现

```java
public static void park(Object blocker) {
    // 获取当前线程
    Thread t = Thread.currentThread();
    // 设置Blocker
    setBlocker(t, blocker);
    // 获取许可
    UNSAFE.park(false, 0L);
    // 重新可运行后再此设置Blocker
    setBlocker(t, null);
}
```

```java
private static void setBlocker(Thread t, Object arg) {
    // 设置线程t的parkBlocker字段的值为arg
    UNSAFE.putObject(t, parkBlockerOffset, arg);
}  
```



其实现依赖于unsafe的park方法，当传入blocker时，调用两次setBlocker，在阻塞前后分别setBlocker，否则会出现下次未park的时候getBlocker的时候，得到的还是前一个线程的Blocker。

#### parkNanos实现

等待一个相对时间后释放或者unpark时释放。

```java
public static void parkNanos(Object blocker, long nanos) {
    if (nanos > 0) { // 时间大于0
        // 获取当前线程
        Thread t = Thread.currentThread();
        // 设置Blocker
        setBlocker(t, blocker);
        // 获取许可，并设置了时间
        UNSAFE.park(false, nanos);
        // 设置许可
        setBlocker(t, null);
    }
}
```

#### parkUntil实现

```java
public static void parkUntil(Object blocker, long deadline) {
    // 获取当前线程
    Thread t = Thread.currentThread();
    // 设置Blocker
    setBlocker(t, blocker);
    UNSAFE.park(true, deadline);
    // 设置Blocker为null
    setBlocker(t, null);
}
```

#### unpark实现

```java
public static void unpark(Thread thread) {
    if (thread != null) // 线程为不空
        UNSAFE.unpark(thread); // 释放该线程许可
}
```

## LockSupport使用

使用wait/notify实现线程同步，wait的调用要在notify之前，否则会导致无法notify，线程会一直阻塞，而使用LockSupport可以在park之前调用unpark。

### 使用park/unpark实现线程同步

先调用unpark，再调用park时，仍能够正确实现同步，不会造成由wait/notify调用顺序不当所引起的阻塞。因此park/unpark相比wait/notify更加的灵活。

```java
import java.util.concurrent.locks.LockSupport;

class MyThread extends Thread {
    private Object object;

    public MyThread(Object object) {
        this.object = object;
    }

    public void run() {
        System.out.println("before unpark");        
        // 释放许可
        LockSupport.unpark((Thread) object);
        System.out.println("after unpark");
    }
}

public class ParkAndUnparkDemo {
    public static void main(String[] args) {
        MyThread myThread = new MyThread(Thread.currentThread());
        myThread.start();
        try {
            // 主线程睡眠3s
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("before park");
        // 获取许可
        LockSupport.park("ParkAndUnparkDemo");
        System.out.println("after park");
    }
}
```



###  中断响应

```java
import java.util.concurrent.locks.LockSupport;

class MyThread extends Thread {
    private Object object;

    public MyThread(Object object) {
        this.object = object;
    }

    public void run() {
        System.out.println("before interrupt");        
        try {
            // 休眠3s
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }    
        Thread thread = (Thread) object;
        // 中断线程
        thread.interrupt();
        System.out.println("after interrupt");
    }
}

public class InterruptDemo {
    public static void main(String[] args) {
        MyThread myThread = new MyThread(Thread.currentThread());
        myThread.start();
        System.out.println("before park");
        // 获取许可
        LockSupport.park("ParkAndUnparkDemo");
        System.out.println("after park");
    }
}

```

```html
before park
before interrupt
after interrupt
after park
```



在主线程调用park阻塞后，在myThread线程中发出了中断信号，此时主线程会继续运行，也就是说明此时interrupt起到的作用与unpark一样

## 更深入的理解

### Thread.sleep()和Object.wait()的区别

最大的区别就是Thread.sleep()不会释放锁资源，Object.wait()会释放锁资源。

###  Thread.sleep()和Condition.await()的区别

Object.wait()和Condition.await()的原理是基本一致的，不同的是Condition.await()底层是调用LockSupport.park()来实现阻塞当前线程的。

实际上，它在阻塞当前线程之前还干了两件事，一是把当前线程添加到条件队列中，二是“完全”释放锁，也就是让state状态变量变为0，然后才是调用LockSupport.park()阻塞当前线程。

###  Thread.sleep()和LockSupport.park()的区别

- 从功能上来说，Thread.sleep()和LockSupport.park()方法类似，都是阻塞当前线程的执行，且都不会释放当前线程占有的锁资源；

- Thread.sleep()没法从外部唤醒，只能自己醒过来；

- LockSupport.park()方法可以被另一个线程调用LockSupport.unpark()方法唤醒；

- Thread.sleep()方法声明上抛出了InterruptedException中断异常，所以调用者需要捕获这个异常或者再抛出；

- LockSupport.park()方法不需要捕获中断异常；

- Thread.sleep()本身就是一个native方法；

- LockSupport.park()底层是调用的Unsafe的native方法；

### Object.wait()和LockSupport.park()的区别

- Object.wait()方法需要在synchronized块中执行；

- LockSupport.park()可以在任意地方执行；

- Object.wait()方法声明抛出了中断异常，调用者需要捕获或者再抛出；

- LockSupport.park()不需要捕获中断异常；

- Object.wait()不带超时的，需要另一个线程执行notify()来唤醒，但不一定继续执行后续内容；

- LockSupport.park()不带超时的，需要另一个线程执行unpark()来唤醒，一定会继续执行后续内容；

- 如果在wait()之前执行了notify()会怎样? 抛出IllegalMonitorStateException异常；

- 如果在park()之前执行了unpark()会怎样? 线程不会被阻塞，直接跳过park()，继续执行后续内容；

### LockSupport.park()会释放锁资源吗?

不会，它只负责阻塞当前线程，释放锁资源实际上是在Condition的await()方法中实现的。