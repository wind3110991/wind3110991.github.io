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