---
title: JAVA 8 新特性
date: 2021/2/2
description: JAVA 8 新特性
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/MYxEQ51GWCyp8R9.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/MYxEQ51GWCyp8R9.jpg'
categories:
  - Java
tags:
  - Java
abbrlink: 62781
---

# JAVA 8 新特性

## 一、Lambda 表达式

> Lambda 表达式，也可称为闭包，它是推动 Java 8 发布的最重要新特性。

Lambda 允许把函数作为一个方法的参数（函数作为参数传递进方法中）。使用 Lambda 表达式可以使代码变的更加简洁紧凑。

**lambda表达式的重要特征:**

- **可选类型声明：**不需要声明参数类型，编译器可以统一识别参数值。
- **可选的参数圆括号：**一个参数无需定义圆括号，但多个参数需要定义圆括号。
- **可选的大括号：**如果主体包含了一个语句，就不需要使用大括号。
- **可选的返回关键字：**如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定明表达式返回了一个数值。

~~~java
// 1. 不需要参数,返回值为 5  
() -> 5  
  
// 2. 接收一个参数(数字类型),返回其2倍的值  
x -> 2 * x  
  
// 3. 接受2个参数(数字),并返回他们的差值  
(x, y) -> x – y  
  
// 4. 接收2个int型整数,返回他们的和  
(int x, int y) -> x + y  
  
// 5. 接受一个 string 对象,并在控制台打印,不返回任何值(看起来像是返回void)  
(String s) -> System.out.print(s)
~~~

比如在多线程编程中，我们使用Lambda表达式来开启新的线程

~~~java
// 1.使用匿名内部类  
new Thread(new Runnable() {  
    @Override  
    public void run() {  
        System.out.println("Hello world !");  
    }  
}).start();  
  
// 2.使用 lambda 表达式
new Thread(() -> System.out.println("Hello world !")).start();  
~~~

1. Lambda表达式可以理解为一种匿名函数：它没有名称，但有参数列表、函数主体、返回类型，可能还有一个可以抛出的异常的列表
2. Lambda表达式让你可以简洁地传递代码
3. 函数式接口就是仅仅声明了一个抽象方法的接口
4. 只有在接受函数式接口的地方才可以使用Lambda表达式
5. Lambda表达式允许你直接内联，为函数式接口的抽象方法提供实现，并且将整个表达式作为函数式接口的一个实例

## 二、函数式接口

- 只包含一个抽象方法的接口，称为**函数式接口**。
- 你可以通过Lambda表达式来创建该接口的对象。（若Lambda表达式抛出一个受检异常，那么该异常需要在目标接口的抽象方法上进行声明）。
- 我们可以在任意函数式接口上使用`@FunctionalInterface`注解，这样做可以检查它是否是一个函数式接口，同时javadoc也会包含一条声明，说明这个接口是一个函数式接口。

函数式接口(Functional Interface)就是一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口。

函数式接口可以被隐式转换为 lambda 表达式。

Lambda 表达式和方法引用（实际上也可认为是Lambda表达式）上。

如定义了一个函数式接口如下：

```java
@FunctionalInterface
interface GreetingService 
{
    void sayMessage(String message);
}
```

那么就可以使用Lambda表达式来表示该接口的一个实现(注：JAVA 8 之前一般是用匿名类实现的)：

```java
GreetingService greetService1 = message -> System.out.println("Hello " + message);
```

**函数式接口可以对现有的函数友好地支持 lambda。**

目前的函数式接口：

- java.lang.Runnable
- java.util.concurrent.Callable
- java.security.PrivilegedAction
- java.util.Comparator
- java.io.FileFilter
- java.nio.file.PathMatcher
- java.lang.reflect.InvocationHandler
- java.beans.PropertyChangeListener
- java.awt.event.ActionListener
- javax.swing.event.ChangeListener
- java.util.function

### 四大函数式接口

#### 1. Consumer\<T> : 消费型接口，void accept(T t);

<img src="https://fabian.oss-cn-hangzhou.aliyuncs.com/img/pTNkc6lAUCiIVwj.png" alt="image-20210209152805992" style="zoom: 50%;" />

```java
Consumer<String> consumer = s -> {
    s = s.toUpperCase();
    System.out.println(s);
};
consumer.accept("test");
// TEST
```

#### 2. Supplier\<T> : 供给型接口，T get();

<img src="https://fabian.oss-cn-hangzhou.aliyuncs.com/img/nwliEdcvCqItV2J.png" alt="image-20210209153222393" style="zoom: 50%;" />

```java
Supplier<String> supplier = () -> {
    String s = "test";
    return s.toUpperCase();
};
System.out.println(supplier.get());
```

#### 3. Function<T, R> : 函数型接口，R apply(T t);

<img src="https://fabian.oss-cn-hangzhou.aliyuncs.com/img/fouHtDVOSB19k2E.png" alt="image-20210209153650946" style="zoom: 50%;" />

```java
Function<Integer, Integer> function = s -> {
    int result = s + 100;
    return ++result;
};
System.out.println(function.apply(3));
```

#### 4. Predicate\<T> : 断言型接口，boolean test(T t);

<img src="https://fabian.oss-cn-hangzhou.aliyuncs.com/img/8yaiMRhTtqkAQx9.png" alt="image-20210209154226464" style="zoom:50%;" />

```java
Predicate<Integer> predicate = s -> {
    s = s * 3;
    return s > 43;
};
System.out.println(predicate.test(8));
```

## 三、Stream 流

> Java 8 API添加了一个新的抽象称为流Stream，可以让你以一种声明的方式处理数据。
>
> Stream 使用一种类似用 SQL 语句从数据库查询数据的直观方式来提供一种对 Java 集合运算和表达的高阶抽象。
>
> Stream API可以极大提高Java程序员的生产力，让程序员写出高效率、干净、简洁的代码。
>
> 这种风格将要处理的元素集合看作一种流， 流在管道中传输， 并且可以在管道的节点上进行处理， 比如筛选， 排序，聚合等。
>
> 元素流在管道中经过中间操作（intermediate operation）的处理，最后由最终操作(terminal operation)得到前面处理的结果。

Stream（流）是一个来自数据源的元素队列并支持聚合操作

- 元素是特定类型的对象，形成一个队列。 Java中的Stream并不会存储元素，而是按需计算。
- **数据源** 流的来源。 可以是集合，数组，I/O channel， 产生器generator 等。
- **聚合操作** 类似SQL语句一样的操作， 比如filter, map, reduce, find, match, sorted等。

和以前的Collection操作不同， Stream操作还有两个基础的特征：

- **Pipelining**: 中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同流式风格（fluent style）。 这样做可以对操作进行优化， 比如延迟执行(laziness)和短路( short-circuiting)。
- **内部迭代**： 以前对集合遍历都是通过Iterator或者For-Each的方式, 显式的在集合外部进行迭代， 这叫做外部迭代。 Stream提供了内部迭代的方式， 通过访问者模式(Visitor)实现。

### 生成流

在 Java 8 中, 集合接口有两个方法来生成流：

- **stream()** − 为集合创建串行流。
- **parallelStream()** − 为集合创建并行流。

~~~java
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());
~~~

### forEach

Stream 提供了新的方法 'forEach' 来迭代流中的每个数据。以下代码片段使用 forEach 输出了10个随机数：

~~~java
Random random = new Random();
random.ints().limit(10).forEach(System.out::println);
~~~

### map

map 方法用于映射每个元素到对应的结果，以下代码片段使用 map 输出了元素对应的平方数：

~~~java
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
// 获取对应的平方数
List<Integer> squaresList = numbers.stream().map( i -> i*i).distinct().collect(Collectors.toList());
~~~

### filter

filter 方法用于通过设置的条件过滤出元素。以下代码片段使用 filter 方法过滤出空字符串：

~~~java
List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
// 获取空字符串的数量
long count = strings.stream().filter(string -> string.isEmpty()).count();
~~~

### limit

limit 方法用于获取指定数量的流。 以下代码片段使用 limit 方法打印出 10 条数据：

~~~java
Random random = new Random();
random.ints().limit(10).forEach(System.out::println);
~~~

### sorted

sorted 方法用于对流进行排序。以下代码片段使用 sorted 方法对输出的 10 个随机数进行排序：

~~~java
Random random = new Random();
random.ints().limit(10).sorted().forEach(System.out::println);
~~~

### 并行（parallel）程序

parallelStream 是流并行处理程序的代替方法。以下实例我们使用 parallelStream 来输出空字符串的数量：

~~~java
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
// 获取空字符串的数量
long count = strings.parallelStream().filter(string -> string.isEmpty()).count();
~~~

### Collectors

Collectors 类实现了很多归约操作，例如将流转换成集合和聚合元素。Collectors 可用于返回列表或字符串：

~~~java
List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());
 
System.out.println("筛选列表: " + filtered);
String mergedString = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.joining(", "));
System.out.println("合并字符串: " + mergedString);
~~~

### 统计

另外，一些产生统计结果的收集器也非常有用。它们主要用于int、double、long等基本类型上，它们可以用来产生类似如下的统计结果。

~~~java
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
 
IntSummaryStatistics stats = numbers.stream().mapToInt((x) -> x).summaryStatistics();
 
System.out.println("列表中最大的数 : " + stats.getMax());
System.out.println("列表中最小的数 : " + stats.getMin());
System.out.println("所有数之和 : " + stats.getSum());
System.out.println("平均数 : " + stats.getAverage());
~~~

