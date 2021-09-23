---
title: JDK源码系列（二）：HashMap源码解析
date: 2021/3/37
description: JDK源码系列（二）：HashMap源码解析
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/20210921115132.png'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/20210921115132.png'
categories:
  - jdk源码
tags:
  - java
  - 源码
  - jdk
  - hashmap
abbrlink: 14375
---

# JDK源码系列（二）：HashMap源码解析

作为面试必问的hashmap，我们今天来一起来探究一下他的源码。这里先贴一张 hashmap 的内部类和部分方法函数

<img src="https://fabian.oss-cn-hangzhou.aliyuncs.com/img/image-20210921204040629.png" alt="image-20210921204040629" style="zoom: 67%;" />

## 一、什么是 哈希， 哈希函数（散列函数）

> Hash，一般翻译做散列、杂凑，或音译为哈希，是把任意长度的[输入](https://baike.baidu.com/item/输入/5481954)（又叫做预映射pre-image）通过散列算法变换成固定长度的[输出](https://baike.baidu.com/item/输出/11056752)，该输出就是散列值。这种转换是一种[压缩映射](https://baike.baidu.com/item/压缩映射/5114126)，也就是，散列值的空间通常远小于输入的空间，不同的输入可能会散列成相同的输出，所以不可能从散列值来确定唯一的输入值。简单的说就是一种将任意长度的消息压缩到某一固定长度的[消息摘要](https://baike.baidu.com/item/消息摘要/4547744)的函数。

上面是百度百科的解释。我们可以简单的概括为

- hash 就是将 **任意的输入** 通过 一种 **特殊的算法** 变成 **长度固定** 的输出。 
- 可以类似于一种摘要算法，获取我们输入内容的特征来生成一个简短的摘要
- 由于算法可以保证：相同的输入每次都是可以得到相同的输出，即hash值
- 但是也会发生：不同的输入哈希之后会得到相同的结果 **即 哈希冲突**

### object 中的 hashcode 方法

~~~java
public native int hashCode();
~~~

方法本身是 native 方法，调用了底层的的类库，但是根据注释：

hashCode 遵守着如下的约定：

- Whenever it is invoked on the same object more than once during an execution of a Java application, the hashCode method must consistently return the same integer, provided no information used in equals comparisons on the object is modified. This integer need not remain consistent from one execution of an application to another execution of the same application.
- If two objects are equal according to the equals(Object) method, then calling the hashCode method on each of the two objects must produce the same integer result.
- It is not required that if two objects are unequal according to the equals(Object) method, then calling the hashCode method on each of the two objects must produce distinct integer results. However, the programmer should be aware that producing distinct integer results for unequal objects may improve the performance of hash tables.

翻译一下就是

- 在程序一次执行的过程中，无论何时被调用，这个方法都会返回相同的值。但是程序的多次执行就不能保证每一次都相等

- 如果两个对象的 equals 方法返回 true，那么他们调用此方法就一定返回相同的结果

- 当然也不一定要确保：两个对象如果根据重写的 equals 方法返回为 false，那么此方法的返回结果一定不同。

  > 即：两个对象 根据重写的equals判断为不相等，此时的 hashcode 也可以相同
  >
  > 不过我们在写程序时应该注意尽量是不同的对象返回不同的hashcode，以便加强哈希效率，避免哈希冲突

### hashmap 采用的哈希方法

贴一下其中 hash 方法的源码

~~~java

~~~





## 二、存储方式和数据结构

> 问：HashMap的底层数据结构是什么
>
> 答：数组+链表+红黑树

首先我们知道 hashmap 是一个 K-V 结构的存储关系，qi

## 三、构造方法和初始化

## 四、扩容和rehash

## 五、添加元素和查找元素

