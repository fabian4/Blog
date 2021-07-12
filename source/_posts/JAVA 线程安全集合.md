---
title: JAVA 线程安全集合
date: 2021/1/31
description: JAVA 线程安全集合
top_img: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/7xPYzltZs94ROI6.jpg'
cover: 'https://fabian.oss-cn-hangzhou.aliyuncs.com/img/7xPYzltZs94ROI6.jpg'
categories:
  - Java
tags:
  - Java
  - 多线程
abbrlink: 12941
---

# JAVA 线程安全集合

## 一、线程安全

- 线程安全

  就是当多线程访问时，采用了加锁的机制；即当一个线程访问该类的某个数据时，会对这个数据进行保护，其他线程不能对其访问，直到该线程读取完之后，其他线程才可以使用。防止出现数据不一致或者数据被污染的情况。

- 线程不安全

  就是不提供数据访问时的数据保护，多个线程能够同时操作某个数据，从而出现数据不一致或者数据污染的情况。

- 线程安全 工作原理： 

  jvm中有一个main memory对象，每一个线程也有自己的working memory，一个线程对于一个变量variable进行操作的时候， 都需要在自己的working memory里创建一个copy,操作完之后再写入main memory。 
  当多个线程操作同一个变量variable，就可能出现不可预知的结果。 
  而用synchronized的关键是建立一个监控monitor，这个monitor可以是要修改的变量，也可以是其他自己认为合适的对象(方法)，然后通过给这个monitor加锁来实现线程安全，每个线程在获得这个锁之后，要执行完加载load到working memory 到 use && 指派assign 到 存储store 再到 main memory的过程。才会释放它得到的锁。这样就实现了所谓的线程安全。

## 二、相关对象集合比较

### `Vector、ArrayList、LinkedList`

1. **Vector** 	线程安全
   Vector与ArrayList一样，也是通过数组实现的，不同的是它支持线程的同步，即某一时刻只有一个线程能够写Vector，避免多线程同时写而引起的不一致性，**但实现同步需要很高的花费，因此，访问它比访问ArrayList慢**。 
2. **ArrayList** 
   1. 当操作是在一列数据的后面添加数据而不是在前面或者中间，并需要随机地访问其中的元素时，使用ArrayList性能比较好
   2. ArrayList是最常用的List实现类，内部是通过数组实现的，它允许对元素进行快速随机访问。数组的缺点是每个元素之间不能有间隔，当数组大小不满足时需要增加存储能力，就要讲已经有数组的数据复制到新的存储空间中。当从ArrayList的中间位置插入或者删除元素时，需要对数组进行复制、移动、代价比较高。因此，它适合随机查找和遍历，不适合插入和删除。 
3. **LinkedList** 
   1. 当对一列数据的前面或者中间执行添加或者删除操作时，并且按照顺序访问其中的元素时，要使用LinkedList。 
   2. LinkedList是用链表结构存储数据的，很适合数据的动态插入和删除，随机访问和遍历速度比较慢。另外，他还提供了List接口中没有定义的方法，专门用于操作表头和表尾元素，可以当作堆栈、队列和双向队列使用。

> Vector和ArrayList在使用上非常相似，都可以用来表示一组数量可变的对象应用的集合，并且可以随机的访问其中的元素。

### `HashTable、HashMap、HashSet`

- **HashMap**
  1. 采用数组方式存储key-value构成的Entry对象，无容量限制；
  2. 基于key hash查找Entry对象存放到数组的位置，对于hash冲突采用链表的方式去解决； 
  3. 在插入元素时，可能会扩大数组的容量，在扩大容量时须要重新计算hash，并复制对象到新的数组中； 
  4. **是非线程安全的**； 
  5. 遍历使用的是Iterator迭代器；
- **HashTable**     线程安全
  1. **是线程安全的**； 
  2. 无论是key还是value都不允许有null值的存在；在HashTable中调用Put方法时，如果key为null，直接抛出NullPointerException异常； 
  3. 遍历使用的是Enumeration列举；
- **HashSet** 
  1. 基于HashMap实现，无容量限制； 
  2. **是非线程安全的**； 
  3. 不保证数据的有序；

### `TreeSet、TreeMap`

TreeSet和TreeMap都是完全基于Map来实现的，并且都不支持get(index)来获取指定位置的元素，需要遍历来获取。另外，TreeSet还提供了一些排序方面的支持，例如传入Comparator实现、descendingSet以及descendingIterator等。 

- **TreeSet**
  1. 基于TreeMap实现的，支持排序； 
  2. **是非线程安全的**；
- **TreeMap** 
  1. 典型的基于红黑树的Map实现，因此它要求一定要有key比较的方法，要么传入Comparator比较器实现，要么key对象实现Comparator接口； 
  2. **是非线程安全的**；

### `StringBuffer、StringBulider`

StringBuilder与StringBuffer都继承自AbstractStringBuilder类，在AbstractStringBuilder中也是使用字符数组保存字符串。

　　 1、在执行速度方面的比较：StringBuilder > StringBuffer ； 
　　 2、他们都是字符串变量，是可改变的对象，每当我们用它们对字符串做操作时，实际上是在一个对象上操作的，不像String一样创建一些对象进行操作，所以速度快； 
　 　3、**StringBuilder：线程非安全的**； 
　　 4、**StringBuffer：线程安全的**； 

> 　**对于String、StringBuffer和StringBulider三者使用的总结：** 
> 　　 1.如果要操作少量的数据用 = String 
> 　 　2.单线程操作字符串缓冲区 下操作大量数据 = StringBuilder 
> 　　 3.多线程操作字符串缓冲区 下操作大量数据 = StringBuffer

## 三、JUC 包下的三个数据集合

### `ConcurrentHashMap<K,V>`

**为什么要使用ConcurrentHashMap**

1. 在多线程环境下，使用HashMap，有可能会导致HashMap的Entry链表形成环形数据结构，一旦形成环形数据结构，Entry的next节点永远不为空，就会产生死循环。
2. 虽然可以使用HashTable来应对多线程环境，但是当线程访问HashTable同步方法时，其他线程将进入阻塞或者轮询，所以HashTable的效率十分低下。并且HashTable已经慢慢被淘汰了。
3. 所有访问HashTable的线程都必须竞争同一把锁来获得访问HashTable的权利，但是ConcurrentHashMap使用分段锁技术，将所有数据分段，每段数据配备一把锁，那么当一段数据的锁被获得的时候，其他段的数据依然能够被访问，有效的提高了并发的效率。

### `CopyOnWriteArrayList<E>` 和  `CopyOnWriteArraySet<E>`

一个Set使用内部CopyOnWriteArrayList其所有操作。 因此，它具有相同的基本属性：

- 它最适合于集合大小通常保持较小，只读操作大大超过突变操作的应用程序，并且您需要防止遍历期间线程之间的干扰。
- 它是线程安全的。
- 可变操作（ add ， set ， remove ，等）是昂贵的，因为它们通常意味着复制整个底层数组。
- 迭代器不支持突变remove操作。
- 遍历遍历迭代器是快速的，不能遇到来自其他线程的干扰。 迭代器构建时迭代器依赖于数组的不变快照。

## 四、阻塞队列 BlockingQueue

**一个线程往里边放，另外一个线程从里边取的一个 BlockingQueue。**

- 一个线程将会持续生产新对象并将其插入到队列之中，直到队列达到它所能容纳的临界点。也就是说，它是有限的。如果该阻塞队列到达了其临界点，负责生产的线程将会在往里边插入新对象时发生阻塞。它会一直处于阻塞之中，直到负责消费的线程从队列中拿走一个对象。
- 负责消费的线程将会一直从该阻塞队列中拿出对象。如果消费线程尝试去从一个空的队列中提取对象的话，这个消费线程将会处于阻塞之中，直到一个生产线程把一个对象丢进队列。

**BlockingQueue 的方法**

BlockingQueue 具有 4 组不同的方法用于插入、移除以及对队列中的元素进行检查。如果请求的操作不能得到立即执行的话，每个方法的表现也不同。这些方法如下：

|      | 抛异常     | 特定值   | 阻塞    | 超时                        |
| :--- | :--------- | :------- | :------ | :-------------------------- |
| 插入 | add(o)     | offer(o) | put(o)  | offer(o, timeout, timeunit) |
| 移除 | remove(o)  | poll(o)  | take(o) | poll(timeout, timeunit)     |
| 检查 | element(o) | peek(o)  |         |                             |


四组不同的行为方式解释：

1. **抛异常**：如果试图的操作无法立即执行，抛一个异常。
2. **特定值**：如果试图的操作无法立即执行，返回一个特定的值(常常是 true / false)。
3. **阻塞**：如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行。
4. **超时**：如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行，但等待时间不会超过给定值。返回一个特定值以告知该操作是否成功(典型的是 true / false)。

> 无法向一个 BlockingQueue 中插入 null。如果你试图插入 null，BlockingQueue 将会抛出一个 NullPointerException。

可以访问到 BlockingQueue 中的所有元素，而不仅仅是开始和结束的元素。比如说，你将一个对象放入队列之中以等待处理，但你的应用想要将其取消掉。那么你可以调用诸如 remove(o) 方法来将队列之中的特定对象进行移除。但是这么干效率并不高(译者注：基于队列的数据结构，获取除开始或结束位置的其他对象的效率不会太高)，因此你尽量不要用这一类的方法，除非你确实不得不那么做。