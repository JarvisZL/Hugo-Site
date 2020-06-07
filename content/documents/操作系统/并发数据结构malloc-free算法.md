---
author: "Lingyu Zhang"
author_link: "https://jarviszly.com"
title: "并发数据结构malloc Free算法"
date: 2020-05-30T21:37:09+08:00
lastmod: 2020-05-30T21:37:09+08:00
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

## 并发数据结构-并发链表
- 在要使用很多次链表的场合，定义链表头时应该直接定义为一个链表结点，而不是一个指针。
  - 当链表头是一个指针时，在操作时就需要考虑指针是否为空的情况。
  - 当链表头是一个结点时，就不需要考虑这个情况，此时链表头不存储任何数据 。