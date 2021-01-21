

# final关键字

[TOC]



## 带着面试问题学知识

- 所有的final修饰的字段都是编译期常量吗?不一定

- 如何理解private所修饰的方法是隐式的final?private修饰的方法无法被子类重写

- 说说final类型的类如何拓展? 比如String是final类型，我们想写个MyString复用所有String中方法，同时增加一个新的toMyString()的方法，应该如何做?声明一个String的变量，用这个变量去调用String的方法

- final方法可以被重载吗? 可以

- 父类的final方法能不能够被子类重写? 不可以

- 说说final域重排序规则?
- 说说final的原理?

- 使用 final 的限制条件和局限性?

- 看本文最后的一个思考题

## final基础使用

### 修饰类

final修饰的类无法继承

### 修饰方法

- private 方法是隐式的final（final修饰的方法无法重写）
- final方法是可以被重载的

### 修饰变量

- 一个既是static又是final 的字段只占据一段不能改变的存储空间，它必须在定义的时候进行赋值，否则编译器将不予通过。
- 被final修饰的变量在初始化之后就无法修改
- 被final修饰的变量必须在构造方法之前被初始化
- 被final修饰的变量会在构造完成之前完成初始化，禁止处理器对该final域进行重排序，既在读一个对象的final域之前，会确保先读该对象，既执行对象的构造函数，不会出现读取到未初始化的final域的值。

## final再深入理解

###  final的实现原理

通过内存屏障实现。写final域会要求编译器在final域写之后，构造函数返回前插入一个StoreStore屏障。读final域的重排序规则会要求编译器在读final域的操作前插入一个LoadLoad屏障。

### 为什么final引用不能从构造函数中“溢出”

当对象引用在构造函数中溢出，既可能出现其他线程在读该对象的引用时候，出现线程安全问题，可能会出现构造函数中的对象引用未被初始化就被别的线程读取的问题。

```java
public class FinalReferenceEscapeDemo {
    private final int a;
    private FinalReferenceEscapeDemo referenceDemo;

    public FinalReferenceEscapeDemo() {
        a = 1;  //1
        referenceDemo = this; //2
    }

    public void writer() {
        new FinalReferenceEscapeDemo();
    }

    public void reader() {
        if (referenceDemo != null) {  //3
            int temp = referenceDemo.a; //4
        }
    }
}
```

### 再思考一个有趣的现象：

```java
byte b1=1;
byte b2=3;
byte b3=b1+b2;//当程序执行到这一行的时候会出错，因为b1、b2可以自动转换成int类型的变量，运算时java虚拟机对它进行了转换，结果导致把一个int赋值给byte-----出错
```

```java
如果对b1 b2加上final就不会出错
final byte b1=1;
final byte b2=3;
byte b3=b1+b2;//不会出错，因为基本数据类型被声明为final就无法改变其值
```

