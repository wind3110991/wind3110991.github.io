---
layout:     post
title:      深入Mysql：(一)你真的了解索引？
date:       2019-03-11 22:42:22
summary:    Mysql索引
categories: mysql
thumbnail:  jekyll
tags:
 - mysql
---

![Thumper](https://upload.wikimedia.org/wikipedia/zh/thumb/6/62/MySQL.svg/1200px-MySQL.svg.png)


### 一、索引的原理

Mysql我相信大部分人都接触过，但是很多人刚刚开始使用Mysql时，都是随便建表，一堆查询条件瞎搞的。代码日久未生情，却生出了一堆问题，过度使用Mysql与不合理使用索引，往往会导致慢查询等性能问题。 能区分一个低级与高级程序员的关键，就是看他是否能够掌握一门软件的原理，从而合理分析并使用它。

`查询`应该是CRUD中最为频繁的操作，如果是你会怎么来选择查询算法？顺序查询显然是不现实的，二分查找（binary search）、二叉树查找（binary tree search）也许是不错的选择，但是我们要知道，不同的查询算法，也是要对应不同的数据结构的，尚且不可能用一个简单的查找算法来实现，这样是远远不够的。 
所以我们需要设计一个优秀的数据结构，来实现高效的查询算法，而这个数据结构就是我们所说的索引（Index），即用来帮助MySQL高效获取数据的数据结构。

### 二、聊聊B+树

目前大部分数据库系统及文件系统，都是采用B-Tree或其变种B+Tree作为索引结构。下图简单展示了数据库系统的索引结构：
![Thumper](http://ww1.sinaimg.cn/large/afce444dgy1g0z3g9fv5bj20dq05z0t1.jpg)

在B+树的基础上，增加了`顺序访问指针`。那么为什么我们要用这个数据结构呢，为什么不是用红黑树？ 
（1）索引本身也是很大的，需要存在磁盘中，那么我们要做的就是减少磁盘I/O次数； 
（2）根据B树的定义，可知检索一次最多需要访问h个节点。主存和磁盘是以页为单位交换数据，这样的话，我们将一个节点的大小设为等于一个页大小，这样每个节点不就只需要一次I/O就可以完全载入吗？ 
（3）B+树相对是一个扁平的数据结构，它的深度h一般要远低于红黑树的。 

举个例子，现在假设上面的图中，我们需要输出大于等于18的所有数字：
（1）首先我们从第一层根节点看到，15——56之间满足上面的情况，我们从这里继续往下走； 
（2）非叶子节点，所以第二层依然为索引。目标数字18是在15到20之间，继续向下； 
（3）到达叶子节点了，第一个节点中的值为15、18，终于找到18了！那么我们可以推测，后面的数字一定大于18了； 
（4）此时无需再次回到上一层，因为顺序访问指针，已经帮我们指向了下一个比18大的数字，剩下要做的，就是一个个输出他们； 

对于上面的索引本质的探究，推荐大家可以参看[这篇文章][1]。

### 三、MyISAM和InnobDB 

众所周知，Mysql支持很多种存储引擎，其中MyISAM和InnobDB是使用的最为频繁的。

[1]: http://blog.codinglabs.org/articles/theory-of-mysql-index.html

（1）MyISAM索引
MyISAM引擎使用B+树作为索引结构，叶节点的data域存放的是数据记录的地址，我们称MyISAM的索引方式也叫做`非聚集`的。

![Thumper](http://ww1.sinaimg.cn/large/afce444dgy1g0z4rlukopj20j80e43yy.jpg)

（2）InnoDB索引
InnoDB的数据文件本身就是索引文件，表数据文件本身就是按B树组织的一个索引结构，这棵树的叶节点data域保存了完整的数据记录。还有一点很重要，InnoDB的辅助索引data域，存储相应记录`主键的值`，而不是地址。换句话讲，在叶子节点上存储的是主键的值，所以一般最好设置主键为一个整型id比较好，并且这也就代表了用`辅助索引来查询`时，需要先查到主键，再用主键索引查询到值，一共`两次`。

![Thumper](http://ww1.sinaimg.cn/large/afce444dgy1g0z53lfhr6j20d806n0t5.jpg)

### 四、最左前缀原则

（1）聚合索引
我们要先来说下什么是`Clustered Index`。聚合（联合）索引，顾名思义就是用一个组把几个列整合为一个索引。例如我们建一张表：
```
CREATE TABLE `student` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `sex` varchar(128) NOT NULL DEFAULT 'male',
  `name` varchar(128) NOT NULL DEFAULT '',
  `grade` int(11) NOT NULL,
  `age` int(11) NOT NULL,
  PRIMARY KEY (`id`),
  KEY `cluster_index` (`grade`,`name`,`age`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1
```

cluster_index这个值便是一个聚合索引。我们可以用`explain`关键字来对我们的查询语句进行分析。
`explain select * from student where grade="3" and name="Tom" and age="10";`

```
+------+-------------+---------+------+---------------+---------------+---------+-------------------+------+-----------------------+
| id   | select_type | table   | type | possible_keys | key           | key_len | ref               | rows | Extra                 |
+------+-------------+---------+------+---------------+---------------+---------+-------------------+------+-----------------------+
|    1 | SIMPLE      | student | ref  | cluster_index | cluster_index | 138     | const,const,const |    1 | Using index condition |
+------+-------------+---------+------+---------------+---------------+---------+-------------------+------+-----------------------+
```

（2）单列索引