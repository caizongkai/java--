# Java并发理论知识

> 带着问题学本节知识点

- 多线程的出现是要解决什么问题的?
- 线程不安全是指什么? 举例说明
- 并发出现线程不安全的本质什么? 可见性，原子性和有序性。
- Java是怎么解决并发问题的? 3个关键字，JMM和8个Happens-Before
- 线程安全是不是非真即假? 不是
- 线程安全有哪些实现思路?
- 如何理解并发和并行的区别?

## 为什么需要多线程

因为CPU、内存和I/O设备性能的差异，为了平衡这三者的差异我们需要用到多线程，具体表现如下：

- 因为CPU于I/O设备的性能差异巨大，为了平衡该差异，以分时复用CPU； //会导致**原子性**问题
- 因为CPU和内存性能差异，CPU采用了缓存； //会导致**可见性**问题
- 编译程序优化指令执行顺序，使得缓存能更好的使用； //会导致有序性问题

## Java出现并发问题的三要素：

### 原子性: 分时复用CPU引起

一个操作要么都执行，要么都不执行，典型的如转账问题。比如从账户A向账户B转1000元，那么必然包括2个操作：从账户A减去1000元，往账户B加上1000元。

### 可见性：CPU缓存引起

一个线程对一个变量的修改，另一个线程能立马看到该变量的修改，否则就会出现可见性问题。

```java
//线程1执行的代码
int i = 0;
i = 10;
 
//线程2执行的代码
j = i;

```

### 有序性: 重排序引起

有序性：即程序执行的顺序按照代码的先后顺序执行。

```java
int i = 0;              
boolean flag = false;
i = 1;                //语句1  
flag = true;          //语句2

```

上面代码定义了一个int型变量，定义了一个boolean类型变量，然后分别对两个变量进行赋值操作。从代码顺序上看，语句1是在语句2前面的，那么JVM在真正执行这段代码的时候会保证语句1一定会在语句2前面执行吗? 不一定，为什么呢? 这里可能会发生指令重排序。

重排序包含三种：

- 编译器重排序
- 指令级重排序
- 内存系统重排序

这些重排序都可能会导致多线程程序出现内存可见性问题。对于编译器，JMM 的编译器重排序规则会禁止特定类型的编译器重排序（不是所有的编译器重排序都要禁止）。对于处理器重排序，JMM 的处理器重排序规则会要求 java 编译器在生成指令序列时，插入特定类型的内存屏障（memory barriers，intel 称之为 memory fence）指令，通过内存屏障指令来禁止特定类型的处理器重排序。

## java是如何解决并发问题的（JMM Java内存模型）

java内存模型规范了JVM如何按需提供禁用缓存和编译优化的方法，具体的方法包括：

- volatile、synchronized、final关键字
- Happens-Before规则

### **可见性，有序性，原子性**

- 原子性问题

  ```java
  x = 10;        //语句1: 直接将数值10赋值给x，也就是说线程执行这个语句的会直接将数值10写入到工作内存中
  y = x;         //语句2: 包含2个操作，它先要去读取x的值，再将x的值写入工作内存，虽然读取x的值以及 将x的值写入工作内存 这2个操作都是原子性操作，但是合起来就不是原子性操作了。
  x++;           //语句3： x++包括3个操作：读取x的值，进行加1操作，写入新的值。
  x = x + 1;     //语句4： 同语句3
  
  ```

  JAVA内存模型只保证简单的赋值操作如语句一是原子性操作，如需要更大范围的原子性操作，需要用到synchronized和Lock，由于synchronized和Lock能够保证任一时刻只有一个线程执行该代码块，那么自然就不存在原子性问题了，从而保证了原子性。

- 可见性

  可见性问题可以通过volatile来保证，volatile是通过保证该变量的每次修改都会写入到主存中，每次读取都从主存中读取，从而保证了其可见性。

  其他普通变量何时写入主存是不确定的，所以无法保证其可见性。sychronized和Lock也能保证可见性。

- 有序性

  在Java里面，可以通过volatile关键字来保证一定的“有序性”。另外可以通过synchronized和Lock来保证有序性，很显然，synchronized和Lock保证每个时刻是有一个线程执行同步代码，相当于是让线程顺序执行同步代码，自然就保证了有序性。当然JMM是通过Happens-Before 规则来保证有序性的。



### 三个关键字:sychronized、volatile、final

- sychronized关键字 	//todo	
- volatile关键字    //todo
- final关键字   //todo

## Happens-Before 规则

除了可以用sychronized和volatile来保证有序性，JVM还规定了先行发生规则，让一个操作无需控制就能先于另一操作完成。

### 1、单一线程原则

在一个线程内，程序前面的操作先于程序后面的操作完成。

### 2、管程锁定规则

一个 unlock 操作先行发生于后面对同一个锁的 lock 操作。

### 3、volatile 变量规则

对一个 volatile 变量的写操作先行发生于后面对这个变量的读操作。

### 4、线程启动规则

Thread 对象的 start() 方法调用先行发生于此线程的每一个动作。

### 5、线程加入规则

Thread 对象的结束先行发生于 join() 方法返回。

### 6、线程中断规则

对线程 interrupt() 方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过 interrupted() 方法检测到是否有中断发生。

###  7、对象终结规则

一个对象的初始化完成(构造函数执行结束)先行发生于它的 finalize() 方法的开始。

### 8、传递性

如果操作 A 先行发生于操作 B，操作 B 先行发生于操作 C，那么操作 A 先行发生于操作 C。

## 线程安全：不是一个非真即假的命题

一个类可以被多个线程调用时就是线程安全的，线程安全按照数据共享的强弱分为五个等级：不可变、绝对线程安全、相对线程安全、线程兼容和线程对立。

### 1、不可变

不可变的对象一定是线程安全的，只要一个不可变的对象被正确地构建出来，永远也不会看到它在多个线程之中处于不一致的状态。

不可变的类型：

- String

- final关键字修饰的基本数据类型

- 枚举类型

- Number部分子类，如Long和Double等数值包装类，BigInteger 和 BigDecimal 等大数据类型。但同为 Number 的原子类 

  AtomicInteger 和 AtomicLong 则是可变的。

对于集合类型，可以使用 Collections.unmodifiableXXX() 方法来获取一个不可变的集合。

```java
public class ImmutableExample {
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();
        Map<String, Integer> unmodifiableMap = Collections.unmodifiableMap(map);
        unmodifiableMap.put("a", 1);
    }
}
```

```html
Exception in thread "main" java.lang.UnsupportedOperationException
    at java.util.Collections$UnmodifiableMap.put(Collections.java:1457)
    at ImmutableExample.main(ImmutableExample.java:9)
```

### 2、绝对线程安全

不管运行时环境如何，调用者都不需要任何额外的同步措施。

###  3、相对线程安全

相对线程安全需要保证对这个对象单独的操作是线程安全的，在调用的时候不需要做额外的保障措施。但是对于一些特定顺序的连续调用，就可能需要在调用端使用额外的同步手段来保证调用的正确性。

在 Java 语言中，大部分的线程安全类都属于这种类型，例如 Vector、HashTable、Collections 的 synchronizedCollection() 方法包装的集合等。

对于下面的代码，如果删除元素的线程删除了 Vector 的一个元素，而获取元素的线程试图访问一个已经被删除的元素，那么就会抛出 ArrayIndexOutOfBoundsException。

```java
public class VectorUnsafeExample {
    private static Vector<Integer> vector = new Vector<>();

    public static void main(String[] args) {
        while (true) {
            for (int i = 0; i < 100; i++) {
                vector.add(i);
            }
            ExecutorService executorService = Executors.newCachedThreadPool();
            executorService.execute(() -> {
                for (int i = 0; i < vector.size(); i++) {
                    vector.remove(i);
                }
            });
            executorService.execute(() -> {
                for (int i = 0; i < vector.size(); i++) {
                    vector.get(i);
                }
            });
            executorService.shutdown();
        }
    }
}
```

```html
Exception in thread "Thread-159738" java.lang.ArrayIndexOutOfBoundsException: Array index out of range: 3
    at java.util.Vector.remove(Vector.java:831)
    at VectorUnsafeExample.lambda$main$0(VectorUnsafeExample.java:14)
    at VectorUnsafeExample$$Lambda$1/713338599.run(Unknown Source)
    at java.lang.Thread.run(Thread.java:745)
```

如果要保证上面的代码能正确执行下去，就需要对删除元素和获取元素的代码进行同步。例如用synchronized锁即可。

### 4、线程兼容

线程兼容是指对象本身并不是线程安全的，但是可以通过在调用端正确地使用同步手段来保证对象在并发环境中可以安全地使用，我们平常说一个类不是线程安全的，绝大多数时候指的是这一种情况。Java API 中大部分的类都是属于线程兼容的，如与前面的 Vector 和 HashTable 相对应的集合类 ArrayList 和 HashMap 等。

### 5、线程对立

线程对立是指无论调用端是否采取了同步措施，都无法在多线程环境中并发使用的代码。由于 Java 语言天生就具备多线程特性，线程对立这种排斥多线程的代码是很少出现的，而且通常都是有害的，应当尽量避免。

## 线程安全的实现方法

//todo

### 