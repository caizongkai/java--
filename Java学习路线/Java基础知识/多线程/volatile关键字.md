# Volatile关键字

[TOC]

## 带着面试常见问题学知识

- volatile关键字的作用是什么?

- volatile能保证原子性吗?

- 之前32位机器上共享的long和double变量的为什么要用volatile? 现在64位机器上是否也要设置呢?

- i++为什么不能保证原子性?

- volatile是如何实现可见性的?  内存屏障。

- volatile是如何实现有序性的?  happens-before等

- 说下volatile的应用场景?

## volatile关键字的作用

### 禁止重排序

我们从一个最经典的例子来分析重排序问题。大家应该都很熟悉单例模式的实现，而在并发环境下的单例实现方式，我们通常可以采用**双重检查加锁**(DCL)的方式来实现。其源码如下：

```java
public class Singleton {
    public static volatile Singleton singleton;
    /**
     * 构造函数私有，禁止外部实例化
     */
    private Singleton() {};
    public static Singleton getInstance() {
        if (singleton == null) {
            synchronized (singleton) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

构造一个对象的过程如下：

- 为对象分配空间
- 初始化对象
- 将内存空间的地址赋值给引用的对象

上述的步骤2跟3可能发生重排序，可能会导致对象未实例化就暴露出来，造成严重的后果。volatile关键字可以禁止该重排序。

#### volatile有序性实现的原理

##### volatile 的 happens-before 关系

happens-before 规则中有一条是 volatile 变量规则：对一个 volatile 域的写，happens-before 于任意后续对这个 volatile 域的读。

##### 内存屏障

### 实现可见性

因为工作线程在进行写操作的时候会将变量的值先写入缓存，然后在一定的时间后才写入内存中，这就可能导致在一个线程进行写操作后另外一个线程未嗅探到该值的变化，导致可见性问题。volatile可以让线程在执行写操作的时候立刻将值写入到内存中，防止可见性问题。

#### volatile对可见性的实现原理

volatile的可见性的实现是依赖于内存屏障，内存屏障会禁止编译器和处理器对该代码进行重排序，然后将线程的高速缓存区的内容写入到内存中，并通过**缓存一致性协议（MESI）**使其他线程对于该值的引用发生更新。

##### 缓存一致性协议

既在同一个时间只有一个CPU可以对内存进行读写操作，CPU缓存除了在读写的时候和内存打交道，还会不断嗅探内存的值的变化，当内存的值变化后，CPU缓存会更新自身对该值的引用。

## volatile只能保证单次读写操作的原子性

基于这个原因，我们可以明白两件事：

1. i++为什么不能保证原子性：

   因为i++操作是两次操作，既读i的值，对i的值进行+1操作。

2. 共享long和double是否需要加volatile：

需要，因为long和double分为高32位和低32位，加volatile修饰符可以保证对long和double的单次读写操作具有原子性。

