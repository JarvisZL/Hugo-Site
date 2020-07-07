# GraphTheory


## 图论

### 正则图(regular graph)
- 图中所有顶点的度数一样。

### 生成子图(spanning) 和 诱导子图(induced)
- 生成子图：具有原图所有的顶点和部分边。
- 诱导子图：an induced subgraph of a graph is another graph, formed from a subset of the vertices of the graph and all of the edges connecting pairs of vertices in that subset.


### 图的同构(isomorphism)
- an isomorphism of graphs G and H is a bijection between the vertex sets of G and H
- the graph isomorphism is an equivalence relation.
  - 自反(reflexive property)，对称(symmetric property)，传递(transitive property)。

### 匹配
- a matching in a graph is a set of edges without common vertices.

### 最大匹配
- Among all the matches in a graph, the match with the largest number of matching edges

### 完美匹配
- the perfect match covers all the nodes in the graph.

### Menger 定理
- 在连通图G中一个u-v分离集的最小顶点数量等于G中u到v不相交路经的最大条数。

### 欧拉迹，欧拉回路，欧拉图
- an Eulerian trail (or Eulerian path) is a trail in a finite graph that visits every edge exactly once (allowing for revisiting vertices). 
- an Eulerian circuit or Eulerian cycle is an Eulerian trail that starts and ends on the same vertex.
- Eulerian graph is a graph which contains an Eulerian cycle.
- a graph is an Eulerain graph if and only if the degree of each node is even.

### 哈密尔顿路，哈密尔顿回路，哈密尔顿图
- a Hamiltonian path  is a path in a graph that visits each vertex exactly once.
- an Hamiltonian circuit or EHamiltonian cycle is a Hamiltonian path that starts and ends on the same vertex.
- Hamiltonian graph is a graph which contains an Hamiltonian cycle.


### 竞赛图
- 在无相完全图上给每条边确定一个方向后得到竞赛图。
