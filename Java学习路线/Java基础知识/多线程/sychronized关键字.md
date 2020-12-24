# sychronized关键字详解

[TOC]



> 带着问题学知识

- Synchronized可以作用在哪里? 分别通过对象锁和类锁进行举例。

- Synchronized本质上是通过什么保证线程安全的? 分三个方面回答：加锁和释放锁的原理，可重入原理，保证可见性原理。

- Synchronized有什么样的缺陷?  Java Lock是怎么弥补这些缺陷的。
- Synchronized和Lock的对比，和选择?
- Synchronized在使用时有何注意事项?
- Synchronized修饰的方法在抛出异常时,会释放锁吗?
- 多个线程等待同一个snchronized锁的时候，JVM如何选择下一个获取锁的线程?
- Synchronized使得同时只有一个线程可以执行，性能比较差，有什么提升的方法?
- 我想更加灵活地控制锁的释放和获取(现在释放锁和获取锁的时机都被规定死了)，怎么办?
- 什么是锁的升级和降级? 什么是JVM里的偏斜锁、轻量级锁、重量级锁?
- 不同的JDK中对Synchronized有何优化?



## Synchronized锁的使用

synchronized锁可以分为对象锁，类锁。一个synchronized锁只能由一个线程获得，其他没有获得的线程只能等待；

### 对象锁

使用对象锁有两种方式

1. 在普通方法上加锁
2. 在代码块上加锁，可以给当前对象this加锁，也可以自定义锁

以下是代码实现：

在普通方法上加锁：

```java
public synchronized void method() {
        System.out.println("我是线程" + Thread.currentThread().getName());
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "结束");
 }
```

在代码块上加锁，锁对象是this:

```java
public class SynchronizedObjectLock implements Runnable {
    static SynchronizedObjectLock instence = new SynchronizedObjectLock();

    @Override
    public void run() {
        // 同步代码块形式——锁为this,两个线程使用的锁是一样的,线程1必须要等到线程0释放了该锁后，才能执行
        synchronized (this) {
            System.out.println("我是线程" + Thread.currentThread().getName());
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "结束");
        }
    }

    public static void main(String[] args) {
        Thread t1 = new Thread(instence);
        Thread t2 = new Thread(instence);
        t1.start();
        t2.start();
    }
}
```

在代码块上加锁，并且自定义锁对象：

```java
public class SynchronizedObjectLock implements Runnable {
    static SynchronizedObjectLock instence = new SynchronizedObjectLock();
    // 创建2把锁
    Object block1 = new Object();
    Object block2 = new Object();

    @Override
    public void run() {
        // 这个代码块使用的是第一把锁，当他释放后，后面的代码块由于使用的是第二把锁，因此可以马上执行
        synchronized (block1) {
            System.out.println("block1锁,我是线程" + Thread.currentThread().getName());
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("block1锁,"+Thread.currentThread().getName() + "结束");
        }

        synchronized (block2) {
            System.out.println("block2锁,我是线程" + Thread.currentThread().getName());
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("block2锁,"+Thread.currentThread().getName() + "结束");
        }
    }

    public static void main(String[] args) {
        Thread t1 = new Thread(instence);
        Thread t2 = new Thread(instence);
        t1.start();
        t2.start();
    }
}

```

### 类锁

类锁的加锁形式有两种：

1. 在静态方法上加关键字synchronized
2. 在方法块上加锁，并且锁对象是当前Class

以下是代码实现：

在静态方法上加关键字synchronized：

```java
   // synchronized用在静态方法上，默认的锁就是当前所在的Class类，所以无论是哪个线程访问它，需要的锁都只有一把
    public static synchronized void method() {
        System.out.println("我是线程" + Thread.currentThread().getName());
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "结束");
    }
```

在方法块上加锁，并且锁对象是当前Class:

```java

 synchronized(SynchronizedObjectLock.class){
     System.out.println("我是线程" + Thread.currentThread().getName());
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "结束");
 }
```

## Synchronized加锁、释放锁原理

线程获取锁时会去获取monitor，当monitor计数器为0的时候表示没有线程进入该锁，否则表示已经有线程在该锁里面，此时线程进入等待状态。

当线程成功获取monitor，且monitor为0时，线程会执行monitorenter指令进入锁，monitor计数器会+1，如果线程拿到了monitor的所有权，并且重入了这把锁，那么monitor的计数器的值会+1，(注意:此时线程不需要执行monitorenter指令)，如果一直重入会一直累加。当锁定的代码执行完毕之后，monitor会-1，如果-1之后monitor的值变为0，那么线程会执行monitorexit指令，释放锁。

任意线程对Object的访问，首先要获得Object的监视器，如果获取失败，该线程就进入同步状态，线程状态变为BLOCKED，当Object的监视器占有者释放后，在同步队列中得线程就会有机会重新获取该监视器。

## JVM的锁优化

//todo