---
author: "Lingyu Zhang"
author_link: "https://jarviszly.com"
title: "持久数据的可靠性RAID"
date: 2020-06-26T11:10:06+08:00
lastmod: 2020-06-26T11:10:06+08:00
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

## 持久数据的可靠性
- 增加持久数据可靠性的方法-备份

### random corruption(某块随机损坏)和FAT备份
- 损坏情况
  - 元数据(FAT)损坏-高破坏性
  - 系统文件损坏-可能造成无法开机
  - 数据文件损坏-丢失一个文件，勉强可以接受
- 应对部分数据损坏-自动备份最重要的数据(FAT)$\times k$
  - k由文件系统初始化时设置
  - 更改FAT时需要对多个FAT同时更改
- 性能影响
  - FAT读取：不受影响，从任意一个读取即可，当FAT-i损坏的时候，从FAT-i+1读取即可
  - FAT更新：需要同时写入k个，但因为FAT部分经常访问所以其在内存中有副本，所以其实写入磁盘的频率很低
- 可靠性分析：
  - 假设有一个FAT有n个cluster，每个以p概率损坏
  - 可以抵抗任意连续不超过n个的损坏。
  - 原来存在损坏$1-(1-p)^n$变成$1-(1-p^k)^n$ 
- FAT损坏恢复(M5)
  - mmap将所有cluster读入内存
  - 分类所有的cluster(目录文件，bitmap头，bitmap数据)
  - 然后解析目录文件找到bitmap头，根据假设bitmap头后面的一些cluster就是bitmap数据，可以进行简单的恢复，或者根据相似性去进行更严谨的恢复

### fail-stop(整块磁盘坏掉)和RAID
- RAID(廉价磁盘冗余阵列)
  - 把多块磁盘虚拟成一块虚拟磁盘(RAID)
  - 允许任意一块磁盘的损坏
  - 利用带宽
    - 比如有两个磁盘，需要读取1234，则可以发成从A中读12，从B中读34，可以通过DMA并行完成，从而提高读取效率。
    - 写入的时候，也可以利用DMA并行的写入，虽然浪费了一半的带宽。
- RAID更多的设计空间
  - RAID-0: 不考虑备份，可以用两块1G的磁盘虚拟成一块2G的磁盘，并且相对一块2G的物理磁盘读写效率提升了一倍。
  - RAID-1: 就是上面所说的备份
  - RAID-10: 将上面两种组合起来，比如4块盘，先分别用两块构成RAID-1，然后在其基础上构成一个RAID-0
  - $v_0,v_1,v_2,v_3 \rightarrow A_0,B_0,C_0,D_0,E_0$，用多一块磁盘实现任意一块物理盘损坏的容忍和高效率，类似于奇偶校验。
    - $A,B,C,D$中对应存放$v_0,v_1,v_2,v_3$的值，而$E$中存放$v_0\oplus v_1 \oplus v_2 \oplus v_3$
    - 性能：随机读顺序读都是$80\%$带宽，顺序写也是$80\%$带宽(提前在内存中算好奇偶校验然后一起写)，随机写(单独写一块时，读取该块原来的数据和对应的校验磁盘中的对应数据)$P^\prime = P \oplus oldval \oplus newval$
  - 性能瓶颈解决办法：将校验位P均匀分布到每一个磁盘中。
  - ![](/images/documents/操作系统/RAID5.png)

## 更可靠的存储