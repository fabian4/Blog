---
title: Mysql 隔离级别与MVCC
date: 2021/4/19
description: Mysql 隔离级别与MVCC
top_img: >-
  https://fabian.oss-cn-hangzhou.aliyuncs.com/img/CuscoCathedral_ZH-CN9834821723_1920x1080.jpg
cover: >-
  https://fabian.oss-cn-hangzhou.aliyuncs.com/img/CuscoCathedral_ZH-CN9834821723_1920x1080.jpg
categories:
  - Mysql
tags:
  - Mysql
  - MVCC
abbrlink: 10729
---

# Mysql 隔离级别与MVCC

> 以下内容都是基于 5.7 的 MySQL 的 innode 的存储引擎

## 一、问题的出现

- 读-读：没有并发问题
- 读-写：会造成数据不一致  
- 写-写：造成数据丢失问题

**很明显在两个事务同时读操作，是不存在线程并发安全问题。但当一旦开始出现了写操作，那么就会开始干扰另一个事务了。**

为了解决 **读-写** 和 **写-写** 这两个场景下的问题，innodb 引入了 **MVCC** 和 **锁机制** 来保证事务的隔离

## 二、隔离级别

这里我们引入一个场景再来看看事务的隔离级别

~~~sql
create table T(c int) engine=InnoDB;
insert into T(c) values(1);
~~~

|     事务A      |     事务B      |
| :------------: | :------------: |
|    启动事务    |                |
| 查询得到结果 1 |                |
|                |    启动事务    |
|                | 查询得到结果 1 |
|                |  将 1 改成 2   |
|  查询得到值 a  |                |
|                |    提交事务    |
|  查询得到值 b  |                |
|    提交事务    |                |
|  查询得到值 c  |                |

**在不同隔离级别下 a b c 的值：**

- **读未提交** `Read UnCommitted`：

  - a：2
  - b：2
  - c：2

  事务A 总是能直接读到 事务B 未提交的内容，那么查询到的就一直是最新的数据

- **读已提交** `Read Committed`：

  - a：1
  - b：2
  - c：2

  事务A 只能读到 事务B 已提交的内容，那么在提交之后就能读到新的值

- **可重复读** `Repeatable Read`：

  - a：1
  - b：1
  - c：2

  事务A 在可重复读 的隔离级别下，能保证在同一个事务中读到的值是不受其他事务影响的

- **串行化** `Serializable`：

  - a：1
  - b：1
  - c：2

  在串行化的隔离级别下，事务B 的修改操作会被 事务A 阻塞住，这样知道 事务A 提交才会继续执行，所以 事务A 查询之后任然是初始的值

> 在实现上，数据库里面会创建一个视图，访问的时候以视图的逻辑结果为准。
>
> - 在“可重复读”隔离级别下，这个视图是在事务启动时创建的，整个事务存在期间都用这个视图。
> - 在“读提交”隔离级别下，这个视图是在每个 SQL 语句开始执行的时候创建的。
> - 这里需要注意的是，“读未提交”隔离级别下直接返回记录上的最新值，没有视图概念；
> - 而“串行化”隔离级别下直接用加锁的方式来避免并行访问。

我们可以通过 show variables 来查看当前的隔离级别

~~~bash
mysql> show variables like 'transaction_isolation';
+-----------------------+----------------+
| Variable_name | Value |
+-----------------------+----------------+
| transaction_isolation | READ-COMMITTED |
+-----------------------+----------------+
~~~

## 三、MVCC 及 实现

### 1. 当前读和快照读

- 关于当前读：

  每一次数据的查询都是基于目前最新版本的数据，这种很显然是最简单的一种实现方式。但是由于当前读，遇到多事务并行的场景的时候，很明显就会出现各种各样的问题。
  除非把数据库的隔离级别调整到最高即**串行化**：所有事务串行执行，那很明显就大大的降低了数据库的并发性能。

- 快照读：

  为了保证事务的隔离级别，在每一个事务开始之前对当前的数据创建一个快照，这样在数据读取的过程中，只读自己事务的快照。**这样其他事务的写入就不会影响自己的读取，也就不会产生一些脏读和幻读的现象。**

### 2. MVCC（Mutil-Version Concurrency Control）

> MVCC(Mutil-Version Concurrency Control)，就是多版本并发控制。MVCC 是一种并发控制的方法，一般在数据库管理系统中，实现对数据库的并发访问。
>
> 在Mysql的InnoDB引擎中就是指在 **已提交读(READ COMMITTD)** 和 **可重复读(REPEATABLE READ)** 这两种隔离级别下的事务对于SELECT操作会**访问版本链中的记录**的过程。

- 版本链

  这里的版本链就是 **快照读** 在 innodb 中的具体的实现方式。

  在每一个事务开始的时候创建一个版本分支，每一次该事务的读或者写都是在各自对应的版本链中实现，从而可以达到事务之间的隔离。

- 对于版本链的引入，可以在**不加锁**的情况下实现读事务，进而大大的提高数据库的并发性能。

### 3. 基于 Undo log 的版本链

> 如果快照读的实现是在事务开始的时候对数据库做一个全部的快照，那么在高并发场景下的内存开销一定是灾难

InnoDB 的 MVCC是通过在每行记录后面保存两个隐藏的列来实现的。

- **一个保存了行的事务ID（DB_TRX_ID）**

  每行数据也都是有多个版本的，每次事务更新数据的时候，都会生成一个新的数据版本，并且把`transaction id`赋值给这个数据版本的事务ID，记为`row trx_id`。同时，旧的数据版本要保留，并且在新的数据版本中，能够有信息可以直接拿到它。

  也就是说，数据表中的一行记录，其实可能有多个版本(row)，每个版本有自己的`row trx_id`。

- **一个保存了行的回滚指针（DB_ROLL_PT）**。

  每次对某条聚簇索引记录进行改动时，都会把旧的版本写入到`undo日志`中，然后这个隐藏列就相当于一个指针，可以通过它来找到该记录修改前的信息

![image-20211007151855474](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/image-20211007151855474.png)

`undo log`的回滚机制也是依靠这个版本链，每次对记录进行改动，都会记录一条undo日志，每条undo日志也都有一个`roll_pointer`属性（INSERT操作对应的undo日志没有该属性，因为该记录并没有更早的版本），可以将这些undo日志都连起来，串成一个链表，所以现在的情况就像下图一样

![image-20211007152027329](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/image-20211007152027329.png)

### 4. **insert undo log** 和 **update undo log**



> 根据行为的不同，undo log分为两种：**insert undo log** 和 **update undo log**

- **insert undo log：**

  insert 操作中产生的undo log，因为insert操作记录只对当前事务本身课件，对于其他事务此记录不可见，所以 insert undo log 可以在事务提交后直接删除而不需要进行purge操作。

  > purge的主要任务是将数据库中已经 mark del 的数据删除，另外也会批量回收undo pages

  数据库 Insert时的数据初始状态：![image-20211007155050351](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/image-20211007155050351.png)

- **update undo log：**

  update 或 delete 操作中产生的 undo log。因为会对已经存在的记录产生影响，为了提供 MVCC机制，因此update undo log 不能在事务提交时就进行删除，而是将事务提交时放到入 history list 上，等待 purge 线程进行最后的删除操作。

  **数据第一次被修改时：**

  ![image-20211007155107177](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/image-20211007155107177.png)

  **当另一个事务第二次修改当前数据：**

  ![image-20211007155121264](https://fabian.oss-cn-hangzhou.aliyuncs.com/img/image-20211007155121264.png)

> 为了保证事务并发操作时，在写各自的undo log时不产生冲突，InnoDB采用回滚段的方式来维护undo log的并发写入和持久化。回滚段实际上是一种 Undo 文件组织方式。

### 5. 可重复读隔离级别下的增删改查

#### **SELECT**

- InnoDB 会根据以下两个条件检查每行记录：
  1. InnoDB只查找版本早于当前事务版本的数据行（也就是，行的事务编号小于或等于当前事务的事务编号），这样可以确保事务读取的行，要么是在事务开始前已经存在的，要么是事务自身插入或者修改过的。
  2. 删除的行要事务ID判断，读取到事务开始之前状态的版本，只有符合上述两个条件的记录，才能返回作为查询结果。

#### **INSERT**

- InnoDB为新插入的每一行保存当前事务编号作为行版本号。

#### **DELETE**

- InnoDB为删除的每一行保存当前事务编号作为行删除标识。

#### **UPDATE**

- InnoDB为插入一行新记录，保存当前事务编号作为行版本号，同时保存当前事务编号到原来的行作为行删除标识。

## 四、ReadView

> 对于 **RU(READ UNCOMMITTED)** 隔离级别下，所有事务直接读取数据库的最新值即可，和 **SERIALIZABLE** 隔离级别，所有请求都会加锁，同步执行。所以这对这两种情况下是不需要使用到 **Read View** 的版本控制。

对于 **RC(READ COMMITTED)** 和 **RR(REPEATABLE READ)** 隔离级别的实现就是通过上面的版本控制来完成。

两种隔离界别下的核心处理逻辑就是判断所有版本中哪个版本是当前事务可见的处理。

针对这个问题InnoDB在设计上增加了**ReadView**的设计，**ReadView**中主要包含当前系统中还有哪些活跃的读写事务，把它们的事务id放到一个列表中，我们把这个列表命名为为**m_ids**。

对于查询时的版本链数据是否看见的判断逻辑：

- 如果被访问版本的 trx_id 属性值小于 m_ids 列表中最小的事务id，表明生成该版本的事务在生成 ReadView 前已经提交，所以该版本可以被当前事务访问。
- 如果被访问版本的 trx_id 属性值大于 m_ids 列表中最大的事务id，表明生成该版本的事务在生成 ReadView 后才生成，所以该版本不可以被当前事务访问。
- 如果被访问版本的 trx_id 属性值在 m_ids 列表中最大的事务id和最小事务id之间，那就需要判断一下 trx_id 属性值是不是在 m_ids 列表中，如果在，说明创建 ReadView 时生成该版本的事务还是活跃的，该版本不可以被访问；如果不在，说明创建 ReadView 时生成该版本的事务已经被提交，该版本可以被访问。

## 五、事务的启动

MySQL 的事务启动方式有以下两种：

1. 显式启动事务语句， begin 或 start transaction。配套的提交语句是 commit，回滚语句是 rollback。
2. set autocommit=0，这个命令会将这个线程的自动提交关掉。意味着如果你只执行一个 select 语句，这个事务就启动了，而且并不会自动提交。这个事务持续存在直到你主动执行 commit 或 rollback 语句，或者断开连接。

有些客户端连接框架会默认连接成功后先执行一个 set autocommit=0 的命令。这就导致接下来的查询都在事务中，如果是长连接，就导致了意外的长事务。

> 因此，我会建议你总是使用 set autocommit=1, 通过显式语句的方式来启动事务。
>
> 但是有的开发同学会纠结“多一次交互”的问题。对于一个需要频繁使用事务的业务，第二种方式每个事务在开始时都不需要主动执行一次 “begin”，减少了语句的交互次数。如果你也有这个顾虑，我建议你使用 commit work and chain 语法。
>
> 在 autocommit 为 1 的情况下，用 begin 显式启动的事务，如果执行 commit 则提交事务。如果执行 commit work and chain，则是提交事务并自动启动下一个事务，这样也省去了再次执行 begin 语句的开销。同时带来的好处是从程序开发的角度明确地知道每个语句是否处于事务中。

你可以在 information_schema 库的 innodb_trx 这个表中查询长事务，比如下面这个语句，用于查找持续时间超过 60s 的事务。

~~~sql
select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60
~~~

## 六、总结

所谓的MVCC（Multi-Version Concurrency Control ，多版本并发控制）指的就是：

在使用 **READ COMMITTD** 、**REPEATABLE READ** 这两种隔离级别的事务在执行普通的 SEELCT 操作时访问记录的版本链的过程，这样子可以使不同事务的 `读-写` 、 `写-读` 操作并发执行，从而提升系统性能。

在 MySQL 中， READ COMMITTED 和 REPEATABLE READ 隔离级别的的一个非常大的区别就是**它们生成 ReadView 的时机不同。**

- 在 READ COMMITTED 中每次查询都会生成一个实时的 ReadView，做到保证每次提交后的数据是处于当前的可见状态。
- REPEATABLE READ 中，在当前事务第一次查询时生成当前的 ReadView，并且当前的 ReadView 会一直沿用到当前事务提交，以此来保证可重复读（REPEATABLE READ）。