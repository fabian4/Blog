---
title: JAVA 多线程编程
date: 2021/1/25
description: JAVA 多线程编程
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/DBQoWytONRSr5I3.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/DBQoWytONRSr5I3.jpg'
categories:
  - Java
tags:
  - Java
  - 多线程
abbrlink: 6499
---

# JAVA 多线程编程

## 一、线程和进程

**进程**：是代码在数据集合上的一次运行活动，是系统进行资源分配和调度的基本单位。

**线程**：是进程的一个执行路径，一个进程中至少有一个线程，进程中的多个线程共享进程的 资源。

虽然系统是把资源分给进程，但是CPU很特殊，是被分配到线程的，所以线程是CPU分配的基本单位。

<img src="https://fabian.oss-cn-hangzhou.aliyuncs.com/img/145NoOFLkjP3RVU.png" alt="image-20210202170351505" style="zoom:50%;" />

### 二者联系

一个进程中有多个线程，多个线程共享进程的堆和方法区资源，但是每个线程有自己的程序计数器和栈区域。

 **程序计数器**：是一块内存区域，用来记录线程当前要执行的指令地址 。

**栈**：用于存储该线程的局部变量，这些局部变量是该线程私有的，除此之外还用来存放线程的调用栈祯。

**堆**：是一个进程中最大的一块内存，堆是被进程中的所有线程共享的。

**方法区**：则用来存放 NM 加载的类、常量及静态变量等信息，也是线程共享的 。

### 二者区别

**进程**：有独立的地址空间，一个进程崩溃后，在保护模式下不会对其它进程产生影响。

**线程**：是一个进程中的不同执行路径。线程有自己的堆栈和局部变量，但线程之间没有单独的地址空间，一个线程死掉就等于整个进程死掉。

- 简而言之,一个程序至少有一个进程,一个进程至少有一个线程。
- 线程的划分尺度小于进程，使得多线程程序的并发性高。
- 另外，进程在执行过程中拥有独立的内存单元，而多个线程共享内存，从而极大地提高了程序的运行效率。
- 每个独立的线程有一个程序运行的入口、顺序执行序列和程序的出口。但是线程不能够独立执行，必须依存在应用程序中，由应用程序提供多个线程执行控制。
- 从逻辑角度来看，多线程的意义在于一个应用程序中，有多个执行部分可以同时执行。但操作系统并没有将多个线程看做多个独立的应用，来实现进程的调度和管理以及资源分配。这就是进程和线程的重要区别

## 二、创建线程的四种方式

### 1. 继承 Thread 类

```java
public class ThreadDemo extends java.lang.Thread {

    @Override
    public void run() {
        for (int i = 1; i <= 5; i++) {
            System.out.println(Thread.currentThread().getName() + " =================================> " + i);
        }
    }

    public static void main(String[] args) {
        ThreadDemo threadDemo1 = new ThreadDemo();
        ThreadDemo threadDemo2 = new ThreadDemo();
        ThreadDemo threadDemo3 = new ThreadDemo();
        // 这里是调用 start()方法而不是run()！！！！
        threadDemo1.start();
        threadDemo2.start();
        threadDemo3.start();
    }
}
```

<img src="https://fabian.oss-cn-hangzhou.aliyuncs.com/img/xlXQuYndBgPa3oq.png" alt="image-20210204130244903" style="zoom: 50%;" />

### 2. 实现 runnable 接口

```java
public class RunnableDemo implements Runnable {
    
    @Override
    public void run() {
        for (int i = 1; i <= 5; i++) {
            System.out.println(Thread.currentThread().getName() + " =================================> " + i);
        }
    }

    public static void main(String[] args) {
        RunnableDemo runnableDemo = new RunnableDemo();
        new Thread(runnableDemo, "a").start();
        new Thread(runnableDemo, "b").start();
        new Thread(runnableDemo, "c").start();
    }
}
```

> 这里也可以直接使用 lambda 表达式

```java
public static void main(String[] args) {
    new Thread(()->{
        for (int i = 1; i <= 50; i++) {
            System.out.println(Thread.currentThread().getName() + " ====> " + i);
        }
    }, "a").start();
    new Thread(()->{
        for (int i = 1; i <= 50; i++) {
            System.out.println(Thread.currentThread().getName() + " ===========> " + i);
        }
    }, "b").start();
    new Thread(()->{
        for (int i = 1; i <= 50; i++) {
            System.out.println(Thread.currentThread().getName() + " ======================> " + i);
        }
    }, "c").start();
}
```

### 3. 实现 callable 接口

```java
import java.util.concurrent.Callable;
import java.util.concurrent.FutureTask;

public class CallableDemo implements Callable<String> {
    @Override
    public String call(){
        for (int i = 1; i <= 50; i++) {
            System.out.println(Thread.currentThread().getName()+" "+i);
        }
        return "";
    }

    public static void main(String[] args) {
        CallableDemo callableDemo = new CallableDemo();
        FutureTask<String> task = new FutureTask<>(callableDemo);
        new Thread(task, "a").start();
        new Thread(task, "b").start();
        new Thread(task, "c").start();
    }
}
```

### 4. 线程池

```java
public class ThreadPoolDemo {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < 10; i++) {
            executorService.execute(()->{
                for (int j = 0; j < 10; j++) {
                    System.out.println(Thread.currentThread().getName());
                }
            });
        }
        executorService.shutdown();
    }
}
```

## 三、线程池的创建

### 1. 线程池七大参数

```java
public ThreadPoolExecutor(int corePoolSize, // 核心线程池大小
                          int maximumPoolSize, // 核心线程池大小
                          long keepAliveTime, // 存活时间
                          TimeUnit unit, // 时间单位
                          BlockingQueue<Runnable> workQueue, // 阻塞队列
                          ThreadFactory threadFactory, // 线程工厂
                          RejectedExecutionHandler handler // 拒绝策略) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

根据阿里巴巴的Java开发手册

![image-20210204143107656](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/jYWKO69hpPEF8tS.png)

~~~java
1. Executors.newSingleThreadExecutor(); 
// 创建单个的线程池
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }

 public LinkedBlockingQueue() {
        this(Integer.MAX_VALUE);
    }


2. Executors.newFixedThreadPool(3);
// 创建固定线程量的线程池
 public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
    }

 public LinkedBlockingQueue() {
        this(Integer.MAX_VALUE);
    }


3. Executors.newCachedThreadPool();
// 创建可伸缩的线程池
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
    }
~~~

**以上三种创建方法都默认参数填入了最大值，可能会堆积大量的请求，创建大量的线程，从而导致 OOM。**

我们应该指定参数进行创建

```java
ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
    3,
    5,
    1,
    TimeUnit.MINUTES,
    new LinkedBlockingDeque<>(3),
    Executors.defaultThreadFactory(),
    new ThreadPoolExecutor.AbortPolicy()
);
```

### 2. 四种拒绝策略

~~~java
ThreadPoolExecutor.AbortPolicy(); 
// 丢弃任务并抛出RejectedExecutionException异常。

ThreadPoolExecutor.CallerRunsPolicy(); 
// 丢弃任务，但是不抛出异常。如果线程队列已满，则后续提交的任务都会被丢弃，且是静默丢弃。

ThreadPoolExecutor.DiscardOldestPolicy();
// 丢弃队列最前面的任务，然后重新提交被拒绝的任务。

ThreadPoolExecutor.DiscardPolicy();
// 由调用线程处理该任务
~~~

