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

方法本身是 native 方法，调用了底层的的类库，根据注释：

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

~~~java
static final int hash(Object key) {
    int h;
    // 这里当 key 不为 null 的时候，将key的哈希值 和 右移16位的哈希值 做异或运算
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
~~~

![image-20210924132832294](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/image-20210924132832294.png)

将h无符号右移16为相当于将高区16位移动到了低区的16位，再与原hashcode做异或运算，可以**将高低位二进制特征混合起来**。

从上文可知高区的16位与原hashcode相比没有发生变化，低区的16位发生了变化。

我们都知道重新计算出的新哈希值在后面将会参与hashmap中数组槽位的计算，计算公式：(n - 1) & hash，假如这时数组槽位有16个，则槽位计算如下：

![image-20210924133141544](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/image-20210924133141544.png)

**高区的16位很有可能会被数组槽位数的二进制码锁屏蔽，如果我们不做刚才移位异或运算，那么在计算槽位时将丢失高区特征**

也许你可能会说，即使丢失了高区特征不同hashcode也可以计算出不同的槽位来，但是细想当两个哈希码很接近时，那么这高区的一点点差异就**可能导致一次哈希碰撞**，所以这也是将性能做到极致的一种体现

## 二、存储方式和数据结构

> 问：HashMap的底层数据结构是什么
>
> 答：数组+链表+红黑树

![image-20210924142231035](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/image-20210924142231035.png)

从上面的图中我们可以看出hashmap的存储结构，默认是数组+链表的结构。

当**链表的长度大于 8 (默认)** 的时候，链表会转换为红黑树。

- node 的数组

  ~~~java
  /**
  * The table, initialized on first use, and resized as
  * necessary. When allocated, length is always a power of two.
  * (We also tolerate length zero in some operations to allow
  * bootstrapping mechanics that are currently not needed.)
  */
  transient Node<K,V>[] table;
  ~~~

- node 结点

  ~~~java
  /**
  * Basic hash bin node, used for most entries.  (See below for
  * TreeNode subclass, and in LinkedHashMap for its Entry subclass.)
  */
  static class Node<K,V> implements Map.Entry<K,V> {
      final int hash;
      final K key;
      V value;
      Node<K,V> next;
  
      Node(int hash, K key, V value, Node<K,V> next) {
          this.hash = hash;
          this.key = key;
          this.value = value;
          this.next = next;
      }
  
      public final K getKey()        { return key; }
      public final V getValue()      { return value; }
      public final String toString() { return key + "=" + value; }
  
      /**
      * node结点的哈希值：将 key 的哈希值和 value 的哈希值进行异或运算
      */
      public final int hashCode() {
          return Objects.hashCode(key) ^ Objects.hashCode(value);
      }
  
      public final V setValue(V newValue) {
          V oldValue = value;
          value = newValue;
          return oldValue;
      }
  
      public final boolean equals(Object o) {
          if (o == this)
              return true;
          if (o instanceof Map.Entry) {
              Map.Entry<?,?> e = (Map.Entry<?,?>)o;
              if (Objects.equals(key, e.getKey()) &&
                  Objects.equals(value, e.getValue()))
                  return true;
          }
          return false;
      }
  }
  ~~~

- treenode 结点

  ~~~java
  /**
  * Entry for Tree bins. Extends LinkedHashMap.Entry (which in turn
  * extends Node) so can be used as extension of either regular or
  * linked node.
  */
  static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
      TreeNode<K,V> parent;  // red-black tree links
      TreeNode<K,V> left;
      TreeNode<K,V> right;
      TreeNode<K,V> prev;    // needed to unlink next upon deletion
      boolean red;
      TreeNode(int hash, K key, V val, Node<K,V> next) {
          super(hash, key, val, next);
      }
  }
  ~~~

  

## 三、构造方法和初始化

```java
/**
 * Constructs an empty <tt>HashMap</tt> with the specified initial
 * capacity and load factor.
 *
 * @param  initialCapacity the initial capacity
 * @param  loadFactor      the load factor
 * @throws IllegalArgumentException if the initial capacity is negative
 *         or the load factor is nonpositive
 */
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
```

这个是最后被调用的构造方法，本质上就是先给 hashmap 初始两个值：**initialCapacity** 和 **loadFactor**：即初始容量和负载因子

### 1. 初始容量 initialCapacity

```java
/**
 * 获取 比 cap 大的最小的二进制数
 * Returns a power of two size for the given target capacity.
 */
static final int tableSizeFor(int cap) {
    // 避免 输入为2的幂时 返回了 cap * 2
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

这里我们以输入 9 为例

1. n = cap - 1;

   **这里如果直接没有 -1 操作，如果输入的为2的幂，补1之后再+1，就不会返回原值而是 cap*2**

   n =  00000000 00000000 00000000 00001000

2. n |= n >>> 1

   00000000 00000000 00000000 00001000  n

   00000000 00000000 00000000 00000100  n >>> 1

   ======================================

   00000000 00000000 00000000 00001100 => n  

3. n |= n >>> 2

   00000000 00000000 00000000 00001101  n

   00000000 00000000 00000000 00000011  n >>> 2

   ======================================

   00000000 00000000 00000000 00001111 => n  

4. 后续操作使得后续全为一

5. 最后检查 大于零且小于最大值，将结果加一返回：16

## 四、添加元素和查找元素

## 五、扩容和rehash

