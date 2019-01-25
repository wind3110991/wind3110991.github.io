---
layout:     post
title:      深入浅出c内存管理
date:       2014-06-09 12:32:18
summary:    Transform your plain text into static websites and blogs. Simple, static, and blog-aware.
categories: c memory managemant
thumbnail: jekyll
tags:
 - about
 - jekyll
---

## 一、内存管理
（https://www.jianshu.com/p/b2380e47d005）

c语言的内存模型：
![title](https://yunpan.oa.tencent.com/note/api/file/getImage?fileId=5c4181416f0b932c36c1ac54)

在Linux中，其逻辑地址等于线性地址。因为Linux 所有的段（用户代码段、用户数据段、内核代码段、内核数据段）的段基线性地址都是从 0x00000000 开始，长度4G，这样线性地址 = 0 + 偏移地址，也就是说逻辑地址等于线性地址了。

###（1）栈（stack）
 什么是栈，它是你的电脑内存的一个特别区域，它用来存储被每一个function（包括main（）方法）创建的临时变量。栈是FILO，就是先进后出原则的结构体，它密切的被CPU管理和充分利用。当一个function退出时，所有它的变量都会从栈中弹出,以后都会永远消失

![title](https://yunpan.oa.tencent.com/note/api/file/getImage?fileId=5c41819f6f0b932c36c1ac56)
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

###（2）堆（heap）
a、变量可以被全局访问
b、没有内存大小限制
c、堆内存读出和写入都相对慢,因为它必须使用指针图访问堆内存
d、没有高效地使用空间，随着块内存的创建和销毁，内存可能会变成碎片。
e、你必须管理内存（变量的创建和销毁你必须要负责）
f、变量大小可以用realloc( )调整

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
    
___

###（3）BSS段
Block Started by Symbol的简称，通常是指用来存放程序中未初始化的全局变量和静态变量。

###（4）数据段
通常是指用来存放程序中已初始化的全局变量和静态变量以及字符串常量

###（5）代码段
通常是指用来存放程序执行代码的一块内存区域。这部分区域的大小在程序运行前就已经确定。

(代码段、数据段、BSS段在程序编译期间由编译器分配空间，在程序启动时加载，由于未初始化的全局变量存放在BSS段，已初始化的全局变量存放在数据段，所以程序中应该尽量少的使用全局变量以节省程序编译和启动时间；栈和堆在程序运行中由系统分配空间)

###（6）关于局部变量
局部变量存储细节：由于是a、b是临时变量，因此他们的内存空间分配在栈上，栈中内存寻址由高到低，所以 a 变量的地址比 b 变量的地址要大，其次由于是在64位编译环境中，int 型变量占据4个字节的空间，每一个字节由低到高依次对应着8位二进制数，四个8位二进制数就是十进制中的 1 或 2，而变量a、b的地址就是四个字节中最小值的内存地址。
全局变量存储细节：关于全局变量存储在前面介绍内存组成已经说明，这里不再赘述。
变量的存储类别：C的存储类别包括4种：auto（自动的）、static（静态的）、register（寄存器的）、extern（外部的）。根据变量的存储类别可以得知其作用域和生命周期。


###（7）物理内存与虚拟内存
物理内存就是实实际际存在的内存，程序最终运行的地方。现在的内存管理方法在程序和物理内存之间引入了虚拟内存这个概念，虚拟内存介于程序和物理内存之间，程序只能看见虚拟内存，不能直接访问物理内存，如我们的 malloc、new 等函数开辟的都是虚拟内存空间。每个进程都有自己独立的进程地址空间（虚拟地址），这样就做到了进程隔离。最终都需要将虚拟地址映射到物理地址。内核为每个进程维护不同的页表，不同进程可以虚拟地址一样，但映射后的物理地址不一样


###（8）分页机制
分页机制就是把内存地址空间分为若干个很小的固定大小的页，Linux 中一般页的大小是 ***4KB***，我们把进程的地址空间按页分割，把常用的数据和代码页装载在内存中，不常用的代码和数据则保存在磁盘中。内核只是创建虚拟内存（初始化进程控制表中内存相关的链表），实际上并不立即就把虚拟内存对应位置的程序数据和代码拷贝到物理内存中，只是建立好虚拟内存和磁盘文件间的映射，等到运行到对应的程序时，才会通过缺页异常，调用缺页异常处理程序，从磁盘拷贝数据到对应物理内存


    struct page {
    	unsigned long flags;        /*存放页的状态*/
    	atomic_t _count;        /* 页的引用计数*/
    	union {
    		atomic_t _mapcount; /* Count of ptes mapped in mms,
    							* to show when page is mapped
    							* & limit reverse map searches.
    							*/
    		struct {        /* SLUB */
    			u16 inuse;
    			u16 objects;
    		};
    	};
    	union {
    		struct {
    			unsigned long private;      /* Mapping-private opaque data:
    										* usually used for buffer_heads
    										* if PagePrivate set; used for
    										* swp_entry_t if PageSwapCache;
    										* indicates order in the buddy
    										* system if PG_buddy is set.
    										*/
    			struct address_space *mapping;  /* If low bit clear, points to
    											* inode address_space, or NULL.
    											* If page mapped as anonymous
    											* memory, low bit is set, and
    											* it points to anon_vma object:
    											* see PAGE_MAPPING_ANON below.
    											*/
    		};
     
    	void *virtual;          /* 页的虚拟地址 Kernel virtual address (NULL if
    								not kmapped, ie. highmem) */
    	……
    };


###（9）Heap的内存模型
一般来说，malloc所申请的内存主要从heap区域分配的。

![title](https://yunpan.oa.tencent.com/note/api/file/getImage?fileId=5c45905e6f0b932c36c1b2aa)

linux 内核维护一个break指针，这个指针指向堆空间的某个地址。从堆起始地址（Heap’s Start）到break之间的地址空间为映射好的（虚拟地址与物理地址的映射，通过MMU实现），可以供进程访问；而从break往上，是未映射的地址空间，如果访问这段空间则程序会报错。

所以，如果Mapped Region（虚拟内存至物理内存MMP的部分） 空间不够时，会调整break指针，扩大映射空间，重新分配内存。而调整break指针的函数，就是brk/sbrk。

    int brk(void *addr);
    void *sbrk(intptr_t increment);
    
###（10）MMAP内存映射的原理

1、进程启动映射过程，并在虚拟地址空间中为映射创建Mapped Region——虚拟映射区域。
2、调用内核空间的系统调用函数mmap（不同于用户空间函数），实现文件物理地址和进程虚拟地址的一一映射关系。
3、进程发起对这片映射空间的访问，引发缺页异常，实现文件内容到物理内存（主存）的拷贝。

_____________

# 二、c语言内存管理函数

###（1）malloc
【函数原型】void *malloc(size_t __size)
【参数说明】size 需要分配的内存空间的大小，单位是字节。
【返回值类型】void * 表示未确定类型的指针，分配成功返回指向该内存的地址，失败则返回NULL。C、C++规定，void * 类型可以强制转换为任何其它类型的指针。
【函数功能】 表示向系统申请分配指定 size 个字节的内存空间。例如：
    int *a = malloc(4);  //申请4个字节的空间用于存放一个int类型的值
    char *b = malloc(2);  //申请2个字节的空间用于存放一个char类型的值
###（2）calloc
【函数原型】void *calloc(size_t __count, size_t __size)
【参数说明】count 表示个数，size 单位个需要分配的内存空间的大小，单位是字节。
【返回值类型】void * 表示未确定类型的指针。例如：
【函数功能】 表示向系统申请分配 count 个长度为 size 一共为 count 乘以 size 个字节长度的连续内存空间，并将每一个字节都初始化为 0。
###（3）realloc
【函数原型】void *realloc(void *__ptr, size_t __size)
【参数说明】ptr 表示需要修改的内存空间的地址，size 表示需要重写分配的内存空间的大小，单位是字节。
【返回值类型】void * 表示未确定类型的指针。
【函数功能】 表示更改已经配置好的内存空间到指定的大小。例如：
    char *d = calloc(2, sizeof(char));  //申请2个sizeof(char) 字节的空间
    char *f = realloc(d, 5 * sizeof(char));  //将原来变量d指向的2个sizeof(char) 字节的空间更改到5个sizeof(char) 字节的空间并由变量f指向。
###（4）free
【函数原型】void free(void *)
【参数说明】void * 表示需要释放的内存空间对应的内存地址。
【返回值类型】返回值为空。
【函数功能】 表示用来释放已经动态分配的内存空间。free() 可以释放由 malloc()、calloc()、realloc() 分配的内存空间，以便其他程序再次使用。需要注意的是：free() 不会改变 传入的指针的值，调用 free() 后它仍然会指向相同的内存空间，但是此时该内存已无效，不能被使用。所以建议将释放完的指针设置为 NULL。例如：
    char *g = malloc(sizeof(char)); //申请sizeof(char)大小内存空间
    free(g);      //释放掉g指针指向的内存空间
    g = NULL;     //将g指针指向NULL  
###（5）void
除了free的返回值为空外，其他三个函数的返回值均为void *类型。void应该理解为 “指向空类型” 或者 “不指向确定” 的类型的数据。在将它的值赋给另一个指针变量时由系统对它进行类型转换，使之适合被赋值变量的类型。
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
###（6）数组
数值中存储的元素，是从所占用的低地址开始存储的。
    int main(int argc, const char * argv[]) {
        char chars[4] = {'l','o','v','e'};    
        printf("chars[0] = %p\n",&chars[0]); //0x7fff5fbff79b
        printf("chars[1] = %p\n",&chars[1]); //0x7fff5fbff79c
        printf("chars[2] = %p\n",&chars[2]); //0x7fff5fbff79d
        printf("chars[3] = %p\n",&chars[3]); //0x7fff5fbff79e
        return 0;
    }
![title](https://yunpan.oa.tencent.com/note/api/file/getImage?fileId=5c418a976f0b932c36c1ac96)
数组中的元素按照存放顺序依次从低地址到高地址存放，但是每个元素中的内容又是按高地址向低地址方向存储：
    int main(int argc, const char * argv[]) {
        int nums[2] = {5, 6};
        printf("nums[0] = %p\n",&nums[0]); // 0x7fff5fbff7a0
        printf("nums[1] = %p\n",&nums[1]); // 0x7fff5fbff7a4
        return 0;
    }
![title](https://yunpan.oa.tencent.com/note/api/file/getImage?fileId=5c418b3b6f0b932c36c1acae)
数组在使用过程中遇到的最多的问题可能就是下标越界：
    int main(int argc, const char * argv[]) {
        char charsOne[2] = {'a', 'b'};
        char charsTwo[3] = {'c', 'd', 'e'};
        charsTwo[3] = 'f';
        printf("charsOne[0] = %p\n",&charsOne[0]); // 0x7fff5fbff79e
        printf("charsTwo[0] = %p\n",&charsTwo[0]); // 0x7fff5fbff79b
        printf("charsOne[0] = %c\n",charsOne[0]); // f 
        return 0;
    }
![title](https://yunpan.oa.tencent.com/note/api/file/getImage?fileId=5c418ba16f0b932c36c1acb1)
结合下标越界示意图看上面的的代码会发现，由于越界设置charsTwo[3]元素的值，导致变相更改了charsOne[0]的值。
思考：为什么这里改charsTwo，却最后改了charsOne的值呢？（stack FILO）
____
## 三、理解Free的工作原理
####（1）概述
The more interesting part is how free works (and in this direction, malloc too can be understood better).
>(1) malloc/free cannot handle OS memory or virtual memory, but the mem-blocks that are of a specific size.
In many malloc/free implementations, free does normally ***not return the memory to the operating system*** (or at least only in rare cases). The reason is that ***you will get gaps in your heap*** and thus it can happen, that you just finish off your 2 or 4 GB of virtual memory with gaps. This should be avoided, since as soon as the virtual memory is finished, you will be in really big trouble. The other reason is, that the OS can only handle memory chunks that are of a specific size and alignment. To be specific: Normally the OS can only handle blocks that the virtual memory manager can handle (most often multiples of 512 bytes e.g. 4KB).
So returning 40 Bytes to the OS will just not work. So what does free do?
>(2) Free has its own block list and Malloc will reuse these blocks
Free will put the memory block in its ***own free block list***. Normally it also tries to meld together adjacent blocks in the address space. The free block list is just ***a circular list of memory chunks*** which have some administrative data in the beginning. This is also the reason why managing very small memory elements with the standard malloc/free is not efficient（高效）. Every memory chunk needs additional data and with smaller sizes more fragmentation（分裂） happens.
***The free-list is also the first place that malloc looks at when a new chunk of memory is needed***. It is scanned before it calls for new memory from the OS. When a chunk is found that is bigger than the needed memory, it is divided into two parts. One is returned to caller, the other is put back into the free list.
There are many different optimizations to this standard behaviour (for example for small chunks of memory). But since malloc and free must be so universal(普遍的), the standard behaviour is always the fallback when alternatives are not usable. There are also optimizations in handling the free-list — for example storing the chunks in lists sorted by sizes. But all optimizations also have their own limitations.
>(3) Your Code crash while Free tries to put chunks into the free list, which can touch this administrative-data and therefore stumble over an overwritten pointer.
Why does your code crash:
The reason is that by writing 9 chars (don't forget the trailing null byte) into an area sized for 4 chars:
    char a[4] = {0};
    a = "123456789";
you will probably overwrite the administrative-data stored for another chunk of memory that resides "behind" your chunk of data (since this data is most often stored "in front" of the memory chunks). When free then tries to put your chunk into the free list, it can touch this administrative-data and therefore stumble over an overwritten pointer. This will crash the system.
This is a rather graceful behaviour. I have also seen situations where a runaway pointer somewhere has overwritten data in the memory-free-list and the system did not immediately crash but some subroutines later. Even in a system of medium complexity such problems can be really, really hard to debug! In the one case I was involved, it took us (a larger group of developers) several days to find the reason of the crash -- since it was in a totally different location than the one indicated by the memory dump. It is like a time-bomb. You know, your next "free" or "malloc" will crash, but you don't know why!
Those are some of the worst C/C++ problems, and one reason why pointers can be so problematic.
####（2）Free的实现
    void free(void *p)
    {
        t_block b;
        if(valid_addr(p))//地址的有效性验证
        {
            b = get_block(p);//得到对应的block
            b->free = 1; 
    //如果相邻的上一块内存是空闲的就合并,
    //合并之后的上一块还是空闲的就继续合并，直到不能合并为止
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
                if(b->prev)//当前block前面还有占用的block
                    b->prev->next = NULL;
                else//当前block就是整个heap仅存的
                    base = NULL;//则重置base
                brk(b);//调整break指针到b地址位置
            }
            //否则不能调整break
        }
    }
