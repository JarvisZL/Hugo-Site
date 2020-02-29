# 计算机网络-第五章




# 第五章
<!-- more -->

## 路由选择算法
- 路由选择算法的目的是从发送方到接收方的过程中确定一条通过路由器网络的好的路径。
### 分类
- 集中式路由选择算法和分散式路由选择算法
  - 集中式路由选择算法：用**完整的全局性的网络信息**计算出最低开销路径，以所有节点之间的连通性及所有链路的开销作为输入，如**链路状态算法(LS)**
  - 分散式路由选择算法：路由器以 **迭代，分布式方法**计算出最低开销路径，每个节点仅有与其直接相连的链路开销信息即可开始工作。如 **距离向量算法(DV)**
- 静态路由选择算法和动态路由选择算法
- 负载敏感算法和负载迟钝算法
  - 现在因特网路由选择算法(RIP,OSPF,BGP)都是负载迟钝算法。

### LS算法
- 在该算法中，网络拓扑和所有链路开销都是已知的。实践中是通过让每个节点向网络中**广播链路状态分组**来完成的。其中链路状态分组包含和自己相连的链路的开销信息，这常常由 **链路状态广播算法**完成。
- 使用协议：OSPF，使用的算法：Dijkstra算法，复杂度$O({|V|}^2)$
- ```C
  1   Initialization: 
  2    N’ = {u} 
  3    for all nodes v 
  4      if v is a neighbor of u 
  5        then D(v) = c(u, v) 
  6      else D(v) = ∞ 
  7  
  8   Loop
  9    find w not in N’ such that D(w) is a minimum 
  10   add w to N’ 
  11   update D(v) for each neighbor v of w and not in N’: 
  12         D(v) = min(D(v), D(w)+ c(w, v) ) 
  13    /* new cost to v is either old cost to v or known 
  14     least path cost to w plus cost from w to v */ 
  15  until N’= N
  ```
- 链路状态算法运行过程：
  ![table](/images/documents/计算机网络-第五章/LStable.png)
- 存在的问题：**拥塞敏感的选择振荡**
  - 在拥塞敏感且将路径负载当作开销的情况下，所有路由器同时运行LS算法，会使得由于选择了某一条目前最短开销路径而导致网络中路径开销改变，LS算法得出的最短开销路径会不断摇摆振荡。
  - 解决办法是 **使每台路由器广播链路状态分组的时间随机化**。

### DV算法
- 该算法是一种 **迭代的，异步的，分布式**的算法，使用的协议有BGP，RIP.
- 使用的算法：Bellman-ford算法，复杂度$O(|V||E|)$，算法关键 $d_x(y) = min_v\{c(x,v),d_v(y)\}$
- DV算法的基本思想为：
  - 初始计算自己的距离向量$D_x = [D_x(y): y\in N]$,发送自己的距离向量给相邻节点。
  - 保存相邻节点距离向量，并依据其更新自己的距离向量，再循环。
- ```C
  1  Initialization: 
  2    for all destinations y in N: 
  3       D (y)= c(x, y)/* if y is not a neighbor then c(x, y)= ∞ */ 
  4    for each neighbor w 
  5       D (y) = ? for all destinations y in N 
  6    for each neighbor w 
  7       send distance vector  D = [D (y): y in N] to w 
  8 
  9  loop 
  10    wait  (until I see a link cost change to some neighbor w or 
  11            until I receive a distance vector from some neighbor w) 
  12 
  13    for each y in N: 
  14        D (y) = min {c(x, v) + D (y)} 
  15 
  16 if Dx(y) changed for any destination y 
  17       send distance vector D  = [D (y): y in N] to all neighbors 
  18 
  19 forever
  ```
- 算法运行过程
  ![table](/images/documents/计算机网络-第五章/DVtable.png)
- 存在的问题：**无穷计数**
  - 当DV算法稳定后，网络中某条链路开销突然改变后，可能会出现**路由选择环路**，节点间的最短路径互相依赖，结果会在节点间不断来回变换，直到最后稳定，变换的次数和链路开销改变后的值相关，值足够大时会导致变换很多次，因此被称为无穷计数问题。
  - 解决方法：**毒性逆转**，即某一结点z到达x的**最小开销路径的下一跳**是y的话，z发送给y的信息中，z到x的最小开销为正无穷，**使得此时y不可能通过依赖z来选择到达x的最小开销链路**，从而避免无穷计数。当然毒性逆转并不能完全解决无穷计数问题，如**作业题**。

### 路由协议
#### RIP（Route Information Protocol)协议
- 使用DV算法，应用于7层OSI模型的应用层。
- 在路由表或报文中Distance Matric表明当前节点(路由器)到达目的子网的跳数，取值为[0,15]，到达和路由器直接相连的子网跳数为0，取值为16则表示正无穷。后来不仅使用跳数，还增加了队列长度作为链路开销。
- 各个路由器每30s中向相邻节点发送一次路由表信息。
- 超过180s没有接收到相邻节点的路由表信息则认为已经失去和相邻节点的连接。
#### OSPF(Open Shortest Path First)协议
- 使用LS算法和洪泛链路状态信息，周期性广播链路状态。
- 每一个路由器上都储存了一个含有着整个自治系统网络信息的有向图，其中路由器，主机，子网都是节点，他们之间的关联作为图的边。<br>
<img src= "/images/documents/计算机网络-第五章/OSPF.png" style = "zoom:45%">
- 通过在本地运行LS算法，生成一个以自己为根的最短路径树(SPF Tree)。
- 路由表中储存目的节点，下一跳位置和距离。
- OSPF的优点：
  - 安全：所有OSPF消息都经过身份验证，以防止恶意入侵。
  - 多条相同开销的路径：当存在多条相同开销的路径时，OSPF允许使用多条路径。
  - 对单播和多播路由选择综合支持。
  - 支持在单个AS中的层次结构：在AS内部可以划分区域，每个区域运行自己的OSPF,然后由一个主区域为其他区域之间流量提供路由选择。

#### BGP(Border Gateway Protocol)协议
- 是一种 **自治系统间路由选择协议**。
- 在BGP中，分组并不是路由到一个具体的目的地，而是一个CIDR化的前缀，即路由到一个子网或一个子网的集合，所以运行BGP协议的路由器的转发表项格式大致为(前缀，接口)。
- BGP给每个AS提供了如下工具：
  - 通告路由信息：
    - 通过eBGP(external BGP)来确保AS内部某个子网被整个因特网中其他AS所知。
    - 通过iBGP(internal BGP)来确保这个信息在其他AS内部传播到所有BGP路由器上。
  - 确定到目的前缀"最好的"路由：
    - 当BGP通告前缀的时候，前缀中会带有一些属性比如AS-PATH和NEXT-HOP.
      - 当一个前缀通过某个AS时，该AS会在该前缀的AS-PATH上加上自己的AS number. 该属性用来检测和防止环路，当路由器在AS-PATH中看到了自己所在的AS的number,则说明有环路，拒绝该通告。 
      - NEXT-HOP是AS-PATH中起始的路由器接口的IP地址。
    - 热土豆路由选择：选择的路由是到 **开始该路由的NEXT-HOP路由器的最小开销**，即自己AS内部开销最小，而不管其他AS。
    - 路由选择算法:(1)本地偏好优先。(2)最短AS-PATH优先。(3)热土豆选择。((4)BGP标识符选择)

#### IP任播
- 常用于DNS中，使得每个用户能够从最靠近自己的服务器上获得想要的内容。
- 比如DNS根IP只有13个，但是每一个IP被分配在不同地方多个DNS根服务器上，通过BGP通告后，这些不同地方会被当作到达同一地址的不同路径，然后在用户申请访问的时候，根据上面说的路由选择算法，选择最靠近自己的服务器。

### SDN控制平面
TBD(好像没讲)
