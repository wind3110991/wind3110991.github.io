---
layout:     post
title:      深入Mysql：(二)事务的隔离性
date:       2019-03-14 20:12:05
summary:    Mysql事务
categories: mysql
thumbnail:  jekyll
tags:
 - mysql
---

![Thumper](https://www.fengweishang.com/wp-content/uploads/2019/01/mysql-sysnc-2.jpg)

### 一、事务（Transaction）
上一部分介绍了索引在Mysql中的应用以及一些基本的原理，本文我们将就事务的隔离性进行深究。说实话，在我深究Mysql的知识之前，我确实会觉得，这不过就是个很简单的系统，只要会大概用一下就好，但是在踩了许多坑后发现，Mysql其实有很多东西可以细细品味和深究。 

MySQL 事务主要用于处理操作量大，复杂度高的数据。且只有在使用`InnoDB引擎`的表时，对其进行delete、update或者insert操作，才会涉及事务。 

我们来回顾一下事务的`ACID四大条件`：

`原子性（Atomicity，或称不可分割性）、一致性（Consistency）、隔离性（Isolation，又称独立性）、持久性（Durability）` 

原子性：一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。 

一致性：在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。 

隔离性：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。 

持久性：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。 

________________________


### 二、命令行

在Mysql中，默认可以使用下面的命令查看事务的隔离性。 

会话事务的隔离级别：
```select @@tx_isolation;```
![Thumper](http://ww1.sinaimg.cn/large/afce444dgy1g13kjtbj4qj207l03owea.jpg)

系统的隔离级别：
```select @@global.tx_isolation;```

你可以设置当前会话隔离级别：
```set session transaction isolatin level repeatable read;```

或者可以设置系统隔离级别：
```set global transaction isolation level repeatable read;```

除此之外，你还可以查看当前是否是自动提交事务请求的：
```show variables like '%autocommit%';```

你可以在开启事务时设置：`set autocommit=off` 或者 `start transaction`。 

________________________


### 三、隔离现象 
在事务的并发操作中，难免会出现以下的几种现象：

`脏读（Dirty Read）`：指一个线程中的事务读取到了另外一个线程中未提交的数据。 

`不可重复读（Norepeatable Read）`：指一个线程中的事务读取到了另外一个线程中提交的update的数据。例如一个事务读进一条记录，另一个事务更改了这条记录并提交完毕，这时候第一个事务再次读这条记录时，它已经改变了。 

`幻读（Phantom Read）`：一个事务用Where子句来检索一个表的数据，另一个事务插入一条新的记录，并且符合Where条件，这样，第一个事务用同一个where条件来检索数据后，就会多出一条记录，就像出现了“幻觉”。 

_______________________


### 四、隔离级别 
#### （1）未提交读（Read uncommitted） 
`所有事务都可以看到其他未提交事务的执行结果`。这是隔离级别最低的一个，但是相对的它的并发性能高。相应的会出现脏读、不可重复读、幻读三种现象。

#### （2）已提交读（Read committed）
`一个事务只能看见已经提交的事务所做的修改`。这个级别会锁定当前正在阅读的行，会出现不可重复读、幻读问题。 

#### (3) 可重复读（Repeatable Read）
作为MySQL的默认事务隔离级别，这个场景保证了同个事务的多个实例并发请求读取数据时，数据行一定是相同的。但是无法避免出现 `幻读`现象。

#### (4) 串行化（Serializable）
虽然解决了幻读问题，但是最高的隔离级别带来的一定是惨重的代价——大量的超时现象和锁竞争。举个例子：假如AB两个事务都操作到同一数据行，A首先锁定数据行，B只有先等拿到共享锁的事务A做了提交，其他事务才能进行修改操作。

总而言之，我们可以优先考虑将数据库系统的隔离级别设置`Read Committed`。这能够在避免脏读的同时，保证并发性能。在应用程序端主动采用悲观锁或乐观锁来进行事务控制，而不是一味依赖数据库的隔离性进行过粗粒度的操作。

_______________________


### 五、锁与并发 
首先要知道，为什么我们需要锁？因为有并发场景，而并发控制的任务是确保在多个事务同时存取数据库中同一数据时不破坏事务的隔离性和统一性，以及数据库的统一性。 

乐观并发控制（Optimistic Concurrency Control）和悲观并发控制(Pessimistic Concurrency Control)就是并发控制主要采用的技术手段。定义太多难理解，下面只会简述其中的奥妙。

#### （1）悲观锁 
悲观锁的核心就是`阻止一个事务以影响其他用户的方式来修改数据。` 

还是用一串SQL举个栗子，以一个people表和student表为例子，在people表中有一个status字段。现在我们需要修改这个字段，我们重点关注people这个表： 
```
//0.开始事务
start transaction;

//1.查询出学生的信息，for update开启排他锁
select status from people where id = 1 for update;

//2.根据people信息生成学生表（无需关注）
insert into student (name, p_id) values (#{name}, 1);

//3.修改所有人的status为2
update people set status = 2 where id = 1;

//4.提交事务
commit;
```

上面我们引用到了数据库带有的排他锁，下面我们看下它的工作流程： 
（1）关闭autocommit； 
（2）对当前修改记录尝试加上排他锁； 
（3）如果成功，则修改成功，事务结束即解锁； 
（4）反之等待或者处理异常； 
（5）当其他事务企图修改本记录，处理与上面过程一致。 

#### （2）乐观锁 
核心：`各事务能够在不产生锁的情况下，引用版本标识处理各自影响的那部分数据。` 

```
1.查询出people信息
select id, status, name, version from people where id = 1;

2.生成学生信息（无需关注）
insert into student (name, p_id) values (#{name}, 1);

3.修改status为2
update people set status=2, version=version+1 where id=#{id} and version=#{version};
```

我们看到并没有使用数据库自带的锁功能，而是通过业务的逻辑进行了锁的控制，上面流程大致为： 
（1）在设计表时，设计一个版本（时间/标识）字段，用来描述当前记录是什么版本； 
（2）在读取记录时，由于Repeatable Read的原理，这个标识一定为一致的； 
（3）提交更新的时候，判断对应记录版本信息与第一次取出的版本标识是否一致； 
（4）若一致，则修改，反之则失败，发起重试或抛出异常（过期数据）。 

#### （3）CAS算法 
说到这里，对于一个合格的后台开发同学，不得不提及一些课外知识：`CAS算法原理`。 上面的乐观锁使用了`版本控制`的思想，那么什么是CAS呢？ 
`compare and swap（比较&交换）`是一种无锁编程的算法，即不使用锁的情况下实现多线程之间的变量同步，也就是在没有线程被阻塞的情况下实现变量的同步，所以也叫非阻塞同步（Non-blocking Synchronization）。CAS算法涉及到三个操作数：`需要读写的内存值V、 进行比较的值A、拟写入的新值B`。当且仅当V的值等于A时，CAS通过原子方式用新值B来更新V的值，否则不会执行任何操作，即一个不停的自旋操作，当然它如果不成功就一直循环执行直到成功，如果长时间不成功，会给CPU带来非常大的执行开销。 

> 对于资源竞争较少的情况，CAS基于硬件实现，不需要进入内核，不需要切换线程，可以使用到这个场景中。 

> Java语言中，对于资源竞争严重（线程冲突严重）的情况，CAS自旋的概率会比较大，会浪费更多的CPU资源，另一个关键字`synchronized`同步锁适用于这种写比较多的情况。

#### （4）ABA问题 
上面的问题看似都完美解决了？等等，还有一个乐观锁很经典的问题——`ABA问题。`
让我用一张我自己画的丑图来稍微解释下：

![Thumper](http://ww1.sinaimg.cn/large/afce444dgy1g19ge031ibj20ys0j0q45.jpg)

有没有稍微理解上面图中带来的问题是什么？线程3在被唤醒前后，余额都为100，但是对于线程3本身而言并不知道【已进行过了扣款，而有人又汇款了50元过来】这件事，这就是ABA问题，一个线程在处理V这个变量时它的值为A，中途可能被挂起过，那么当再次访问V时，可能这个值中途已变为了B，但是最后又变成了A，线程本身是无法得知的，那么在进行操作时，可能会导致各种各样的问题（例如上面线程3又将余额改为了3，有50元就从账户中平白无故消失了）。

因为业务逻辑存在回退的可能性，那么如果加入一个与业务逻辑不相关的属性，比如在一个数据中加入版本号，约定只要修改了数据该版本号就会递增，且不会回退，那么ABA问题就解决了。JDK的`atomic`包里，提供了一个类`AtomicStampedReference`，使用`compareAndSet`方法可以避免这个问题，我们看下它的定义：

```
/**
 *expectedReference - 该引用的预期值
 *newReference - 该引用的新值
 *expectedStamp - 该标志的预期值
 *newStamp - 该标志的新值
 */
public boolean compareAndSet(V   expectedReference,
                                 V   newReference,
                                 int expectedStamp,
                                 int newStamp) {
        Pair<V> current = pair;
        return
            expectedReference == current.reference &&
            expectedStamp == current.stamp &&
            ((newReference == current.reference &&
              newStamp == current.stamp) ||
             casPair(current, Pair.of(newReference, newStamp)));
    }
```

可以通过对引用参数设置标志，如果当前引用等于预期引用，并且当前标志等于预期标志，则用原子方式将该引用和该标志的值设置为给定的更新值。 

### 六、尾声 
其实上文也是在使用和运营Mysql的过程中，有了许多了体会，如有错误之处欢迎评论或邮件我指正。预计后面会基于本文的内容，谈谈死锁的产生与分析思路。