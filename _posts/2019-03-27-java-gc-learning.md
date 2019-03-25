---
layout:     post
title:      聊聊Java的GC机制
date:       2019-03-27 20:12:05
summary:    Java
categories: Java
thumbnail:  jekyll
tags:
 - Java
---

![Thumper](http://ww1.sinaimg.cn/large/afce444dgy1g1an306m9zj20nw0eg3z5.jpg)

使用Java快一年时间了，从最早大学时候对Java的憎恶，到逐渐接受，到工作中体会到了Java开发的各种便捷与福利，这确实是一门不错的开发语言。不仅是Intellij开发Java程序的爽快，还有无需手动管理内存的便捷、Maven管理依赖的整洁、SpringCloud(SpringBoot)大礼包的规整等等。  

所以，作为一个有追求的Java程序员——深入Java底层，了解和掌握GC（垃圾回收）的机制，应该算是必备的技能了。本文即我在学习过程中的一些个人观点以及心得，不正之处敬请指正。 

![Thumper](http://ww1.sinaimg.cn/large/afce444dgy1g1fb6txhe6j20ci0b5ahi.jpg)

### 一、 JVM的运行数据区 
首先我简单来画一张JVM的结构原理图： 
![Thumper](http://ww1.sinaimg.cn/large/afce444dgy1g1an2tdm1jj20v50j5wfb.jpg)

我们重点关注JVM在运行时的数据区，你可以看到在程序运行时，大致有5个部分：

#### （1）方法区 
不止是存“方法”，而是存储整个class文件的信息，JVM运行时，类加载器子系统将会提取`class文件里面的类信息`，并将其存放在方法区中。例如类的名称、类的类型（枚举、类、接口）、字段、方法等等。

#### （2）堆（Heap） 
熟悉c/c++编程的同学们应该相当熟悉Heap了，而对于Java而言，每个应用都唯一对应一个JVM实例，而每一个JVM实例唯一对应一个堆。`堆主要包括关键字new的对象实例、this指针，或者数组都放在堆中，并由应用所有的线程共享`。堆由JVM的自动内存管理机制所管理，名为垃圾回收——GC（garbage collection）。


#### （3）栈（Stack） 
`操作系统内核为某个进程或者线程建立的存储区域`,它保存着一个线程中的方法的调用状态，它具有先进后出的特性。在栈中的数据大小与生命周期严格来说都是确定的，例如在一个函数中声明的int变量便是存储在stack中，它的大小是固定的，在函数退出后它的生命周期也从此结束。在栈中，每一个方法对应一个栈帧，JVM会对Java栈执行两种操作： `压栈和出栈`。 这两种操作在执行时都是以栈帧为单位的。还有一些常量、静态变量、即时编译器编译后的代码等数据。


#### （4）PC寄存器
pc寄存器用于存放一条指令的地址，每一个线程都有一个PC寄存器。

#### （5）本地方法栈
用来调用其他语言的本地方法，例如C/C++写的本地代码， 这些方法在本地方法栈中执行，而不会在Java栈中执行。

### 二、初识GC 
自动垃圾回收机制，简单来说就是寻找Java堆中的无用对象。有趣的是，`JVM并不是使用类似于objective-c的ARC（Automatic Reference Counting）的方式来引用计数对象`，而是使用了叫`GC Roots`的方法，搜索所走过的`引用链（Reference Chain）`。这是一个相当复杂的算法，你不必记住里面的所有细节，但是你要知道的一点就是，可以作为GC Root的对象有四种：
```
1、JVM栈中引用的对象； 
2、方法区中，静态属性引用的对象； 
3、方法区中，常量引用的对象； 
4、本地方法栈中，JNI（即Native方法）引用的对象； 
```

在JDK1.2之后，Java将引用分为`强引用、软引用、弱引用、虚引用`4种，这4种引用强度依次减弱。而虚拟机并不要求方法区一定要实现垃圾回收，除了Java HotSpot这种较新的虚拟机技术，会回收无用的常量和的类，以免大量运用反射这类频繁自定义ClassLoader的操作时方法区溢出。 


### 三、回收算法 

好了，讲了一堆废话，现在我们重点讲下，JVM是如何具体来进行垃圾回收的。

#### (1) Marking
首先，所有堆中的对象都会被扫描一遍，要进行垃圾清理，我们总得知道垃圾是哪些吧。
![Thumper](http://ww1.sinaimg.cn/large/afce444dgy1g1fbewd1p2j20n70dv0t8.jpg)

#### (2) Normal Deletion
垃圾收集器将清除掉标记的对象。
![Thumper](http://ww1.sinaimg.cn/large/afce444dgy1g1fat64esyj20k50dpjs4.jpg)

#### (3) Deletion with Compacting
我们知道，内存有空闲，并不代表着我们就能使用它，例如我们要分配数组这种一段连续空间，假如内存中碎片较多，肯定是行不通的，
所以需要进行压缩，把能使用的内容并到一起——压缩清除的方法就诞生了。
![Thumper](http://ww1.sinaimg.cn/large/afce444dgy1g1faw7limbj20k70czdgo.jpg)


嗯，听起来这样就可以了？但是实际情况下，这样进行GC往往是无法满足我们需求的，无论是效率方面还是开销。所以，`JVM将内存中的Heap部分，进行了分代：新生代，老年代和永久代`。
![Thumper](http://ww1.sinaimg.cn/large/afce444dgy1g1fbdc0qotj20ca0dr7dl.jpg)