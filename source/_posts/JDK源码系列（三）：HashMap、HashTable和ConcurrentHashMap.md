---
title: JDK源码系列（三）：HashMap、HashTable和ConcurrentHashMap
date: 2021/4/3
description: JDK源码系列（三）：HashMap、HashTable和ConcurrentHashMap
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/20210921115132.png'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/20210921115132.png'
categories:
  - jdk源码
tags:
  - java
  - 源码
  - jdk
  - HashMap
  - HashTable
  - ConcurrentHashMap
abbrlink: 5986
---

# JDK源码系列（三）：HashMap、HashTable和ConcurrentHashMap

## 一、接口和继承

![image-20211001114726723](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/image-20211001114726723.png)

- `HashMap`：继承于 `AbstractMap`类
- `ConcurrentHashMap`：继承于`AbstractMap`类
- `HashTable`：继承于`Dictionary`类，自己实现了`Map`接口

> ![image-20211001120130544](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/image-20211001120130544.png)
>
> 根据jdk中的注释我们可以看出`Dictionary`已经是一个历史遗留的集合抽象类：
>
> **注意：这个类已经过时了。 新的实现应该实现 Map 接口，而不是扩展这个类。**
>
> Hashtable类与HashMap类的作用一样，实际上，它们拥有相同的接口。与Vector类的方法一样。Hashtable的方法也是同步的。如果对同步性或与遗留代码的兼容性没有任何要求，就应该使用HashMap。如果需要并发访问，则要使用ConcurrentHashMap。

## 二、ConcurrentHashMap 的 `put`方法

```java
public V put(K key, V value) {
    return putVal(key, value, false);
}
```

```java
/** Implementation for put and putIfAbsent */
final V putVal(K key, V value, boolean onlyIfAbsent) {
    // 不允许键或值为 null
    if (key == null || value == null) throw new NullPointerException();
    // 计算哈希值
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        // 先判断有没有初始化，懒加载
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            // 没有哈希冲突就直接 cas 尝试写入
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        // 如果在扩容，该线程就一起帮助辅助扩容
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            // 加锁写入
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    // 链表结点，加在末尾
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    // 红黑树
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            // 判断是否需要链表转二叉树
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    // 增加计数
    addCount(1L, binCount);
    return null;
}
```

![image-20211001160134033](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/3.png)