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

脏读（Dirty Read）：指一个线程中的事务读取到了另外一个线程中未提交的数据。 

不可重复读（Norepeatable Read）：指一个线程中的事务读取到了另外一个线程中提交的update的数据。例如一个事务读进一条记录，另一个事务更改了这条记录并提交完毕，这时候第一个事务再次读这条记录时，它已经改变了。 

幻读（Phantom Read）：一个事务用Where子句来检索一个表的数据，另一个事务插入一条新的记录，并且符合Where条件，这样，第一个事务用同一个where条件来检索数据后，就会多出一条记录，就像出现了“幻觉”。 

_______________________

### 四、隔离级别 
说实话，我也不想讲太多原理，因为很多概念不仅理解起来很困难，并且很枯燥。所以，我打算用比较形象的方式来聊聊Mysql事务隔离级别。

#### （1）未提交读（Read uncommitted） 
`所有事务都可以看到其他未提交事务的执行结果`。这是隔离级别最低的一个，但是相对的它的并发性能高。相应的会出现脏读、不可重复读、幻读三种现象。

#### （2）已提交读（Read committed）
`一个事务只能看见已经提交的事务所做的修改`。这个级别会锁定当前正在阅读的行，会出现不可重复读、幻读问题。 

#### (3) 可重复读（Repeatable Read）

#### (4) 串行化（Serializable）