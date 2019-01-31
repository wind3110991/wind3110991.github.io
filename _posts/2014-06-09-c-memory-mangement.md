---
layout:     post
title:      深入浅出c内存管理
date:       2019-01-09 22:32:18
summary:    就算是个API Boy，了解C的内存管理能够大大提升自己的内功
categories: c/c++
author:     Wind Young
thumbnail:  jekyll
tags:
 - c/c++
 - 内存管理
---

之前看了一篇对c/c++内存管理描述的[很不错的文章][1]，后来想来，自己也其实在这方面研究尚浅，故在此回味一下。  

我们先用一张图来归纳c语言的内存模型：
![Thumper](http://ww1.sinaimg.cn/large/afce444dgy1fzprj4e66sj20er0cat9l.jpg)

_____

###  一、内存管理

在Linux中，其逻辑地址等于线性地址。因为Linux 所有的段（用户代码段、用户数据段、内核代码段、内核数据段）的段基线性地址都是从 0x00000000 开始，长度4G，这样线性地址 = 0 + 偏移地址，也就是说逻辑地址等于线性地址了。


#### （1）栈（stack）
 什么是栈，它是你的电脑内存的一个特别区域，它用来存储被每一个function（包括main（）方法）创建的临时变量。栈是FILO，就是先进后出原则的结构体。它密切的被CPU管理和充分利用。当一个function退出时，所有它的变量都会从栈中弹出,以后都会永远消失

![Thumper](http://ww1.sinaimg.cn/large/afce444dgy1fzprlt1ajzj20et0ajwf6.jpg)
<small> (我很喜欢用上手枪弹夹的方式来描述这个概念)</small>  
<br/>

关于栈，我总结为三点：  
a、栈的生长和伸缩就是函数压入或者推出局部变量。  
b、我们不用自己去管理内存，变量创建和释放都是自动的。  
c、栈中的变量只有在函数创建运行时才会存在。  

    #include <stdio.h>
    int main(int argc, const char * argv[]) {
        int a = 100;
        int b = 100;
        printf("%p \n",&a); // 0x7fff5fbff79c
        printf("%p \n",&b); // 0x7fff5fbff798
        
        // a 变量的地址 0x7fff5fbff79c 比 b
        变量的地址 0x7fff5fbff798 要大
        return 0;
    }
    
______

#### （2）堆（heap）  

a、变量可以被全局访问；  
b、没有内存大小限制；  
c、堆内存读出和写入都相对慢,因为它必须使用指针图访问堆内存；  
d、没有高效地使用空间，随着块内存的创建和销毁，内存可能会变成碎片；
e、你必须管理内存（变量的创建和销毁你必须要负责）；  
f、变量大小可以用realloc()调整；  

    #include <stdio.h>
    int a = 0;                  // 全局初始化区
    char p1;                    // 全局未初始化区
    int main(int argc, const char * argv[]) {
          int b ;                 // 栈
          char s[] = "abc";       // 栈
          char p2 ;               // 栈
          char p3 = "123456";     // 123456在常量区，p3在栈上。
          static int c = 0 ;      // 全局（静态）初始化区
          
          p1 = (char )malloc(10); // 分配的10字节的区域就在堆区
          p2 = (char )malloc(20); // 分配的20字节的区域就在堆区
          
          printf("%p\n",p1);      // 0xffffffb0
          printf("%p\n",p2);      // 0xffffffc0
          
          return 0;                
    //p1 变量的地址 0xffffffb0 比 p2 变量的地址 0xffffffc0 要小
    }

#### （3）BSS段
Block Started by Symbol的简称，通常是指用来存放程序中未初始化的全局变量和静态变量。

#### （4）数据段
通常是指用来存放程序中已初始化的全局变量和静态变量以及字符串常量

#### （5）代码段
通常是指用来存放程序执行代码的一块内存区域。这部分区域的大小在程序运行前就已经确定。
(代码段、数据段、BSS段在程序编译期间由编译器分配空间，在程序启动时加载，由于未初始化的全局变量存放在BSS段，已初始化的全局变量存放在数据段，所以程序中应该尽量少的使用全局变量以节省程序编译和启动时间；栈和堆在程序运行中由系统分配空间)

#### （6）关于局部变量
局部变量存储细节：由于是a、b是临时变量，因此他们的内存空间分配在栈上，栈中内存寻址由高到低，所以 a 变量的地址比 b 变量的地址要大，其次由于是在64位编译环境中，int 型变量占据4个字节的空间，每一个字节由低到高依次对应着8位二进制数，四个8位二进制数就是十进制中的 1 或 2，而变量a、b的地址就是四个字节中最小值的内存地址。
全局变量存储细节：关于全局变量存储在前面介绍内存组成已经说明，这里不再赘述。  

变量的存储类别：
    
    C的存储类别包括4种：auto（自动的）、static（静态的）、register（寄存器的）、extern（外部的）。  

根据变量的存储类别可以得知其作用域和生命周期。


#### （7）物理内存与虚拟内存
物理内存就是实实际际存在的内存，程序最终运行的地方。现在的内存管理方法在程序和物理内存之间引入了虚拟内存这个概念，虚拟内存介于程序和物理内存之间，程序只能看见虚拟内存，不能直接访问物理内存，如我们的 malloc、new 等函数开辟的都是虚拟内存空间。每个进程都有自己独立的进程地址空间（虚拟地址），这样就做到了进程隔离。最终都需要将虚拟地址映射到物理地址。内核为每个进程维护不同的页表，不同进程可以虚拟地址一样，但映射后的物理地址不一样


#### （8）分页机制
分页机制就是把内存地址空间分为若干个很小的固定大小的页，Linux 中一般页的大小是 ***4KB***，我们把进程的地址空间按页分割，把常用的数据和代码页装载在内存中，不常用的代码和数据则保存在磁盘中。内核只是创建虚拟内存（初始化进程控制表中内存相关的链表），实际上并不立即就把虚拟内存对应位置的程序数据和代码拷贝到物理内存中，只是建立好虚拟内存和磁盘文件间的映射，等到运行到对应的程序时，才会通过缺页异常，调用缺页异常处理程序，从磁盘拷贝数据到对应物理内存。

用一个结构体来实现page，大概如下：

    struct page {
    	unsigned long flags;        /*存放页的状态*/
    	atomic_t _count;        /* 页的引用计数*/
    	union {
    		atomic_t _mapcount; 
    		struct {
    			u16 inuse;
    			u16 objects;
    		};
    	};
    	union {
    		struct {
    			unsigned long private;
    			struct address_space *mapping;
    		};
     
    	void *virtual;  /* 页的虚拟地址 Kernel virtual address (NULL if
    					not kmapped, ie. highmem) */
    	……
    };


#### （9）Heap的内存模型
一般来说，malloc所申请的内存主要从heap区域分配的。

![Thumper](http://ww1.sinaimg.cn/large/afce444dgy1fzprp9ukjsj20fg04jdg9.jpg)
<small> (Heap的基本构成)</small >  
<br/>

linux 内核维护一个break指针，这个指针指向堆空间的某个地址。从堆起始地址（Heap’s Start）到break之间的地址空间为映射好的（虚拟地址与物理地址的映射，通过MMU实现），可以供进程访问；而从break往上，是未映射的地址空间，如果访问这段空间则程序会报错。

所以，如果Mapped Region（虚拟内存至物理内存MMP的部分） 空间不够时，会调整break指针，扩大映射空间，重新分配内存。而调整break指针的基础API，就是brk/sbrk。

    int brk(void *addr);
    void *sbrk(intptr_t increment);
    
#### （10）MMAP内存映射的原理

1、进程启动映射过程，并在虚拟地址空间中为映射创建Mapped Region——虚拟映射区域。
2、调用内核空间的系统调用函数mmap（不同于用户空间函数），实现文件物理地址和进程虚拟地址的一一映射关系。
3、进程发起对这片映射空间的访问，引发缺页异常，实现文件内容到物理内存（主存）的拷贝。

_____________

### 二、c语言内存管理函数  

#### （1）malloc
    【函数原型】 void *malloc(size_t __size)  
    【参数说明】 size 需要分配的内存空间的大小，单位是字节。  
    【返回值类型】 void * 表示未确定类型的指针，分配成功返回指向该内存的地址，失败则返回NULL。C、C++规定，void* 类型可以强制转换为任何其它类型的指针。  
    【函数功能】 表示向系统申请分配指定 size 个字节的内存空间。

    int *a = malloc(4);  //申请4个字节的空间用于存放一个int类型的值
    char *b = malloc(2);  //申请2个字节的空间用于存放一个char类型的值


#### （2）calloc
    【函数原型】 void *calloc(size_t __count, size_t __size)  
    【参数说明】 count 表示个数，size 单位个需要分配的内存空间的大小，单位是字节。  
    【返回值类型】 void * 表示未确定类型的指针。
    【函数功能】 表示向系统申请分配 count 个长度为 size 一共为 count 乘以 size 个字节长度的连续内存空间，并将每一个字节都初始化为 0。


#### （3）realloc
    【函数原型】 void *realloc(void *__ptr, size_t __size)  
    【参数说明】 ptr 表示需要修改的内存空间的地址，size 表示需要重写分配的内存空间的大小，单位是字节。  
    【返回值类型】 void * 表示未确定类型的指针。  
    【函数功能】 表示更改已经配置好的内存空间到指定的大小。

    char *d = calloc(2, sizeof(char));  //申请2个sizeof(char) 字节的空间
    char *f = realloc(d, 5 * sizeof(char));  //将原来变量d指向的2个sizeof(char) 字节的空间更改到5个sizeof(char) 字节的空间并由变量f指向。

#### （4）free
    【函数原型】 void free(void *)
    【参数说明】 void * 表示需要释放的内存空间对应的内存地址。
    【返回值类型】 返回值为空。
    【函数功能】 表示用来释放已经动态分配的内存空间。free() 可以释放由 malloc()、calloc()、realloc() 分配的内存空间，以便其他程序再次使用。需要注意的是：free() 不会改变 传入的指针的值，调用 free() 后它仍然会指向相同的内存空间，但是此时该内存已无效，不能被使用。所以建议将释放完的指针设置为 NULL。

    char *g = malloc(sizeof(char)); //申请sizeof(char)大小内存空间
    free(g);      //释放掉g指针指向的内存空间
    g = NULL;     //将g指针指向NULL


#### （5）额外提一下void

除了free的返回值为空外，其他三个函数的返回值均为void* 类型。void应该理解为 “指向空类型” 或者 “不指向确定” 的类型的数据。在将它的值赋给另一个指针变量时由系统对它进行类型转换，使之适合被赋值变量的类型。

    int main(int argc, const char * argv[]) 
    {
        int a = 3;                   //定义a为整型变量
        int *p1 = &a;                //p1指向 int 型变量
        char *p2;                    //p2指向 char 型变量
        void *p3;               
        //p3为无类型指针变量
        p3 = (void *)p1; //将p1的值转换为void *类型，然后赋值给p3
        p2 = (char *)p3;    //将p3的值转换为char *类型，然后赋值给p2
        printf("%d\n", *p1); //输出a的值 3
        p3 = &a;                    
        printf("%d", *p3); //此处报错，p3无指向，不能指向a 
        return 0;
    }

_____________

### 三、关于数组  

####  (1) 数值中存储的元素，是从所占用的低地址开始存储的。

    int main(int argc, const char * argv[]) {
        char chars[4] = {'l','o','v','e'};    
        printf("chars[0] = %p\n",&chars[0]); //0x7fff5fbff79b
        printf("chars[1] = %p\n",&chars[1]); //0x7fff5fbff79c
        printf("chars[2] = %p\n",&chars[2]); //0x7fff5fbff79d
        printf("chars[3] = %p\n",&chars[3]); //0x7fff5fbff79e
        return 0;
    }

![Thumper](http://ww1.sinaimg.cn/large/afce444dgy1fzpruuetbpj20ep06cq4z.jpg)
<small> (数组的地址排列1)</small >   
<br/>

#### (2) 数组中的元素按照存放顺序依次从低地址到高地址存放，但是每个元素中的内容又是按高地址向低地址方向存储：

    int main(int argc, const char * argv[]) {
        int nums[2] = {5, 6};
        printf("nums[0] = %p\n",&nums[0]); // 0x7fff5fbff7a0
        printf("nums[1] = %p\n",&nums[1]); // 0x7fff5fbff7a4
        return 0;
    }

![Thumper](http://ww1.sinaimg.cn/large/afce444dgy1fzprvfkbsoj20h608xq6d.jpg)
<small> (数组的地址排列2)</small >  
<br/>

#### (3) 数组在使用过程中遇到的最多的问题可能就是下标越界


    int main(int argc, const char * argv[]) {
        char charsOne[2] = {'a', 'b'};
        char charsTwo[3] = {'c', 'd', 'e'};
        charsTwo[3] = 'f';
        printf("charsOne[0] = %p\n",&charsOne[0]); // 0x7fff5fbff79e
        printf("charsTwo[0] = %p\n",&charsTwo[0]); // 0x7fff5fbff79b
        printf("charsOne[0] = %c\n",charsOne[0]); // f 
        return 0;
    }

![Thumper](http://ww1.sinaimg.cn/large/afce444dgy1fzprvq5h2wj20gw08otbq.jpg)
<small> (数组的地址排列3)</small >  
<br/>

结合下标越界示意图看上面的的代码会发现，由于越界设置charsTwo[3]元素的值，导致变相更改了charsOne[0]的值。
思考：为什么这里改charsTwo，却最后改了charsOne的值呢？（提示：stack FILO)

_____________

### 四、理解Free的工作原理  

说实话，我研究本文的内容，其实也是因为这个问题 [《malloc申请得到的内存后free释放，操作系统会立即收回那块内存吗》][3] 开始的。  
在研究了很多回答后，我得到的最好答案 [在这里][2]，有兴趣可以细读一下。下面我做一个带我理解的简单转述：

#### （1）free的内存并非立即归还OS
The more interesting part is how free works (and in this direction, malloc too can be understood better).
a) malloc/free 不可以直接操作 OS memory/virtual memory, 在linux OS中，每一个内存块实际上都是特定大小的块（block/chunk）。实际上在大部分OS里，我们都无法直接操作操作系统的内存，即物理内存。假设我们可以操作，会带来什么后果呢？由于内存归还（大小、地址）的不确定性，我们的内存中会产生大量的gap。  

这也是为什么我们的OS只能处理特定大小的chunks（一般为512bytes的倍数，比如4KB)，并且需要alignment对齐。  
那么问题来了： 我就是要归还40byte给OS，这时怎么办呢？


####  (2) free维护了自己的块列表

free维护了自己的块列表，通常它也会尝试将地址空间中的相邻块merge在一起。空闲块列表只是内存块的循环列表，其中包含一些administrative-data。这也是为什么使用标准malloc/free管理非常小的内存元素效率不高的原因。每个内存块都需要额外的数据，而更小的大小会产生更多碎片。  

当我们需要新的内存块时，free空闲列表也是malloc看到的第一个部分。它在从OS调用新内存之前进行扫描。当发现大于所需内存的块时，它被分成两部分。一个返回给调用者，另一个返回到空闲列表中。

#### (3) 所以你理解了为啥C的代码老是奔溃了吧？

作为一个苦逼的c/c++程序员，每当delete/free操作时，根据上面的原理，我们会将回收的块放入空闲列表，这可能会触及到free列表中的administrative-data，因此出现覆盖指针的问题。  
举个栗子，我们将9个字符（不要忘记尾部留空字节）写入一个大小为4个字符的区域：

    char a[4] = {0};
    a = "123456789";

当程序core掉时，其实往往可能能够完成一部分工作，然后才gg的。这是因为，当发生指针覆盖时，指针覆盖了另一块内存的地址，在free释放这个指针之前，你仍然可以访问到这个指针指向的区域。  
你可能会覆盖存储在另一块内存中的administrative-data，这些内存位于您的数据块“后面”（因为这些数据通常存储在内存块的“前面”，可以参考上面数组的那个情况）。如果空闲然后尝试将您的块放入空闲列表，它可以触摸此administrative-data，最终这会使系统崩溃。  


#### （4）Free的基本实现

    void free(void *p)
    {
        t_block b;
        if(valid_addr(p))//地址的有效性验证
        {
            b = get_block(p);//得到对应的block
            b->free = 1; //如果相邻的上一块内存是空闲的就合并, 合并之后的上一块还是空闲的就继续合并，直到不能合并为止
            while(b->prev && b->prev->free)
            {
                b = fusion(b->prev);
            } 
    
            //同理去合并后面的空闲block
            while(b->next)
                fusion(b);//内部会判断是否空闲
    
            //如果当前block是最后面的那个block，此时可以调整break指针了
            if(NULL == b->next)
            {
                if(b->prev) // 当前block前面还有占用的block
                    b->prev->next = NULL;
                else       // 当前block就是整个heap仅存的
                    base = NULL; // 则重置base
                brk(b); //调整break指针到b地址位置
            }

            //否则不能调整break
        }
    }


### 五、总结  
要理解c/linux的内存管理机制，的确是一个让人头大的事情。可能这属于很容易让人过目即忘的内容，但是在工作之后，还是得有时间来细细品味，修炼内功对自己的成长可能短时间内不是很多，但是长期积累下去，总会有一些建树和收获。

[1]: https://www.jianshu.com/p/b2380e47d005
[2]: https://stackoverflow.com/questions/1119134/how-do-malloc-and-free-work
[3]: https://www.v2ex.com/t/181639#reply21