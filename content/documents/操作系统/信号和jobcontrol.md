---
author: "Lingyu Zhang"
author_link: "https://jarviszly.com"
title: "信号和jobcontrol"
date: 2020-06-21T20:59:55+08:00
lastmod: 2020-06-21T20:59:55+08:00
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

## 信号机制
- 想要实现ctrl+c使得进程退出，如何实现？

### 谁收到了ctrl+c这个东西?
- 如果在shell中运行一个cat, 然后ctrl+c会发现cat退出
  - 此时shell在wait等待cat执行完返回，所以其不是收到处理ctrl+c的地方
  - cat此时正在执行read系统调用等待用户的输入，所以也不是。
- 实际上是操作系统的对象<font color=red size=3>终端</font>，其收到ctrl+c后会给cat进程发送一个SIGINT的信号。那么怎么样实现信号机制，使得无论该进程在干什么，都能被通知。

### 信号机制的实现
- 这样的需求可以联想到ICS中提到的硬件的中断机制，将硬件上的中断机制搬到进程上来当作信号机制。
- 假设cat进程在cpu上运行，当tty收到一个ctrl+c，操作系统会等到该cpu上一个中断到来，此时cat进程的所有信息(寄存器现场，内存)都被保存到了他的栈上实际上也在物理内存中，所以操作系统可以去修改这部分内存，在其间插入跳转到Signal处理的代码去执行，于是该进程就一定能够被通知。
- 在linux中提供了一些API，比如c语言中的signal可以注册一个处理某个信号的函数，这样就能够捕获到系统中的信号，从而做出不同的行为。



## job control
- 如果一个进程中fork了很多个进程，那么在这个进程执行的时候，如果ctrl+c，那么应该是要所有fork的进程都要退出。
- 但是，如果这样，那么在shell上执行的所有进程都是通过shell进程fork出来的，那么此时ctrl+c，岂不是所有的进程都被杀死，所以这样简单的设计显然是不够的，应该将这些进程分组管理。
- 而实际上，在linux系统中，当一个shell启动后就会构成一个session结构，而session结构有一个control terminal和若干进程组。
  - 进程组中的进程具有不同进程号，但是具有相同的进程组号，fork子进程会继承父进程的pgid，而execve不会改变pgid.
  - shell每次fork出新进程都会为其创造一个独立的进程组。
  - 在一个session中，任意时刻，都只能有一个位于前台的进程组。
  - 信号会发送给前台进程组中所有的进程。
  - man 8 setpgid