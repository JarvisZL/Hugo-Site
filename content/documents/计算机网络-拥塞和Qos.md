---
author: "Lingyu Zhang"
author_link: "https://jarviszly.com"
title: "计算机网络-拥塞和Qos"
date: 2020-02-29T12:59:22+08:00
lastmod: 2020-02-29T12:59:22+08:00
draft: false
description: ""
show_in_homepage: true
description_as_summary: false
license: ""

tags: [计算机网络]
categories: [计算机网络]

featured_image: ""
featured_image_preview: ""

comment: true
toc: true
auto_collapse_toc: true
math: true
---



# 拥塞和Qos
<!-- more -->
## 拥塞

### 拥塞原因

- 大量分组在网络上传输，数量超出路由器的处理能力。

### 拥塞代价

- 造成丢包，时延；重传使得浪费之前发送的资源。

### 拥塞控制方法

- **抑制分组(choke packet):** 产生拥塞的路由器通过特定协议(如ICMP)向源端系统发送抑制分组来抑制源端系统的分组发送。
- **反向施压(Backpressure):** 当传播时间大于传输时间时，choke packet往往没有作用，则通过产生拥塞路由器向前面一个路由器反向施压抑制前面路由器的发送，这样传递下去，最终影响到源端系统。
- **标志位(Warning bit):** 通过在分组上设置一个标志位来告诉端系统可能发生拥塞。

  - 前向标记(FECN): 发生拥塞相同的方向
  - 后向标记(BECN): 发生拥塞相反的方向
- **拥塞窗口**：使用在端系统上(如TCP)
- **早期随机丢弃(RED)**: 作用在路由器上，该方式设置一个下限 $L_{min}$，一个上限 $L_{max}$。
  - 当队列的长度小于小于下限时，保证不会丢掉分组
  - 当队列的长度介于上下限之间时，对于每一个接收到的分组，路由器以概率 **p**丢弃
  - 当队列长度完全大于上限时，丢弃接收到的分组。
- **流量整形（Traffic shaping):** 在建立连接之后，端系统和路由器之间协商流量整形的方式。
  - 约定数据速率(CIR): RED方式。
  - 漏桶：通过像漏斗一样的东西控制流量，输入速率随意，输出速率固定。
  - **令牌桶**：允许多变的输出速率。
    - 令牌桶容量：一次性最多能够输出分组的数量。
    - 令牌产生的速率：用来控制输出分组的平均速率
    - 可以在令牌桶后面连接一个容量为1的令牌桶来控制**峰值速率**



## Qos的两种服务

### ISA(Integrated Services Architecture)

- 由于太复杂，现在还没用。

### DS(Differentiated Services)差异化服务

- 使用IPv4和IPv6里面的服务类型和流量类型来区分不同的分组的重要性。



