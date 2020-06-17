---
author: "Lingyu Zhang"
author_link: "https://jarviszly.com"
title: "DemandPaging"
date: 2020-06-17T11:04:30+08:00
lastmod: 2020-06-17T11:04:30+08:00
draft: false
description: ""
show_in_homepage: true
description_as_summary: false
license: ""

tags: []
categories: [操作系统]

featured_image: ""
featured_image_preview: ""

comment: true
toc: true
auto_collapse_toc: true
math: true
---

## Demand Paging(按需求分配页面)
- mmap实际上只是做了一些标记，并没有实际上的分配物理内存，所以mmap的速度很快。
- 实际上，在execve重置进程的时候，此时只需要
  - 在物理内存中分配一个页面给该进程的cr3.
  - 修改进程对应的映射函数或者说映射数据结构(cr3), 使得进程虚拟地址空间中的内核代码和物理内存中的os代码建立映射。
  - 在操作系统的数据中必然存在一个数据结构来表示该进程，如下
    ```c
    struct mm_area{
        _Area range; //内存映射的区间
        int prot, flags, fd, offset;//映射的属性
        struct page* pages; //用于遍历该映射中所有的页面。
    }
    struct task{
        struct mm_area areas[16];
        ...
    }
    ```
    只需要在加载时设置好这些$mm\_area$,就可以。
   - 然后将$rip$置为$ELF~entry$(静态链接)或者 $lib.so$(动态链接)执行即可。
   - 当发生缺页情况或者dereference nullptr时，cr2寄存器中会保留发生异常的地址，然后根据这些信息到$tas$k的$mm\_area$去查询有没有匹配的，如果没有匹配则出现异常，否则根据$mm\_area$中信息从磁盘中读取数据到物理内存中。


## Swap  
- Demand paging的另一个用途就是其反方向。
- 一个进程已经分配了页面，但我们可以主动将该页面swap out,保存到磁盘中，然后使用原来该页面的物理内存，当又需要该部分数据时，又swap in.
- LRU是较好的Swap策略。


## Shared Objects 和 外部符号
- 动态链接库采用GOT表格，而如果修改对应的GOT表项，比如将printf对应的表项的地址改为自己写的printf\_wrapper的地址，那么当我们调用printf时就会去执行printf\_wrapper，而我们在printf\_wrapper中同样可以去调用printf函数，并且在之前打印一些log，这样我们就得到了<font color=red size=3>ltrace</font>
  
- 如果两个动态库中同时定义了同一个动态符号，则先到先得，如果我们能让自己写的一个动态库在其他所有动态库之前被加载进去，那么就能使得进程在调用某些函数时，进入我们的动态库执行，这就实现了<font color=red size=3>lhooking</font>，在linux中提供了环境变量<font color=red size=3>LD\_PRELOAD</font>，设置这个环境变量为相应的动态库，其就能在加载完系统必须的动态库后第一个被加载。