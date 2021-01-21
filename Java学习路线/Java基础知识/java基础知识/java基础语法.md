# Java基础语法

## 1、Object类

### Object类有哪些方法？

Object()
默认构造方法
clone()
创建并返回此对象的一个副本。
equals(Object obj)
指示某个其他对象是否与此对象“相等”。
hashCode()
返回该对象的哈希码值。
finalize()
当垃圾回收器确定不存在对该对象的更多引用时，由对象的垃圾回收器调用此方法。
getClass()
返回一个对象的运行时类。
notify()
唤醒在此对象监视器上等待的单个线程。
notifyAll()
唤醒在此对象监视器上等待的所有线程。
toString()
返回该对象的字符串表示。
wait()
导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法。
wait(long timeout)
导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者超过指定的时间量。
wait(long timeout, int nanos)
导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者其他某个线程中断当前线程，或者已超过某个实际时间量。

### 为什么重写equals一定要重写hashcode？

首先我们看下equals的javadoc里面的说明：

```
* @return  {@code true} if this object is the same as the obj
*          argument; {@code false} otherwise.
含义：只有当equals前后的两个对象是同一个对象的时候，才返回true，否则就是false。
```

也就是说，我们如果我们不重写hashcode，两个你认为相同的obj，可能出现不同的hashcode，而hashmap和set之类的数据结构都是依赖hashcode来进行的，

如果不重写hashcode就有可能碰到这个情况，当equals为true的时候，hashcode必须完全相同。当hashcode不同的时候，equals必须为false。

如果不覆盖equals，那么默认equals比较的引用的内存地址，如果覆盖了，你就需要自己去保证这些内容的一致。

还有一系列看上去很唬人实际上很好理解的原则在里面：

```java
两个对象相等，hashcode一定相等
两个对象不等，hashcode不一定不等
hashcode相等，两个对象不一定相等
hashcode不等，两个对象一定不等
简而言之，hashcode相等是两个对象相等的必要不充分条件
```



### 深clone和浅clone说下你的认识？

Object里面有个clone方法是浅clone，clone出来的类，新的对象和原来的对象并不是同一个引用，但是如果类中属性存在引用类型，那么新的类的对应属性也会指向这个引用。

也就是，你如果修改了新对象的引用对象的内容，原来的对象也会发生改变。

如果我们需要clone出来一个完全不同的类，可以通过实现cloneable的protected的clone方法，来确定，

可以通过自己拼接一份新的对象返回，不过对于嵌套层次比较深的类，这是个很浩大的工程。当然也有简单的做法，把对象序列化，然后再反序列化回来其实就是一个新的了。

这种clone之后完全不同的两个对象的情况，就是深clone了。

### wait和notify为什么要定义在Object中，wait和notify是怎么使用的？

### wait和sleep的区别？

sleep是Thread类的静态方法，它生效的也仅仅是当前线程，而不能用 某线程.sleep() .

wait是持有该对象锁的线程主动放弃锁，交出monitor。

sleep本身并不释放所和资源，而只是等待一段时间。wait是交出了锁和资源，后面如果再想获取，需要看自己的实现了。

|            | wait                                                         | sleep                                             |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------- |
| 同步       | 只能在同步上下文中调用wait方法，否则或抛出IllegalMonitorStateException异常 | 不需要在同步方法或同步块中调用                    |
| 作用对象   | wait方法定义在Object类中，作用于对象本身                     | sleep方法定义在java.lang.Thread中，作用于当前线程 |
| 释放锁资源 | 是                                                           | 否                                                |
| 唤醒条件   | 其他线程调用对象的notify()或者notifyAll()方法                | 超时或者调用interrupt()方法体                     |
| 方法属性   | wait是实例方法                                               | sleep是静态方法                                   |

### finalize的效果和使用场景？（未理解）

Java 技术允许使用 finalize() 方法在垃圾收集器将对象从内存中清除出去之前做必要的清理工作。finalize方法在Object类中是protected方法，也就是你可以自己去重写这个方法，但是除非很特殊的情况，一般没必要自己去重写。

这个方法是由垃圾收集器在确定这个对象没有被引用时对这个对象调用的。它是在 Object 类中定义的，因此所有的类都继承了它。子类覆盖 finalize() 方法以整理系统资源或者执行其他清理工作。finalize() 方法是在垃圾收集器删除对象之前对这个对象调用的。 

当垃圾回收器(garbage colector)决定回收某对象时，就会运行该对象的finalize()方法。值得C++程序员注意的是，finalize()方法并不能等同与析构函数。Java中是没有析构函数的。C++的析构函数是在对象消亡时运行的。由于C++没有垃圾回收，对象空间手动回收，所以一旦对象用不到时，程序员就应当把它delete()掉。所以析构函数中经常做一些文件保存之类的收尾工作。但是在Java中很不幸，如果内存总是充足的，那么垃圾回收可能永远不会进行，也就是说filalize()可能永远不被执行，显然指望它做收尾工作是靠不住的。

那么finalize()究竟是做什么的呢？它最主要的用途是回收特殊渠道申请的内存。Java程序有垃圾回收器，所以一般情况下内存问题不用程序员操心。但有一种JNI(Java Native Interface)调用non-Java程序（C或C++），finalize()的工作就是回收这部分的内存。

也就是说，finalize并不保证执行以后就会把内存释放掉，而是会到执行后的下一次垃圾回收才有机会被回收掉。

## 2、String类

### String是不可变的有什么好处？

String是不可变类有以下几个优点

- 由于String是不可变类，所以在多线程中使用是安全的，我们不需要做任何其他同步操作。
- String是不可变的，它的值也不能被改变，所以用来存储数据密码很安全。
- 因为java字符串是不可变的，可以在java运行时节省大量java堆空间。因为不同的字符串变量可以引用池中的相同的字符串。如果字符串是可变得话，任何一个变量的值改变，就会反射到其他变量，那字符串池也就没有任何意义了

### String、StringBuffer与StringBuilder之间区别

| String                                                       | StringBuffer                                                 | StringBuilder    |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------- |
| String的值是不可变的，这就导致每次对String的操作都会生成新的String对象，不仅效率低下，而且浪费大量优先的内存空间 | StringBuffer是可变类，和线程安全的字符串操作类，任何对它指向的字符串的操作都不会产生新的对象。每个StringBuffer对象都有一定的缓冲区容量，当字符串大小没有超过容量时，不会分配新的容量，当字符串大小超过容量时，会自动增加容量 | 可变类，速度更快 |
| 不可变                                                       | 可变                                                         | 可变             |
|                                                              | 线程安全                                                     | 线程不安全       |
|                                                              | 多线程操作字符串                                             | 单线程操作字符串 |

## 3、Math

### 面试题:int和Integer和BigInteger的区别是什么?

int:是基本数据类型 

Integer:是包装类 

BigInteger:是不可变的任意精度的整数(对象)

### 关于基本数据类型,包装数据类型和大整数(大浮点数)类型计算方面

1.基本数据类型,之间进行+- * / 

2.包装数据类型:通过拆箱操作然后再计算 

3.BigInteger,BigDecimal:通过方法

## 4、Date

### **获取当前时间有几种方式？**

答：获取当前时间常见的方式有以下三种：

- new Date()
- Calendar.getInstance().getTime()
- LocalDateTime.now()

###  **获取当前时间的时间戳有几种方式？**

答：以下为获取当前时间戳的几种方式：

- System.currentTimeMillis()
- new Date().getTime()
- Calendar.getInstance().getTime().getTime()
- Instant.now().toEpochMilli()
- LocalDateTime.now().toInstant(ZoneOffset.of("+8")).toEpochMilli()

### **SimpleDateFormat 是线程安全的吗？为什么？**

答：SimpleDateFormat 是非线程安全的。因为查看 SimpleDateFormat 的源码可以得知，所有的格式化和解析，都需要通过一个中间对象进行转换，这个中间对象就是 Calendar，这样的话就造成非线程安全。试想一下当我们有多个线程操作同一个 Calendar 的时候后来的线程会覆盖先来线程的数据，那最后其实返回的是后来线程的数据，因此 SimpleDateFormat 就成为了非线程的了。

### 时间格式化

```java
// 时间格式化①
DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String timeFormat = dateTimeFormatter.format(LocalDateTime.now());
System.out.println(timeFormat);  // output:2019-08-16 21:15:43
// 时间格式化②
String timeFormat2 = LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
System.out.println(timeFormat2);    // output:2019-08-16 21:17:48
```

### **时间转换**

```java
String timeStr = "2019-10-10 06:06:06";
LocalDateTime dateTime = LocalDateTime.parse(timeStr,DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
System.out.println(dateTime);
```

###  **怎么保证 SimpleDateFormat 的线程安全？**

答：保证 SimpleDateFormat 线程安全的方式如下：

- 使用 Synchronized，在需要时间格式化的操作使用 Synchronized 关键字进行包装，保证线程堵塞格式化；
- 手动加锁，把需要格式化时间的代码，写到加锁部分，相对 Synchronized 来说，编码效率更低，性能略好，代码风险较大（风险在于不要忘记在操作的最后，手动释放锁）；
- 使用 JDK 8 的 DateTimeFormatter 替代 SimpleDateFormat。



