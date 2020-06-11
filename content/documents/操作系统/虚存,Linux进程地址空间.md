---
author: "Lingyu Zhang"
author_link: "https://jarviszly.com"
title: "虚存,Linux进程地址空间"
date: 2020-06-11T22:51:05+08:00
lastmod: 2020-06-11T22:51:05+08:00
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

## Linux进程的地址空间
- 进程的地址空间如何创建，创建之后如何修改？
```c
//映射
void* mmap(void *addr, size_t length, int prot,  int flags, int fd, off_t offset);

int munmap(void *addr, size_t length);

//修改映射权限
int mprotect(void *addr, size_t length, int prot);
```
- 看mmap手册可以知道其作用是可以将某个文件中的一部分映射到内存中一部分，所以很容易可以想到在进程地址空间创建时，需要将进程的代码和数据搬到地址空间，所以可以用mmap来当作加载器。
- 那么mmap又是如何实现的，通过strace观察mmap可以发现，其使用了/proc/proc_id/maps文件中的信息，这个文件中记录了该进程的所有内容在文件和地址空间间的映射关系。
- 通过cat查看/proc/proc_id/maps的内容，可以发现进程的地址空间为
- ![](/images/documents/操作系统/procaddr.png)
- <font color=blue size=3>动态链接库在heap和stack之间</font>
  
- 而在地址空间最后三个部分，发现了三个神秘的东西(vvar,vdso,vsyscall)
  - vvar: 内核和进程共享的数据
  - vdso: 系统调用的代码实现，这些系统调用不需要陷入内核执行。(比如time，getcpu等)
  - vsyscall: 以前的系统调用的代码实现，但因为安全问题，已经不再使用，为了向后兼容而保留。


## 虚拟存储-分页机制
- 为了实现mmap这样的映射机制，硬件给予的支持就是虚拟存储-分页机制。
- 而分页机制最本质的问题就是找一个合适的数据结构来表示虚拟地址到物理地址的映射关系，已经将虚拟地址和物理地址以页为单位划分，则需要表示$2^{20}$到$2^{20}$的映射关系。
- 使用数组是第一个很直观的想法，但是数组开销太大，无法忍受。
- 使用radix-tree(基数树)
  - 在32位机器中，设定一页为4KB，一个指针为4B，所以一页可以存放1024个指针，所以一页可以表示10位，每一个指针又指向这样一个页面，于是两级表示了20位，在第二级结点中，每个指针直接指向一个实际的页面，于是总共表示了32位。
  - 在64位机器中，用了4级表示了48位，实际上64位机器其物理地址为48位。