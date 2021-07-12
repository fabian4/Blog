---
title: JAVA 并发相关
date: 2021/1/29
description: JAVA 并发相关
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/ujHGUnEID1JSgqF.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/ujHGUnEID1JSgqF.jpg'
categories:
  - Java
tags:
  - Java
  - 多线程
abbrlink: 43332
---

# JAVA 并发相关

## 一、乐观锁和悲观锁

- 悲观锁：

  总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁。

  > 传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。再比如Java里面的同步原语synchronized关键字的实现也是悲观锁。

  

- 乐观锁：

  顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。

  > 乐观锁适用于多读的应用类型，这样可以提高吞吐量，像数据库提供的类似于write_condition机制，其实都是提供的乐观锁。在Java中java.util.concurrent.atomic包下面的原子变量类就是使用了乐观锁的一种实现方式CAS实现的。

Java在JDK1.5之前都是靠 synchronized关键字保证同步的，这种通过使用一致的锁定协议来协调对共享状态的访问，可以确保无论哪个线程持有共享变量的锁，都采用独占的方式来访问这些变量。这就是一种独占锁，独占锁其实就是一种悲观锁，所以可以说 **synchronized 是悲观锁。**

悲观锁机制存在以下问题：　　

1. 在多线程竞争下，加锁、释放锁会导致比较多的上下文切换和调度延时，引起性能问题。
2. 一个线程持有锁会导致其它所有需要此锁的线程挂起。
3. 如果一个优先级高的线程等待一个优先级低的线程释放锁会导致优先级倒置，引起性能风险。

**对比于悲观锁的这些问题，另一个更加有效的锁就是乐观锁。其实乐观锁就是：每次不加锁而是假设没有并发冲突而去完成某项操作，如果因为并发冲突失败就重试，直到成功为止。**

## 二、`java.util.concurrent.atomic` 

在我们 JUC 包下面有个 `atomic` 中的一些类可以确保我们的**原子操作**

![image-20210205143737342](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/NzgiAur2FWCc5Rk.png)

我们可以通过以下程序对其进行一个简单的测试

~~~java
public class AtomicDemo {

    private AtomicInteger atomicInteger= new AtomicInteger(0);

    public void add(){
        System.out.println(Thread.currentThread().getName()+"  ============================== "+atomicInteger.incrementAndGet());
    }

    public static void main(String[] args) throws InterruptedException {
        AtomicDemo atomicDemo = new AtomicDemo();
        for (int i = 0; i < 10000; i++) {
            new Thread(atomicDemo::add).start();
        }
    }

}
~~~

<img src="https://fabian.oss-cn-hangzhou.aliyuncs.com/img/Kw5EbiUZJ3LC4n7.png" alt="image-20210205144251148" style="zoom: 50%;" />

## 三、乐观锁的实现：CAS

> 上面提到的乐观锁的概念中其实已经阐述了它的具体实现细节：主要就是两个步骤：**冲突检测和数据更新。**

CAS（Compare and swap）比较和替换是设计并发算法时用到的一种技术。简单来说，比较和替换是使用一个期望值和一个变量的当前值进行比较，如果当前变量的值与我们期望的值相等，就使用一个新值替换当前变量的值。

而 `atomic` 包中也为我们提供了相应的方法

![image-20210205150723972](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/Urd59WG4MBCmJPZ.png)

```java
atomicInteger.compareAndSet(0, 1);
atomicInteger.weakCompareAndSet(0,1);
```

## 四、ABA问题

线程1准备用CAS将变量的值由A替换为C，在此之前，线程2将变量的值由A替换为B，又由B替换为A，然后线程1执行CAS时发现变量的值仍然为A，所以CAS成功。但实际上这时的现场已经和最初不同了，尽管CAS成功，但可能存在潜藏的问题。

**ABA问题的根本在于在修改变量的时候，无法记录变量的状态，比如修改的次数，是否修改过这个变量。这样就很容易在一个线程将A修改成B时，另一个线程又会把B修改成A，造成多次执行的问题。**

**解决方法**：

Java并发包为了解决这个问题，提供了一个带有标记的原子引用类“AtomicStampedReference”，它可以通过控制变量值的版本来保证CAS的正确性。每次在执行数据的修改操作时，都会带上一个版本号，一旦版本号和数据的版本号一致就可以执行修改操作并对版本号执行+1操作，否则就执行失败。

![image-20210205153138400](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/ZJQhsumCXdVqlge.png)

![image-20210205153155828](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/STjrQE8zvo9WLyP.png)

## 五、JNI  和 native 关键字

> JNI是Java Native Interface的缩写，通过使用 Java本地接口书写程序，可以确保代码在不同的平台上方便移植。 从Java1.1开始，JNI标准成为java平台的一部分，它允许Java代码和其他语言写的代码进行交互。JNI一开始是为了本地已编译语言，尤其是C和C++而设计的，但是它并不妨碍你使用其他编程语言，只要调用约定受支持就可以了。使用java与本地已编译的代码交互，通常会丧失平台可移植性。但是，有些情况下这样做是可以接受的，甚至是必须的。例如，使用一些旧的库，与硬件、操作系统进行交互，或者为了提高程序的性能。JNI标准至少要保证本地代码能工作在任何Java 虚拟机环境。

原因是Java很好，使用的人很多、应用极广，但是Java不是完美的。Java的不足体现在运行速度要比传统的C++慢上许多之外，还有Java无法直接访问到操作系统底层如硬件系统。为此Java提供了JNI来实现对于底层的访问。JNI，Java Native Interface，它是Java的SDK一部分，JNI允许Java代码使用以其他语言编写的代码和代码库，本地程序中的函数也可以调用Java层的函数，即JNI实现了Java和本地代码间的双向交互。

<img src="https://fabian.oss-cn-hangzhou.aliyuncs.com/img/xMbgpu6ZLFQw379.png" alt="image-20210205154317174" style="zoom:50%;" />

### JAVA 调用 C 流程

<img src="https://fabian.oss-cn-hangzhou.aliyuncs.com/img/hJ2FqxpzMUaTPWv.png" alt="image-20210205154418616" style="zoom: 50%;" />

### native 关键字

> **java语言是运行在虚拟机上的，java又是不允许直接访问硬件的（也就是java安全性的体现），而java想要做一些例如绘图、画线之类的要去操作硬件的事情的话，必然要用到底层一些的调用。这就引出了native的关键字。**

**native关键字说明其修饰的方法是一个原生态方法，方法对应的实现不是在当前文件，而是在用其他语言（如C和C++）实现的文件中。Java语言本身不能对操作系统底层进行访问和操作，但是可以通过JNI接口调用其他语言来实现对底层的访问。**

如	在 java.lang.Object 源码中的一个hashCode方法：

~~~java
public native int hashCode(); 
~~~